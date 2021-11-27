---
title: Who logs in the most?
author: Mark
thumbnail: images/thumbnails/linux.png
date: '2011-05-16T22:01:00.007+01:00'
modified_time: '2013-08-16T17:01:15.181+01:00'
categories:
  - Linux
tags:
  - Linux
  - Bash
url: /2011/05/16/who-logs-in-most/
---


I work in a firm with 10s of 1000s of Linux servers. A colleague had been emailed a list of servers and was trying to find out who owned them. At first, I suggested running last to see who last logged in but we realised this probably wasn't the best way to start the hunt. On reflection, the was a high chance that the user who logged in the *most* frequently into the server, was the the owner of that server. Or if not, he would most likely know the owner ;-)

So I used this 30 second one liner (wrapped for viewability) to list most frequent users:
``` perl
$ last | perl -lane '$users{$F[0]}++; END { 
foreach $u (reverse sort { $users{$a} <=> $users{b} } keys %users) 
{ print "$u logged in $users{$u} times"; } }'
tom logged in 108 times
sally logged in 81 times
dave logged in 32 times
root logged in 27 times
richard logged in 8 times
admin logged in 3 times
wtmp logged in 1 times

```

