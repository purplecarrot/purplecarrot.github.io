---
title: Recovering lost git commits (or undoing a git reset!)
author: Mark
thumbnail: images/thumbnails/git.png
date: '2015-10-28T16:31:00.001Z'
modified_time: '2015-11-19T16:04:50.325Z'
categories:
  - Development
tags:
  - Git
url: /2015/10/28/recovering-lost-git-commits-or-undoing/
---


We have recently recruited lots of new staff at the firm where I work and they are all learning git for the first time (and believe it or not, for many, it is their first experience of any VCS). I was surprised today when one of these new staff approached me because he had lost a couple of hours work!
 
The remote HEAD had changed quite a bit since his initial checkout, and he got himself into a bit of a mess pulling down changes and merging his local changes. To further compound the error, he did a git reset --hard HEAD~3, and then tried a few different git revert and git  reset commands to get it into the state he wanted! He asked me if there was an easy way to get these back (after he read git reset --hard is permanent and dangerous!). 
 
So we've all lost work at some point in our lives, but I've never managed to actually *lose* work in DVCS. Fortunately, I remembered a git tutorial a while back where they had talked about the reflog. So I knew his commit was "there somewhere", it just needed to be got back.
 
It turns out this is actually very simple. You simply view your reflog and then hard reset back to the commit SHA you want (I had to update the output below as it contains proprietary information from my firm!)

``` shell
$ git reflog
a30a408 HEAD@{0}: commit: Revert "XYZ"
4149dbf HEAD@{1}: commit: 4149dbf.... updating HEAD
a6b3b59 HEAD@{2}: commit: a6b3b59.... updating HEAD
957745c HEAD@{3}: HEAD~2: updating HEAD
daa6d14 HEAD@{4}: HEAD~3: updating HEAD
ebc7042 HEAD@{5}: pull: Merge made by recursive
635d7ff HEAD@{6}: commit: Last Good Commit
$ git reset --hard 635d7ff
$ git push -f
```

So these reset commits are basically in the reflog as garbage. This revert could be done immediately, but these garbage commits will eventually be deleted (configuration variable gc.reflogExpire which defaults to 90 days). So you won't always be able to go back to them, but it is good for immediate recovery of errors, which is realistically the case when you will be able to use this.
