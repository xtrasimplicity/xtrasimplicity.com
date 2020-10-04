---
title: "EMC Isilon Viewing Audit Logs"
date: 2018-11-27T10:22:29+11:00
draft: false
tags: ['EMC', 'Isilon', 'Auditing', 'Logs']
categories: ['Infrastructure']
---
To view audit logs on an EMC Isilon storage cluster, you can use the following command.


``shell
isi_audit_viewer -h
``

e.g:
 
``shell
isi_audit_viewer -s '2018-01-01 00:00:00' -e '2018-01-31 00:00:00' -t protocol | grep '\\\\ifs\\\\ZoneName'
``
