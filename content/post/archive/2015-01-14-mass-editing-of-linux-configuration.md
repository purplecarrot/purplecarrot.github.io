---
title: Mass In Place Editing of Linux Configuration Files
author: Mark
thumbnail: images/thumbnails/linux.png
date: '2015-01-14T17:20:00.004Z'
modified_time: '2015-01-14T21:15:36.059Z'
categories:
  - Linux
tags:
  - oneliners
  - Linux
url: /2015/01/14/mass-editing-of-linux-configuration/
---

Generally, if I have 100 files that I need to edit and make changes in, I tend to write a perl or python script to make them (if I have 3 files, I just open them in VIM and make them manually!) Yesterday however, a colleague who doesn't code had to change 2 consecutive lines in about 400 similar files and wanted a simple "one liner" type way to do it.

This perl one liner is what I gave him. It allows replacement of multi line text with another piece of multiline text, in place, in the file.

``` shell
$ cat example.conf
Line1
Line2
Line3
Line4
$ perl -i -la0pe '~s/Line2\nLine3/ReplaceLine2\nReplaceLine3/g' example.conf
$ cat example.conf
Line1
ReplaceLine2
ReplaceLine3
Line4

```

It's then easy to use shell script to loop around all the files in a directory and run this command. If you're nervous, it's best to first check what your changes do by perhaps piping the file through perl. 

``` shell
$ cat example.conf  | perl -la0pe '~s/Line2\nLine3/ReplaceLine2\nReplaceLine3/g'
$ cat example.conf 
Line1
ReplaceLine2
ReplaceLine3
Line4

```

Another idea is also to use the -i.bak option to make backup files so you have a copy of the files before the edits ;-)  
