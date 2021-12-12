---
title: Build a Python Version Agnostic RPM For Your Python Module
author: Mark
thumbnail: images/thumbnails/python.png
date: '2015-04-02T15:52:00.001+01:00'
modified_time: '2015-07-07T17:05:01.474+01:00'
categories:
  - Development
tags:
  - Python
url: /2015/04/02/build-rpm-for-your-python-module/
---


I often use the python setup.py bdist_rpm command on RHEL to build RPMs of the python modules I'm developing. This makes it easy for me to distribute my module via the yum repos internally at the company I work for.   At my company, most of our production servers are currently RHEL 6, but we are starting to deploy RHEL 7. When I took one of my recently built RPMs of a python module (that runs under python 2.6 and python 2.7), I was surprised to find that it did not install on RHEL7 because of missing dependencies. It turns out, that my RPMs "required" python 2.6 from RHEL 6 (RHEL 7 moves to python 2.7 as default).  <

``` shell
$ rpm -qpR python-mymodule-2.8-1.noarch.rpm
/usr/bin/python
python(abi) = 2.6
python-requests
rpmlib(CompressedFileNames) <= 3.0.4-1
rpmlib(PayloadFilesHavePrefix) <= 4.0-1
```

Even though I don't have a specific version of python listed in the requirements for the package, the rpmbuild called by setuptools automatically adds a dependency on the python version installed on the system I'm using. This is because when rpmbuild scans through my source, it then finds the shebang that points to #!/usr/bin/env python, which in turn finds the python 3.6 on the server where I build the RPM. Obviously, I could rebuild the rpm on a RHEL 7 host and it would then find python 2.7, but I don't want to have separate RPMs for different RHEL versions when the code is the same.  I wanted to be able to produce an RPM for my python module and code that installs and runs on RHEL 6 and RHEL 7 (when the python code is tested to run under python 2.6+)  Now this isn't particularly well documented, but I did a little digging and found that to do this, you can simply add the --no-autoreq option to setup.py

``` shell
$ python setup.py bdist_rpm --no-autoreq
```

This will then tell bdist_rpm to add the "AutoReq: no" option to the spec file used to generate the RPM. RPM build will then see this and the resulting RPM will then no longer require a specific version of python.

``` shell
$ rpm -qpR python-mymodule-2.8-2.noarch.rpm
python-requests
rpmlib(CompressedFileNames) <= 3.0.4-1
rpmlib(PayloadFilesHavePrefix) <= 4.0-1
```

You can then add the option to your setup.cfg file for the module, so that you don't forget it next time and inadvertently distribute a python version dependent module.  <

``` shell
[bdist_rpm]
no-autoreq = yes
```

Useful References:
------------------

- [RPM Automatic Dependencies](http://www.rpm.org/max-rpm-snapshot/s1-rpm-depend-auto-depend.html)
