---
title: Configuring Out-Of-Band Management Interface from In-Band Server
author: Mark
thumbnail: images/thumbnails/linux.png
date: '2013-10-11T16:43:00.000+01:00'
modified_time: '2013-10-31T11:31:57.059Z'
categories:
  - Linux
tags:
  - Linux
  - Hardware
url: /2013/10/11/configuring-out-of-band-management/
---


Most modern x86 servers include [out-of-band](http://en.wikipedia.org/wiki/Out-of-band_management) Management or Lights-out Management systems. These systems offer remote configuration and remote console access to a headless server normally located in a datacenter. These are typically used when first setting up a server, or perhaps when the server is hosed.  The company I work for is large enough to have dedicated server operators, and in my role I rarely if ever need to access the consoles of real devices. However, I recently needed to get to the console of a device and the server was in a different continent and timezone and I couldn't access the IP address I'd been emailed :( Not a good start to the day.  I remember using ipmitool to configure OOB management before, but of course I never wrote down what I did then. As this blog is really a note taking memory aid system, I'm writing it down for next time.  So using ipmitool I was able to determine the IP address of the server.

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

 From this I noticed that the default gateway for the out-of-band was incorrect (as with all blog posts, the IPs are changed to RFC1918 private addresses here). What must have happen is that the server installation engineer had used a screen and keyboard on a trolley to configure the server, and then probably tested it from a device on the same subnet (I'm being generous here, as he/she may have never bothered testing it!) Anyway, quick man ipmitool for syntax and this was easily fixed from the in-band server with the following command:

```
ipmitool -I open lan set 1 defgw ipaddr 192.168.1.1
```

