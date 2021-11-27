---
title: Remote XDMCP Connection to GDM
author: Mark
date: '2010-04-27T21:16:00.001+01:00'
modified_time: '2010-08-16T17:10:46.503+01:00'
thumbnail: 'images/thumbnails/purplecarrot.png'
categories:
  - Linux
tags:
  - Gnome
thumbnails: images/thumbnails/gnome.png
url: /2011/04/27/remote-xdmcp-connection-to-gdm/
---


I always forget how to do this. And it always takes me ages to find out what I did last time. Edit /etc/gdm/custom.conf and add:

```ini
[xdmcp]
enable=true

[greeter]
RemoteGreeter=/usr/libexec/gdmgreeter
```
