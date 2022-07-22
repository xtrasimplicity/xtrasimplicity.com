---
title: "PowerCLI: Get all powered-on Ubuntu VMs, and their IP addresses"
date: 2022-07-22T15:40:00+10:00
draft: false
tags: ['windows', 'powershell', 'vmware', 'powercli']
categories: ['Windows', 'Infrastructure', 'VSphere']
---

The following powershell script can be used to automatically get a list of all powered-on Ubuntu VMs, along with their IP addresses, from a VCenter server.

```powershell
Connect-VIServer myServer.tld

(Get-VM).where{$_.PowerState -eq "PoweredOn" -and $_.ExtensionData.Guest.GuestFullName -match "Ubuntu"} | Select -Property Name, @{N="IP Address";E={$_.ExtensionData.Guest.IpAddress}}
```