---
title: Integrated Web Serving with BaseHTTPServer
author: Mark
thumbnail: images/thumbnails/python.png
date: '2013-08-16T17:49:00.004+01:00'
modified_time: '2013-09-10T14:52:59.369+01:00'
categories:
  - Development
tags:
  - Python
  - Linux
url: /2013/08/16/these-last-few-days-i-have-been-writing/
---

These last few days I have been writing a command line reporting program at work. For Linux techs, formatted terminal output works great and is actually their preferred view. However, for managers, you simple must have the eye candy of a GUI. Normally in these situations, I would simply add CGI code and then bundle a simple Apache config to *Include* along with the code. With this particular program, it could be run on any one of tens of thousands of servers we have, however it is only likely to be run for a short while to graphically review some data stored in JSON files on the server. The overhead of reconfiguring and restarting Apache, even though only a few commands, is probably too much in this particular scenario.

Enter [BaseHTTPServer](http://docs.python.org/2/library/basehttpserver.html). This includes classes that let your program become the webserver itself, negating the need to run a CGI script or framework under full blown Apache.  


``` python
class MyWebServer(BaseHTTPServer.BaseHTTPRequestHandler):

    def do_HEAD(self):
        self.send_response(200)
        self.send_header('Content-type', 'text/html')
        self.end_headers()

    def do_GET(s):
        try:
            self.send_response(200)
            self.send_header('Content-type', 'text/html')
            self.end_headers()
            self.wfile.write('<html><head><title>Web Page</title></head>')
  
            if self.path == '/':
                self.wfile.write(show_table) 
            else:
                reportjson='%s/%s' % (rootdir,self.path)
                self.wfile.write(show_table(self.path))

        except IOError, e:
            self.send_error(404,'Oops: %s' % (e))


if __name__ == '__main__':
    httpd = BaseHTTPServer.HTTPServer(('0.0.0.0',2000),MyWebServer

    try:
        httpd.serve_forever()

    except KeyboardInterrupt:
        pass
        httpd.server_close()

</html>
```

Now, this obviously isn't going to replace your Apache installation (though I came across bugs in BaseHTTPServer where people were seemingly trying to!) but for development purposes and short lived web serving without affecting the running Apache server on your server, this is a very useful feature I am sure I will be using time and time again.
