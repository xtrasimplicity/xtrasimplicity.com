---
title: "PowerShell: Automate 'WinRM over HTTPS' configuration with Self-signed certificates"
date: 2020-09-26T15:04:34+10:00
draft: false
tags: ['windows', 'powershell', 'WinRM', 'https']
categories: ['Windows', 'Infrastructure']
---

The following powershell script can be used to automatically generate a self-signed SSL certificate, and configure WinRM to accept connections over HTTPS.

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