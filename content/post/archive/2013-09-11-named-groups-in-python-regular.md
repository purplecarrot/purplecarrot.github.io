---
title: Named Groups in Python Regular Expressions
author: Mark
thumbnail: images/thumbnails/python.png
categories:
  - Development
date: '2013-09-11T10:14:00.000+01:00'
modified_time: '2013-09-11T10:14:52.223+01:00'
categories:
  - Development
tags:
  - Python
  - Regex
url: /2013/09/11/named-groups-in-python-regular/
---


To anybody working with Unix and Linux systems, [regular expressions](http://en.wikipedia.org/wiki/Regular_expression) or regexes are extremely useful for fast pattern matching in scripts. First there were basic regexes now known as BREs as used by traditional Unix utilities like sed, and then came along the even more useful Perl Enhanced Regular Expressions. So useful were these, that many of the traditional GNU utilities in Linux were retrofitted with support for these Perl Compatible Regular Expressions (so ubiquitous are these PCREs now in GNU and OSS, that younger nix readers only familiar with Linux may assume the were always this good!)

Anyway, to the point of this post. Whilst checking some syntax earlier, I came across [named groups](http://docs.python.org/2/howto/regex.html#non-capturing-and-named-groups). I've never really stopped to read about these before.  Traditionally (and in most code I come across), groups matched in regexes are captured with ( and ), and then "replayed" with $1, $2 (or \1 or \2 for sed, etc). This is fine when you're only trying to match 1 or 2 groups. But what happens when you have a hideously complicated match with 10+ matches. Editing and maintaining this is nightmare and fraught with problems. This is where named groups come in.

How named groups work is best illustrated with a very simple example. Imagine a file like this, that is used to set shell variables that a command line utility picks up from the environment.

``` shell 
export DBSERVER=dbserver.example.com
export DBPORT=2000
export CITY=London
export COUNTRY=UK
```

Now imagine I need to write a python app that needs to use those same variables without duplicating them in a second config file. An easy way to do this would be with the following code using normal numbered groups.

``` python
#!/usr/bin/env python
import re

regex = re.compile('export (\w+)=(.*)')
settings = {}

f = open('app.conf','r')
for line in f:
    m = regex.match(line)
    if m:
        if m.group(1) and m.group(2):
            key = m.group(1).lower()
            settings[key] = m.group(2)

f.close()
print settings
```

Running this gives you a dictionary with a key for each shell variable that was read in:

``` json
{
 "dbport": "2000", 
 "country": "UK", 
 "city": "London", 
 "dbserver": "dbserver.example.com"
}
```


Switching to named groups, the functionally equivalent code could be written like this:

``` python
#!/usr/bin/env python
import re

regex = re.compile('export (?P<shellvar>\w+)=(?P<value>.*)')
settings={}

f = open('app.conf','r')
for line in f:
    m = regex.match(line)
    if m:
        if m.group('shellvar') and m.group('value'):
            key = m.group('shellvar').lower()
            settings[key] = m.group('value')

f.close()
print settings</value></shellvar>
```


 This is just an example. You probably don't **need** to use named groups in this instance and there is probably a config file parsing module anyway, but it' a good illustration on named groups and numbered groups.

Now my last thought on the matter is that I wonder if the reason I've never come across or used named groups before is because if I'm matching a hideously long string into more than half a dozen groups, I'd probably tackle the problem in another way (read more simple and readable) such as breaking the string down with split, etc. Regular expressions are great, but the complicated ones don't contribute well to code maintainability.   
