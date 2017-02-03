---
layout: post
title: UTF-8 and Euro Symbol
date: '2013-09-09T20:28:00.002+01:00'
author: Mark
tags:
- Linux
modified_time: '2013-09-09T22:30:20.905+01:00'
---

Quick and easy way to show that UTF-8 is configured correctly in shell and terminal emulator is to display the Euro symbol. If you don't see the Euro symbol, then UTF-8 is not configured correctly.

```
linux:~$ perl -CS -le 'print "\x{20AC}"'
€
```


