---
title: Getting VIM Powerline Working with Putty
author: Mark
thumbnail: images/thumbnails/git.png
date: '2013-09-09T22:06:00.000+01:00'
modified_time: '2013-09-10T13:18:19.494+01:00'
categories:
  - Development
tags:
  - Development
  - Linux
  - Git
url: /2013/09/09/getting-vim-powerline-working-with-putty/
---


After using vim and vi for more years than I care to remember, I came across the [vim powerline plugin](https://github.com/Lokaltog/powerline) for the first time today. What an excellent vim statusline, and now I've been using it myself, I can see why it's so popular (that's a relative term - as in popular for a vim extensions! )

Following the [installation documentation](https://powerline.readthedocs.org/en/latest/installation/linux.html#installation-linux), and thanks to [Pathogen](Pathogen) it was very straight forward to setup. After installation in my bundle directory, I simply switched Putty to UTF-8 and then added the following lines to my .vimrc

``` shell
set rtp+=~/.vim/bundle/powerline/powerline/bindings/vim
set encoding=utf-8
set laststatus=2
```

..and then it was all working nicely in Linux gnome-terminal.

However, when I came to using the same setup with Putty from Windows 7 machine (I have no choice, the firm I work for doesn't allow connection of a non-firm owned device to the network, so no connecting my Mac or Linux laptop here!), the character set was all screwed up.  This is because powerline uses custom glyphs in the status bar and the MS Win 7 supplied Consolas font doesn't include those glyphs. 

Unfortunately, my Win 7 machine doesn't have a full development environment, so [fontpatching](https://powerline.readthedocs.org/en/latest/fontpatching.html) the Consolas font I use (as instructed to do in Linux/Mac OS X) was not an [easy] option. Luckily, I found there were already Consolas patched fonts around other people had made available. I first tried [Inconsolata for Powerline](https://github.com/Lokaltog/powerline-fonts/blob/master/Inconsolata/Inconsolata%20for%20Powerline.otf) as this was what's in the powerline repo and what the server and my Linux workstation used, but though fine in Linux, that just didn't look pretty in Putty. Next, I found posts referencing these [patched Consolas](https://github.com/eugeneching/consolas-powerline-vim) fonts, but after installation, it still wasn't displaying the correct characters. Being a bit of a vim plugin amateur, I checked and double-checked settings but to no avail. 

Then I came across some comments about the new (rewritten in Python) powerline plugin not using the same codes as the old powerline.   By this time, I just wanted it working. This was one of those slight 10min deviations that was making me forget what I was originally doing. Enter cut and paste and the perl one-liner.

I used charmap to lookup the UTF-8 character codes, and then printed them to the terminal. Yippee! This at least confirmed my terminal, fonts and putty were correctly setup to display the characters, so the problem wasn't in either of these areas. So I looked at the default powerline config.json, and confirmed that it wasn't displaying the characters I was expecting to see. So next I [configured a local config.json](https://powerline.readthedocs.org/en/latest/configuration.html)
and simply cut and paste the characters output from the perl one-liner and pasted them into the config.json in my home directory.

And voila...a working vim powerline plugin on Linux from Putty on Win7. Now I know this is not the right solution, and normally curiosity would force me get to the bottom of this and understand what was wrong, but having already spent an hr reading about the plugin and comments and dead ends refering to the old non-python version of the plugin, this got me a working powerline in Windows and so I'm happy with that. I just had to have those cute triangles in my menu bar - and no, ugly chars was not an option :-) 

Getting SyntaxHighlighter to display these glyphs is certainly not something I want to spend 2 minutes investigating at the moment, so instead a screenshot of the perl one-liner and the changes to config.json:

![Glyph icons](/images/posts/powerlineputty.png)
