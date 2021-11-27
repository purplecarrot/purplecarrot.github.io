---
title: OpenSSL Command Line Quick Reference
author: Mark
thumbnail: images/thumbnails/linux.png
date: '2014-04-24T15:55:00.000+01:00'
modified_time: '2015-01-14T17:12:26.528Z'
categories:
  - Linux
tags:
  - Linux
  - oneliners
  - Security
url: /2014/04/24/openssl-command-line-quick-reference/
---


Is there anybody in the IT industry that wasn't doing something with openssl in April? <a href="http://heartbleed.com/"></a> I don't use the openssl command line utility that often, but last week it reminded me that openssl command line tool is pretty comprehensive and has some nice features. I thought on this occasion I'd record the most useful ones so that next time I don't have to look them up again!   

``` shell
# Show Certificate Info
$ openssl x509 -text -in server.cert
$ openssl md5 server.cert
$ openssl sha1 server.cert

# Confirming openssl build info
openssl version -a
OpenSSL 1.0.1e-fips 11 Feb 2013
built on: Tue Apr  8 00:29:11 UTC 2014
platform: linux-x86_64
options:  bn(64,64) md2(int) rc4(16x,int) des(idx,cisc,16,int) idea(int) blowfish(idx) 
compiler: gcc -fPIC -DOPENSSL_PIC -DZLIB -DOPENSSL_THREADS -D_REENTRANT -DDSO_DLFCN -DHAVE_DLFCN_H -DKRB5_MIT -m64 -DL_ENDIAN -DTERMIO -Wall -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -grecord-gcc-switches  -m64 -mtune=generic -Wa,--noexecstack -DPURIFY -DOPENSSL_IA32_SSE2 -DOPENSSL_BN_ASM_MONT -DOPENSSL_BN_ASM_MONT5 -DOPENSSL_BN_ASM_GF2m -DSHA1_ASM -DSHA256_ASM -DSHA512_ASM -DMD5_ASM -DAES_ASM -DVPAES_ASM -DBSAES_ASM -DWHIRLPOOL_ASM -DGHASH_ASM
OPENSSLDIR: "/etc/pki/tls"
engines:  dynamic 

# Generate hashed passwords (for example, for Anaconda)
openssl passwd -1 mysecretword
$1$utlS7bht$VLMQrtHnPU0mkSO/Kpzh/.

# Directory complied in and used to search for openSSL files (--openssldir)
openssl version -d 

```

I tend to use commercial SSL certificates or ones from my company's internal CA. However, I found this text file in my home directory which I'm recording here for next time because it's a very easy quick reference for generating a self signed certificate:  

``` shell
# Generate new private key
openssl genrsa -aes256 -out my.key 4096

# Generate new certificate request
openssl req -new -key my.key -out my.csr

# Sign certificate
openssl x509 -req -days 3650 -in my.csr -signkey my.key -out my.crt

# Remove cert password
openssl rsa -in my.key -out my.key 
```
