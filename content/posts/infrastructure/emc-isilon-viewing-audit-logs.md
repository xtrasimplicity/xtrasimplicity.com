---
title: "EMC Isilon Viewing Audit Logs"
date: 2018-11-27T10:22:29+11:00
draft: false
tags: ['EMC', 'Isilon', 'Auditing', 'Logs']
categories: ['Infrastructure']
---
To view audit logs on an EMC Isilon storage cluster, you can use the following command.


```shell
isi_audit_viewer -h
```

e.g:
 
```shell
isi_audit_viewer -s '2018-01-01 00:00:00' -e '2018-01-31 00:00:00' -t protocol | grep '\\\\ifs\\\\ZoneName\\\\path\\\\to\\\\folder'
``` 

Note that we need to escape backslashes when calling `grep`, so `\\` will become `\\\\`.

To make things more efficient for when re-reviewing the same audit log data, or when building your search expression for `grep`, you can simply pipe the output from `isi_audit_viewer` to a file, and then process that file. e.g:
```shell
isi_audit_viewer -s '2018-01-01 00:00:00' -e '2018-01-31 00:00:00' -t protocol > /tmp/audit_log.txt

# and then
cat /tmp/audit_log.txt | grep '\\\\ifs\\\\ZoneName\\\\path\\\\to\\\\folder'
```