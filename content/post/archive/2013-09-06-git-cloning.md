---
title: Git Cloning Behind a Proxy
author: Mark
thumbnail: images/thumbnails/git.png
date: '2013-09-06T17:33:00.005+01:00'
modified_time: '2013-09-10T13:12:44.613+01:00'
categories:
  - Development
tags:
  - Git
url: /2013/09/06/git-cloning/
---


Normally my development environment lives on my nicely configured workstation. However, sometimes you're on a server, with no certificate chain installed for verification and behind a corporate firewall where git protocol is blocked and not permitted. If you just want to get at the code, using the https url to the git repository and turning off certificate verification can get you the code quickly.  I found the following git config items helpful in this situation:  
```shell
git config --global http.proxy https://proxy.mycompany.com:8080/
git config --global https.sslVerify=false
```

It is indeed not good practice to get accustomed to ignoring certs, but on this repo the cert had expired in any case, so it was a moot point. Whilst troubleshooting this issue, I also found the useful GIT_CURL_VERBOSE=1 environment variable that can be set to show curl interactions git is doing under the covers. 
