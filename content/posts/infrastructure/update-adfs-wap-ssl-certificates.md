---
title: "Updating ADFS and ADFS WAP SSL certificates"
date: 2022-12-1T09:30:00+10:00
draft: false
tags: ['windows', 'adfs', 'saml']
categories: ['Windows', 'Infrastructure']
---
These are some details I've found around the net, for future reference.

## ADFS Server
- Import your new cert into the local computer certificate store.
- Get the thumbprint, using `cmd` or `powershell`, with:
```powershell
dir cert:\LocalMachine\My
```
- Update ADFS to use this cert, using PowerShell:
```powershell
Set-AdfsSslCertificate -Thumbprint "THUMBPRINT_HERE" -CertificateType Service-Communications
Set-AdfsSslCertificate -Thumbprint "THUMBPRINT_HERE"
```
- Make sure that your ADFS service account can read the cert (do this using `certlm.msc`).
- Restart the ADFS service


## ADFS WAP Servers
- Import the new cert into the local computer certificate store.
- Get the thumbprint, using `cmd` or `powershell`, with:
```powershell
dir cert:\LocalMachine\My
```
- Update the WebApplicationProxy certificate, and re-configure the ADFS trust:
```powershell
Set-WebApplicationProxySslCertificate -Thumbprint "THUMBPRINT_HERE"
Install-WebApplicationProxy -CertificateThumbprint "THUMBPRINT_HERE"
```
- Update all published applications to use the new certificate:
```powershell
Get-WebApplicationProxyApplication | Set-WebApplicationProxyApplication -ExternalCertificateThumbprint "THUMBPRINT_HERE"
```
