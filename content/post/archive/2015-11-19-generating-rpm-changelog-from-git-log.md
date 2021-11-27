---
title: Generating a RPM changelog using git log
author: Mark
thumbnail: images/thumbnails/python.png
date: '2015-11-19T16:11:00.001Z'
modified_time: '2015-11-19T18:37:06.247Z'
categories:
  - Development
tags:
  - Python
  - Linux
url: /2015/11/19/generating-rpm-changelog-from-git-log/
---

I use python setuptools bdist_rpm to build RPMs from the code in my git repositories and today I updated my automated build script (a wrapper around python setup.py to run pre-requisite functions) to generate an RPM changelog from the git log of the repository hosting the package and then automatically include this in the rpm being built. A search on github shows there are a few python scripts and pypi modules to do this, but I wondered if I could just use git log.

It seems you can. The [Fedora packaging](http://fedoraproject.org/wiki/Packaging:Guidelines#Changelogs) describe the format of the changelog file. They also appear to recommend not using your vcs changelog as your changelog messages would be too terse for the users. However, in my case they're not and it's exactly what they want
 
With git log you can use the --pretty option to format the output of the commit log. You select the appropriate formats from git [pretty-formats](http://git-scm.com/docs/pretty-formats) to make it output the git log in RPM changelog format. Unfortunately though, none of the date formats available are quite the correct format for an RPM changelog. It expects just the date of the change and doesn't want (nor can it handle) the time or the timezone offset of the git commit. The way I addressed this was to pipe the git log output through perl to remove those date fields. It would be nicer if git log had date +%H:%M:%S type formatting, but it doesn't, so we add the perl oneliner with a suitable regex to remove the parts we don't want.
 
Also rpm-build (or perhaps setuptools) doesn't like any blank lines in the changelog file, so we remove those with perl -00 option. The contents of the generated changelog file (not the filename) are then passed into the setup.py --changelog option. The double quotes ensure the whitespace is preserved. 
 
``` shell
$ git log --pretty="tformat:* %cd %an <%ae> (%h)%n- %s%b%n" \
| perl -lap00e '~s/(\d{2]:){2}\d{2} (\d{4}) [+-]\d{4}/$2/g' > changelog
$ python setup.py bdist_rpm --changelog="$(&ltchangelog)
```
