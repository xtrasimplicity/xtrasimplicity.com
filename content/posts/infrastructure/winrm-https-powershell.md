---
title: "PowerShell: Automate 'WinRM over HTTPS' configuration with Self-signed certificates"
date: 2020-09-26T15:04:34+10:00
draft: false
tags: ['windows', 'powershell', 'WinRM', 'https']
categories: ['Windows', 'Infrastructure']
---

The following powershell script can be used to automatically generate a self-signed SSL certificate, and configure WinRM to accept connections over HTTPS.

**Tip:** If using Windows Admin Center, you'll need to import this certificate into the Trusted Root Certification Store on each of your Gateway servers, before you can connect to them.

If you have an internal CA, you might like to use it to generate a trusted certificate with the _[Request-Certificate](https://www.powershellgallery.com/packages/Request-Certificate/1.5.0)_ powershell module, and substitute its thumbprint into the `winrm create` command below.

```powershell
$hostname = $env:computername
$isRunningService = (Get-Service winrm).Status -eq "Running"

if (-not ($isRunningService -eq $true)) {
  Write-Host "Starting WinRM service..."
  Start-Service winrm
}

Write-Host "Generating self-signed SSL certificate..."
$certificateThumbprint = (New-SelfSignedCertificate -DnsName "${hostname}" -CertStoreLocation Cert:\LocalMachine\My).Thumbprint

Write-Host "Configuring WinRM to listen on HTTPS..."
winrm create winrm/config/Listener?Address=*+Transport=HTTPS "@{Hostname=`"${hostname}`"; CertificateThumbprint=`"${certificateThumbprint}`"}"

Write-Host "Updating firewall..."
netsh advfirewall firewall add rule name="Windows Remote Management (HTTPS-In)" dir=in action=allow protocol=TCP localport=5986
```