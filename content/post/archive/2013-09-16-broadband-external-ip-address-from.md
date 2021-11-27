---
title: Broadband External IP Address from Command Line
author: Mark
thumbnail: images/thumbnails/linux.png
date: '2013-09-16T09:04:00.003+01:00'
modified_time: '2013-09-16T09:08:58.542+01:00'
categories:
  - Linux
tags:
  - Linux
url: /2013/09/16/broadband-external-ip-address-from/
---


Just typing "What is my external ip address" into Google is normally the most convenient way to find your external public IP address when you're behind a NAT'd broadband connection. The site ipconfig.me offers an alternative and clean way to get it from the command line, for example for use in a shell script:

``` shell 
server# curl ifconfig.me
132.231.56.65
server# curl ifconfig.me/host
132-231-56-65.myisp.com

```

They have a range of options to return IP, Hostname, UserAgent, etc. The one I may be using is ifconfig.me/all.json or all.xml, to make it even easier to read into a Perl or Python script!
