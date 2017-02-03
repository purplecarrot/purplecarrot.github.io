---
layout: post
title: Remote XDMCP Connection to GDM
date: '2010-04-27T21:16:00.001+01:00'
author: Mark
tags:
- Linux
modified_time: '2010-08-16T17:10:46.503+01:00'
---

I always forget how to do this. And it always takes me ages to find out what I did last time. Edit /etc/gdm/custom.conf and add

``` shell
[xdmcp]
enable=true

[greeter]
RemoteGreeter=/usr/libexec/gdmgreeter

```


