---
title: Writing OpenStack Hooks
author: Mark
thumbnail: images/thumbnails/linux.png
date: '2015-04-23T18:00:00.000+01:00'
modified_time: '2016-02-29T14:59:59.007Z'
categories:
  - Development
tags:
  - OpenStack
  - Python
url: /2015/04/23/writing-openstack-hooks/
---


I've recently been writing OpenStack Nova hooks for integration of newly deployed OpenStack nova instances with our existing infrastructure systems. Though the [OpenStack Developer Documentation](http://developer.openstack.org/) is very good, there is not that much documentation out there on real world end-user experiences of writing nova hooks. In this post, I will document my experiences in writing a nova hook for the first time and may be it will help others approaching the same task.

Summary and Goals
-----------------
Linux or Windows VMs provisioned by OpenStack for use in my firm must be registered with a number of other "Enterprise" systems external to the OpenStack core components. For example, an inventory system, an LDAP directory, a separate corporate IPAM solution (DNS) as well as security audit systems. Generally, the key for recording this data in the external system is either the new instance's hostname, IP address or mac address. This integration must be done at the point of instance/vm creation or deletion (because in some cases the registration event must occur before the new instance has booted) and so OpenStack nova hooks work well for this.

What are OpenStack Hooks?
-------------------------
OpenStack hooks allow you - as the system integrator - to write custom python code to execute any business logic or custom logic and then have this code execute when the relevant OpenStack function is called.  

You write your code as a standard python module and using setuptools for packaging and deployment, just as if you were writing any other python module. You then simply define entry points and your code will get executed.  

Your code can be setup to run either before the hooked function (pre hook) or after (post hook) the hooked function. At the time of writing, nova only has a handful of hooks declared but I would imagine that in time hooks will be declared across many more methods and across many OpenStack components. 

Currently with the version of OpenStack I'm working with, you could add custom code to be run on nova create_instance, delete_instance or resize_instance method and this would get executed when that method is called internally as part of those nova processes. 

You then create your package and define the methods you want to get called alongside the OpenStack nova methods where you want them to run. Here is an example of the setup.py binding custom package methods to nova create and delete instances: 

setup.py
--------

``` python
setup(
    name="myfirmshooks",
    version="1.0",
    description="My Firms Hooks",
    author="Me",
    entry_points={
        'nova.hooks': [
            'create_instance=myfirmshooks.nova:CreateInstance',
            'delete_instance=myfirmshooks.nova:DeleteInstance',
        ]        
    },
)
```


mynovahook.py
-------------
The code is actually very simple indeed. You add the code you want to run inside the post of pre method of the class. Because this code runs inside nova-api, you have access to lots of information inside the args and kwargs passed into the function which can be used inside your code. For example, the uuid, name, flavor and much more are all available with very little efforts for your code to use.

``` python

class MyNovaHook(object)
    """ object for running custom code """
    def __init__(self):
        self.log = logging.getLogger(__name__)

    def pre(self, *args, **kwargs):
        self.log.info("Tasks that run before the instance is created")

    def post(self, *args, **kwargs):
        self.log.info("Tasks that run after the instance is created")
        """REST requests for interaction with external systems"""
```

Scale
-----
There isn't much information out there on customising OpenStack workflow like this. There is lots of information about using ansible, puppet, chef and similar tools for *post* build configuration of the operating system inside instance, but less information on running code that you want to run on behalf of the instance, but which cannot or you do not wish to run inside the instance. However, it should be noted that code running inside the post hook will block and/or error the instance creation. It maybe that a message bus approach would scale better with a short simple hook that puts a message on a bus and a secondary daemon running on the system that consumes and acts on that message and I may look at implementing that in future. 

References
----------
The following pages were useful when I started out writing nova hooks. 

- OpenStack Developer Documentation on Hooks<br>
[http://docs.openstack.org/developer/nova/devref/hooks.html](http://docs.openstack.org/developer/nova/devref/hooks.html)

- Python Setuptools Documentation<br>
[http://pythonhosted.org/setuptools/index.html](http://pythonhosted.org/setuptools/index.html)

- Lars Kellogg-Stedman's Blog Entry on Writing Nova Hooks<br>
[http://blog.oddbit.com/2014/09/27/integrating-custom-code-with-n](http://blog.oddbit.com/2014/09/27/integrating-custom-code-with-n/)


