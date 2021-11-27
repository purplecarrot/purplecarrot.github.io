---
title: Python Setuptools and Building RPMs with Dependencies.
author: Mark
thumbnail: images/thumbnails/python.png
date: '2015-04-07T16:42:00.000+01:00'
modified_time: '2015-07-08T10:02:44.838+01:00'
categories:
  - Development
tags:
  - Python
  - OpenStack
url: /2015/04/07/python-setuptools-and-building-rpms/
---

If you develop python modules for distribution, you likely use python setuptools. It's very useful and takes lots of the manual operations out of distributing python code and modules. I recently started going one step further and using the bdist_rpm setup.py command option to automatically build rpms for my python code and modules too. Unfortunately, I found a bug (on RHEL 6 version of setuptools at least) in that it doesn't appear the install_requires directive in setup.py is being read or used correctly.
 
After adding _install_requires=['requests']_ to the setup.py file, the rpm spec file that _python setup.py bdist_rpm_ created did **NOT** include the correct _Requires:_ line required to include the dependency in the RPM. I experimented with the argument supplied (eg python-requests/requests) and other such things, but was not able to build an RPM with the dependencies on the python-requests RPM that I wanted to include.
 
After some playing and research, I was finally able to get it to generate an RPM with the correct dependency on python-requests by creating and using this custom setup.cfg file:

``` shell
cat << EOF > setup.cfg
[bdist_rpm]
requires = python-requests >= 1.1
no-autoreq = yes
EOF

```

The other line in there (_no-autoreq_) tells setup.py not to analyse your code for dependencies and include those in the RPM too. I added this because by default, the generated RPM had tied a dependency to python 2.6 (RHEL 6) but I wanted an RPM generated that would run on python 2.7 (RHEL 7) too so adding this flag removes any dependency on a specific python version
