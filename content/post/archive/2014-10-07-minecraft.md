---
title: Minecraft GNOME Desktop Icon
author: Mark
thumbnail: images/thumbnails/gnome.png
date: '2014-10-07T18:30:00.000+01:00'
categories:
  - Linux
tags:
  - Linux
url: /2014/10/07/minecraft/
---


I had an email about the minecraft.desktop file in the config file in a previous post. My kids love to play Minecraft, and installation of this simple GNOME .desktop file gives them the launcher icon on their desktops.  

```ini
cat << EOF > /usr/share/applications/minecraft.desktop
[Desktop Entry]
Version=1.0
Name=Minecraft
GenericName=Minecraft
Comment=Minecraft
Exec=java -jar /data/minecraft/Minecraft.jar
Icon=mcicon256
Terminal=false
Type=Application
EOF
```

Then of course, you must copy the nice shiny minecraft icon into /usr/share/icons/mcicon256.png. I also created a very basic puppet manifest to install it on their computers too!:  

``` shell
$ cat minecraft.pp 
file { ["/data/minecraft"]:
    ensure => "directory",
}

file { "/usr/share/icons/mcicon256.png":
    ensure => file,
    owner => root,
    group => root,
    mode => 644,
    source => "puppet:///files/minecraft/mcicon256.png"
}

file { "/usr/share/applications/minecraft.desktop":
    ensure => file,
    owner => root,
    group => root,
    mode => 644,
    source => "puppet:///files/minecraft/minecraft.desktop"
}

file { "/data/minecraft/startminecraft.sh":
    ensure => file,
    owner => root,
    group => root,
    mode => 755,
    source => "puppet:///files/minecraft/startminecraft.sh"
}

file { "/data/minecraft/Minecraft.jar":
    ensure => file,
    owner => root,
    group => root,
    mode => 644,
    source => "puppet:///files/minecraft/Minecraft.jar"
}
```
