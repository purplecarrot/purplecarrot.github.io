---
title: Verifying TLS certificates in Python
author: Mark
thumbnail: images/thumbnails/python.png
date: '2015-12-03T18:11:00.001Z'
categories:
  - Development
tags:
  - Python
  - Security
thumbnail: images/thumbnails/python.png
---

All too often people I see Python code using the requests module to query an API that just bypass TLS verification (i.e it sets `verify=False` in the requests function calls). On RedHat Linux and derivatives, you can simply set this to `verify=/etc/pki/tls/certs/ca-bundle.crt` and your Python code will then automatically use any system CA certs are part of TLS verification.

If like me, the environment you work in has a private Root CA that even requires and issues certs to internal services on an internal private network, you can really easily add this root CA cert (and any intermediate CA certs you might want to trust) to the systems cert chain:

```shell
# cp mycompany-root-ca.crt /etc/pki/ca-trust/source/anchors
# /usr/bin/update-ca-trust
```

But perhaps you work on regulated systems where you don't have privileged root access? In that case, you can set the shell environment variable `REQUESTS_CA_BUNDLE=$HOME/mycert.crt`. 

I can relate to the fact that sometimes we just want to get the code written as quickly as possible, but it doesn't have to be difficult and it's worth getting in the habit of leaving TLS verification turned on.
