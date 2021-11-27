---
title: My Subversion Repo Moved!
author: Mark
thumbnail: images/thumbnails/linux.png
date: '2013-09-30T08:08:00.001+01:00'
modified_time: '2013-10-31T11:31:31.428Z'
categories:
  - Development
tags:
  - Development
url: /2013/09/30/my-subversion-repo-moved/
---


Well, it didn't strictly move anywhere but my broadband IP address changed after re-DHCP. I used to use no-ip.com, but then I no longer really host anything on my home network, so I figured could save myself 20 bucks as I didn't think I needed it anymore. Then I went to check in some old code to an old svn repo (that's another story) and of course it failed (and luckily for me, no bad guy had a repo sitting on the old ip!)  Anyway, this is easily fixed by first checking the URL of the working copy and then switching it with the svn switch command:

``` shell 
server:~/svn/work$ svn info
Path: .
Working Copy Root Path: /home/mark/svn/pyf
URL: https://123.45.56.78/svn/pyf
Repository Root: https://123.45.56.78/svn/pyf
Repository UUID: 7e254e78-910f-47b9-b796-8845927be892
Revision: 31
Node Kind: directory
Schedule: normal
Last Changed Author: mark
Last Changed Rev: 3
Last Changed Date: 2013-02-05 14:15:13 +0000 (Tue, 05 Feb 2013)
server:~/svn/work$ svn switch --relocate https://123.45.56.78/svn/pyf \
https://231.1.2.3/svn/pyf

```

