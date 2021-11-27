---
title: Configuring GNOME 3 Desktop and Shell
author: Mark
date: '2014-05-13T17:30:00.000+01:00'
thumbnail: images/thumbnails/gnome.png
modified_time: '2015-01-14T17:11:03.348Z'
categories:
  - Linux
tags:
  - Linux
  - GNOME
url: /2014/05/13/configuring-gnome-3-desktop-and-shell/
---


When it comes to customising GNOME, its config has always confused me - gconf, dconf, gconf-edit,  gconftool - what are they all for? The furthest I've ever delved into them was copying and pasting some tips (give me my minimize and maximize buttons back!).

My Linux work is almost exclusively with headless servers sitting in data centres I never see, so I've never taken the trouble to read or understand GNOME configuration. There has been lots of [controversy](http://http//en.wikipedia.org/wiki/Controversy_over_GNOME_3) over GNOME 3. I'm all for change, but if only the configuration of GNOME 3 was easy a little easier. Most options aren't available to set in the GUI control panel, but there are plenty of options to be set. Unfortunately they tend to be hidden away and it is not at all clear how to set common settings. In turn, because of GNOME changes between versions, googling GNOME config finds much inaccurate or at best, out of date information.

It was time to properly understand all these things.

Forget gconftool, gconf-edit and scripts that sed gschema XML files (yes, there are some people sed'ing XML files out there!). For GNOME 3 on Fedora 20 (RHEL 7), there are two tools of interest: gsettings and dconf-editor 
This is best illustrated by way of an example. Here is a list of common settings I put as default on my Fedora desktops. They should be pretty self-explanatory. The links below contain list of further configuration settings you can apply. I only wish these were more readily exposed in the GUI.

``` python
# Show Date in Gnome Shell Title Bar
cat \&lt;\<eof> /usr/share/glib-2.0/schemas/mysettings.gschema.override
[org.gnome.desktop.interface]
clock-show-date=true
always-show-log-out=true

# Set the default icons in the icon bar
[org.gnome.desktop.shell]
favorite-apps=['firefox.desktop', 'google-chrome.desktop', 'gnome-terminal.desktop', 'shotwell.desktop', 'nautilus.desktop', 'system-config-printer.desktop', 'minecraft.desktop']

[org.gnome.desktop.screensaver]
idle-activation-enabled=false

[org.gnome.shell.overrides]
button_layout=:minimize,maximize,close

[org.gnome.Terminal.Legacy.Profile]
use-theme-colors=false
use-system-font=false
foreground-color='#ffffff'
background-color='#000000' 
font='Inconsolata Medium 11' 

</eof>
```

**References**
* [RedHat RHEL7 Desktop Migration and Administration Guide](https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/9-Beta/html/Desktop_Migration_and_Administration_Guide/gsettings-dconf.html)
* [GNOME gsettings reference](https://developer.gnome.org/gio/stable/GSettings.html#GSettings.description)
* [dconf, gconf and gsettings and the relationship between them](http://askubuntu.com/questions/249887/gconf-dconf-gsettings-and-the-relationship-between-them)
