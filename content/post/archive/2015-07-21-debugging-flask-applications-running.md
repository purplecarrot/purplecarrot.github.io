---
title: Debugging Flask applications running under Apache mod_wsgi
author: Mark
thumbnail: images/thumbnails/python.png
date: '2015-07-21T10:46:00.002+01:00'
modified_time: '2015-07-21T10:46:25.178+01:00'
categories:
  - Development
tags:
  - Python
thumbnail: images/thumbnails/python.png
url: /2015/07/21/debugging-flask-applications-running/
---


I have recently been developing a new python web application that uses the Flask framework. Whilst running standalone, it was working fine. Then I went to setup the UAT environment which is running under Apache using mod_wsgi. I've done this many times before without any problems. However, this time I was getting a "500 Internal Server Error" in my browser. I opened up the Apache error_log expecting a stack trace error in there that will pinpoint where my typo or forgotten config option that I need to fix is, but no. Nothing. It just said "Loading WSGI script" and implied everything was working ok.  

Then I read that mod_wsgi discards all stdout and as I realised my error was early in the startup process, I wasn't able to see the error message or stack trace I needed to debug the problem. To see this error, you can add basic logging to stderr in your WSGI script to send the missing errors to stderr and therefore on to Apache error_log. In my case the error was simply an IOError exception because of a missing file I had forgot to copy over (yes, I will add that to my tests!)

``` python
import sys
import logging 
logging.basicConfig(stream=sys.stderr)

from myapplication import app as application
```

