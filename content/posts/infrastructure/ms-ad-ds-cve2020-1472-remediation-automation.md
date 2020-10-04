---
title: "MS AD DS - CVE-2020-1472 Remediation Automation"
date: 2020-10-04T18:32:08+11:00
draft: false
tags: ['windows', 'powershell', 'CVE-2020-1472', 'ZeroLogon', 'AD DS', 'Active Directory']
categories: ['Windows', 'Infrastructure']
---

The following is a simple powershell script which takes advantage of PsExec to fetch event log backups from remote systems (defined in the `$servers` array) and store them into the specified `$targetLocation`.

You can bypass PsExec and just use `Get-WMIObject` with the `-ComputerName`, but I had some issues doing this with a few DCs where PsExec worked perfectly.

Once you've run this against all of your DCs, you can simply run [Microsoft's recommended PowerShell script](https://support.microsoft.com/en-us/help/4557233/script-to-help-in-monitoring-event-ids-related-to-changes-in-netlogon) against this path to identify any occurrences of event IDs related to CVE-2020-1472.

```powershell
$servers = @() # An array of FQDNs or IPs of your domain controllers
$targetLocation = "" # A path that is accessible from all DCs. This can be a UNC path, or a path local to each DC.

foreach($server in $servers) {
 psexec \\$server powershell.exe -command "(Get-WmiObject -Class Win32_NTEventlogFile | Where-Object LogfileName -eq 'System').BackupEventLog('$($targetLocation)\$($server).evtx')"
}
```