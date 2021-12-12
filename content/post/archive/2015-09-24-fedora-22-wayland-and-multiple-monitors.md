---
title: Fedora 22, Wayland and Multiple Monitors
author: Mark
date: '2015-09-24T20:57:00.004+01:00'
modified_time: '2015-10-18T09:05:55.084+01:00'
thumbnail: images/thumbnails/gnome.png
categories:
  - Linux
tags:
  - Linux
  - GNOME
url: /2015/09/24/fedora-22-wayland-and-multiple-monitors/
---


By default, a new Fedora installation or a fedup upgrade on my dual monitor machine will result in GDM displaying the list of users and login screen on my left hand monitor. But I want it on my right hand monitor...

Up until now, I simply logged in and set up the configuration I wanted using Settings -> Displays. Then I took the `~/.config/monitors.xml` file that is generated and copied it to `/var/lib/gdm/.config/monitors.xml` ready for GDM to use.

However, with Fedora 22, it appears that GDM is set to use Wayland, however once you login to a session, it continues to use X. This new Wayland config uses slightly different XML directives in the monitors.xml meaning a simple copy of your `~/.config/monitors.xml` to `/var/lib/gdm/.config/monitors.xml` no longer works.

To work around this, click on the cog and select GNOME on Wayland. Then setup the Displays as you need them and then copy the `~/.config/monitors.xml` file (which will be in "Wayland" format) again to `/var/lib/gdm/.config/monitors.xml` and restart GDM (there may be a better way, but I switch to a virtual console and do init 3 followed by init 5)

Then your GDM login screen will be correctly setup.

_UPDATE 20151008: After upgrading to latest rpm set last night, this is now broken again and GDM is displaying the login on the monitor I don't want it to be on. I verified the monitors.xml file for gdm was correct, but for some reason gdm doesn't appear to be honoring it. Not being something I want to investigate, I decided to just disable Wayland for the moment by adding the WaylandEnable=False /etc/gdm/custom.conf :-)_
