---
title: "PowerShell: List all IP addresses within AD DNS that match a specific subnet"
date: 2019-02-13T18:32:50+11:00
draft: true
---
```powershell
PS > Get-ADComputer -Filter * -Properties ipv4Address | Where-Object IPv4Address -Match "^192.168.0.*" | Sort-Object -Property IPv4Address | Format-List name, ipv4*
```
