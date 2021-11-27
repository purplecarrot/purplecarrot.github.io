---
title: PKI Certificate Encoding Formats and File Extensions
author: Mark
thumbnail: images/thumbnails/security.png
date: '2015-05-22T10:29:00.000+01:00'
modified_time: '2015-07-21T10:48:23.669+01:00'
categories:
  - Linux
tags:
  - Linux
url: /2015/05/22/pki-certificate-encoding-formats-and/
---



There are many different file formats and file extensions for storing certificates. It seems each time I come around to renewing a cert, I have forgetten what all the files and extensions mean.  So for next time...  Certificates are generally encoded in either Base64, PEM, or DEM formats. 

**Base64 (.cer)**
Originally developed for use with S/MIME (the first?) and the most widely supported format. More likely to be issued by Certificate Authorities that run on Linux.   A base64 encoded certificate can be opened in vi and will start with "-----BEGIN CERTIFICATE-----" and end with "-----END CERTIFICATE-----" with the BASE64 encoded data in the middle. 
 
**PEM - Privacy Enhanced Mail (.pem,.crt, .key, .cer)**
The PEM format is the most common format that CAs issue certificates in. They are ASCII and also BASE64 encoded and end and start with the same "-----BEGIN CERTIFICATE-----" and "-----END CERTIFICATE-----"  Certificates, certificate chains and private keys can be put into and distributed in PEM format. 
 
**DER - Distinguished Encoding Rules (.der, but also .cer or .crt)**
This is actually the same as PEM but whereas PEM is Base64 ASCII, DER is a binary form of the certificate and it does not have the BEGIN/END statements.   DER format supports storage of a single certificate, but can't be used for storing a private key or certificate chain. DER format is typically seen with java based platforms and applications. 
 
**PFX - Personal Information Exchange (.pfx)**
The PFX format (also known as PKCS #12) is a format that stores multiple certificates and private keys in one file. They are typically used on Windows machines to facilitate import and exports. They can be readily converted to the other formats 
 
**PKCS#7 (.p7b)**
Allows storage of certificates and certificate chains but not private keys and commonly used by Certicate Authorities to provide certificate chains to clients. 
 
