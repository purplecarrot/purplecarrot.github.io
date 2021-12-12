---
title: Subversion and gnome-keyring-daemon
author: Mark
thumbnail: images/thumbnails/linux.png
date: '2013-10-31T11:31:00.000Z'
modified_time: '2013-10-31T11:31:12.728Z'
categories:
  - Development
tags:
  - Subversion
url: /2013/10/31/subversion-and-gnome-keyring-daemon/
---


Subversion's "feature" to store repository passwords in plain text files is a little scary. In my opinion, even a minimal obfuscation routine in the svn client would be better than saving a user's clear text password in text files.

I obviously do not want to store my password in clear, so up until now, I've put up with typing my password in for each commit. But having recently created an automated packaging system for some software at work which uses multiple svn tags and copies, it was becoming tiresome having to repeatedly enter my password all the time.

As an alternative, Subversion supports using gnome-keyring-daemon to store the repository passwords but the servers I used don't have X or GNOME installed, but with a little bit of extra manual config, gnome-keyring-daemon can be used over SSH connections too.

The subversion documentation on setting this up and using gnome-keyring-daemon with SSH from the terminal is pretty poor, and the suggestions on sites and blogs I found discussing the problem didn't work for me. However, after reading these sites and some trial and error, I did get it working. The steps below show what I did to get gnome-keyring-daemon and subversion working on a RHEL 6 server with standard RHEL repos:

First install the following core RPMs that are in the standard RHEL repositories:

```
yum install gnome-keyring dbus-x11 subversion
```

Next, add the following to your .bashrc or .bash_profile to setup gnome-keyring-daemon when you're not running GNOME or a desktop:

```
eval `dbus-launch --sh-syntax`
eval `gnome-keyring-daemon`
```

Edit subversion config files to use gnome-keyring for passwords:
```
~/.subversion/config
[auth]
password-stores = gnome-keyring

~/.subversion/servers
[global]
store-passwords = yes
store-plaintext-passwords = no
```

Then when you do a subversion action for the first time you'll get asked for GNOME keyring password, but subsequent actions will no longer prompt for password:

``` 
server01:~/svn/proj/trunk$ svn update
Updating '.':
Password for 'svn' GNOME keyring:
At revision 864

server01:~/svn/proj/trunk$ svn -m "Test Auth" ci
Sending        GNUmakefile
Transmitting file data .....
Committed revision 865.
```

Sites that I found useful in getting this config working for me were the [Version Control with Subversion](http://svnbook.red-bean.com/en/1.7/svn.advanced.confarea.html) book and [this blog post](http://technicalprose.blogspot.co.uk/2011/06/using-subversion-with-gnome-keyring.html) describing a wrapper around svn and gnome-keyring that the author wrote to do the same.
