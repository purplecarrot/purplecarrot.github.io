---
title: 'Determining the IP Address of a Server''s Out of Band Management '
author: Mark
thumbnail: images/thumbnails/linux.png
date: '2013-10-11T16:11:00.000+01:00'
modified_time: '2013-10-31T11:31:46.330Z'
categories:
  - Linux
tags:
  - Linux
  - Hardware
url: /2013/10/11/determining-ip-address-of-servers-out/
---


Nearly all enterprise x86 servers include an [out-of-band management interface](http://en.wikipedia.org/wiki/Out-of-band_management) such as [HP iLO](http://en.wikipedia.org/wiki/HP_Integrated_Lights-Out), [IBM RSA](http://en.wikipedia.org/wiki/IBM_Remote_Supervisor_Adapter) or [BMC](http://en.wikipedia.org/wiki/Baseboard_Management_Controller#Baseboard_management_controller), [IPMI](http://en.wikipedia.org/wiki/Intelligent_Platform_Management_Interface), Dell DRAC, etc. These are used for remotely power cycling, monitoring and what the data centre ops guys, confusingly for me, call KVM access (that's Keyboard Video Mouse not Kernel Virtual Machine!)

As shown in {{ site.baseurl }}{% post_url 2013-10-11-configuring-out-of-band-management %}
this post, finding out the IP of the Out-of-Band Management interface from the "in-band" server is at times very useful. Here is the quick reference way to find that out from the Linux command line for popular OOBs:

**HP iLO**

``` shell
hpserver:~# hponcfg -a -w ilosettings.xml
HP Lights-Out Online Configuration utility
Version 4.0.0 Date 12/08/2011 (c) Hewlett-Packard Company, 2011
Firmware Revision = 1.29 Device type = iLO 2 Driver name = hpilo
Management Processor configuration is successfully written to file "ilosettings.xml"
hpserver:~# grep "<IP_ADDRESS VALUE" ilosettings.xml
    <IP_ADDRESS VALUE="192.168.1.1">
hpserver:~#
```


**IPMI**

``` shell 
ipmiserver# ipmitool -I open lan print 1
Set in Progress         : Set Complete
Auth Type Support       : NONE MD2 MD5 PASSWORD
Auth Type Enable        : Callback :
                        : User     : MD2 MD5 PASSWORD
                        : Operator : MD2 MD5 PASSWORD
                        : Admin    : MD5
                        : OEM      :
IP Address Source       : Static Address
IP Address              : 192.168.1.1
Subnet Mask             : 255.255.255.0
MAC Address             : 00:00:00:00:00:00
SNMP Community String   : public
IP Header               : TTL=0x40 Flags=0x40 Precedence=0x00 TOS=0x10
BMC ARP Control         : ARP Responses Enabled, Gratuitous ARP Disabled
Gratituous ARP Intrvl   : 2.0 seconds
Default Gateway IP      : 192.168.1.254
Default Gateway MAC     : 00:00:00:00:00:00
Backup Gateway IP       : 0.0.0.0
Backup Gateway MAC      : 00:00:00:00:00:00
802.1q VLAN ID          : Disabled
802.1q VLAN Priority    : 0
RMCP+ Cipher Suites     : 0,1,2,3,4
Cipher Suite Priv Max   : XaaaaXXXXXXXXXX
                        :     X=Cipher Suite Unused
                        :     c=CALLBACK
                        :     u=USER
                        :     o=OPERATOR
                        :     a=ADMIN
                        :     O=OEM
ipmiserver#
```

 
