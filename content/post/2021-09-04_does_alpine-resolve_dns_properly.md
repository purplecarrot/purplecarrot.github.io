---
title: "Does Alpine resolve DNS properly?" 
date: 2021-09-04T08:51:20+01:00 
description: "Investigations into musl libc DNS resolver used in Alpine containers"
featured: true 
draft: false
toc: true
thumbnail: images/thumbnails/container.png
categories:
  - Containers
tags:
  - Kubernetes
  - OpenShift
  - Linux
---

To anybody who has ever used containers, Alpine is a well known base operating system layer to many popular and widely used container images in use today. By design, it's a cut down minimal OS layer using the excellent [busybox](https://www.busybox.net/) and [musl libc](https://musl.libc.org/) C library. By using these alternatives to GNU Coreutils and GNU glibc C Library, it provides a base OS container layer that allows you to build and run very small Linux containers.

This week I was asked to look at a problem where an application team could not resolve a key DNS name used on our internal network when running their application within the container - their first thought was a problem with the OpenShift/Kubernetes environment where the container was running.

Unfortunately, the company I work for doesn't like us to disclose internal proprietary network information, so for the purposes of the post, let's say the DNS name being queried is `vis.company.com` (very important service). All IPs and some other data in the extracts below have been manually changed to private RFC1918 addresses or redacted too.

## Basic Troubleshooting

The first thing to do was to perform my own DNS lookup of `vis.company.com`. This worked fine from both my Linux workstation and Windows laptop. 

Now what made this problem interesting was that the application running in the container was able to resolve all other DNS entries that it was required to in order to function(eg `a.company.com`, `b.company.com`), but it couldn't resolve this one crucial DNS entry `vis.company.com`. When you connected to a shell running inside the container and ran `getent hosts vis.company.com` it simply returned nothing at all, and exited with exit code 2.

## Inside the Container
So how do you go about troubleshooting a problem like this from within a container? In a standard OS, you can simply use the standard tools in your sysadmin toolkit - ss (lsof/netstat), dig, strace, tcpdump, nmap, etc - but inside a container these are unlikely to be available by default (and some containers don't even include a minimal unix shell even). I have a pre-built container I use for situations like this that I then add as a sidecar in the pod I'm troubleshooting. Then by setting `shareProcessNamespace: true` in the podspec, it will allow you to easily debug and strace processes running inside the first container from the sidecar.

## DNS Query
So strace'ing the DNS queries I could see the `socket()` and `bind()` calls, followed by receiving a response from the DNS server. However, this response was empty. Something weird was happening, because running the same command on a standard Linux host you could see 100s of lines more of system calls and a good DNS response with multiple A records was returned.

So the next step was to use tcpdump and capture a packet trace of the DNS query. Below is the relevant section from that pcap that shows the DNS query:

```
18:00:27.376761 IP (tos 0x0, ttl 64, id 32569, offset 0, flags [DF], proto UDP (17), length 68)
    10.96.10.99.43507 > 10.96.0.10:domain: 19977+ A? vis.company.com. (40)
18:00:27.386585 IP (tos 0x0, ttl 64, id 24288, offset 0, flags [DF], proto UDP (17), length 79)
    10.96.0.10.domain > 10.96.10.99.43507: 19977| 0/0/1 (51)
```

So what does this tell us?

  1. This is standard UDP DNS query (id 19977) to query the IPv4 record `vis.company.com`
  2. The DNS server response comes from `10.96.0.10`, which is the Kubernetes `Service` address for CoreDNS.
  3. A DNS response was received, but no DNS records were in it (`0/0/1` = 0 answer records, 0 nameservers and 1 additional record which is the query address itself)
  4. The `|` character after the 19977 id indicates that the TC (TrunCation) bit is set on this packet.

The most interesting of these is the last. The TC bit is used when the DNS response the DNS server wants to send to the client is longer than the 512bytes available to it in a UDP packet (see [RFC1035](https://datatracker.ietf.org/doc/html/rfc1035) for more information.) 

This is a signal to the DNS resolver client that it needs to switch from a standard UDP DNS query and do a new TCP DNS query instead - but in both the strace and tcpdump output of the application running the the Alpine container, the resolver query exited immediate after receiving this TC UDP packet from the DNS server.

I expected this to be a bug, but it turned out that this is a [functional difference between musl libc and glibc](https://wiki.musl-libc.org/functional-differences-from-glibc.html) and is by design. The musl libc author states that he intentionally didn't support TCP as felt it would be better for performance and UX reasons:

{{< tweet user="richfelker" id="994629795551031296" >}}

This is true, and a valid point. However, it means that this particular app would never work when running in a container based on Alpine. Sometimes predictable functionality is more important. 

There are various other ways this could have been made to work. Perhaps making the DNS entry smaller - which was indeed unnecessarily large - would indeed be a better fix, but unfortunately this wasn't a practical option as the DNS entry wasn't under this team's control.

Instead, the quickest fix was to rebuild the application into a new container image using the RedHat UBI8 image as a base layer instead of Alpine. This image uses glibc resolver, and then the app then ran fine, in the same way as it did on the RedHat Linux 8 host where it had ran before being containerized. 
