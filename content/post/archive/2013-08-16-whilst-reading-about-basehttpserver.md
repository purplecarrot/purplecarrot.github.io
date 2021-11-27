---
title: SimpleHTTPServer
author: Mark
thumbnail: images/thumbnails/linux.png
date: '2013-08-16T17:59:00.001+01:00'
modified_time: '2013-09-10T14:53:36.329+01:00'
categories:
  - Linux
tags:
  - Python
url: /2013/08/16/whilst-reading-about-basehttpserver/
---


Whilst reading about [BaseHTTPServer](http://docs.python.org/2/library/basehttpserver.html) class as mentioned in the last post, I also came across the very useful [SimpleHTTPserver](http://docs.python.org/2/library/simplehttpserver.html) which is most definitely worth recording as a one-liner.  Everybody must have come across the situation where you're ssh'd into a headless server with no Apache installed or running on it, and you come across an HTML file you want to look at. You can read it with vi (or lynx!) but that's never really going to be that nice. As python is near-ubiquitous on all Linux installations these days, you can run this simple oneliner to view HTML docs on the system (or in fact any files you might want to view graphically instead of via a terminal?)  

``` python
server:/usr/share/doc/bash-3.2$ python -m SimpleHTTPServer 8000
Serving HTTP on 0.0.0.0 port 8000 ...
127.0.0.1 - - [16/Aug/2013 17:51:49] "GET / HTTP/1.1" 200 -

```

