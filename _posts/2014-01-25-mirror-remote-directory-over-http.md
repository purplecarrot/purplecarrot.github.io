---
layout: post
title: Mirror a Remote Directory over HTTP
date: '2014-01-25T16:07:00.000Z'
author: Mark
tags:
- Linux
modified_time: '2015-04-02T16:07:25.047+01:00'
---

Different ways to mirror a remote directory: 

``` shell
wget -r --no-parent http://server/location
```

If there are any files you don't want you can add the --reject option: <

``` shell
wget -r --no-parent --reject="tmp*" http://server/location
```

``` python
curl
```


