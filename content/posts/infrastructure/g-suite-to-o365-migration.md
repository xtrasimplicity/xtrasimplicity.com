---
title: "G Suite to O365 Migration"
date: 2019-07-17T16:43:13+10:00
draft: true
---

### Background
Assumptions:

1) Have on-prem AD FS. Do NOT want to migrate to Azure AD FS.
2) Want Exchange online.
3) AD source of all truth.
4) No existing Exchange server.
5) Migrating from G Suite.
6) Don't want to buy licenses for users that are not ready to be merged.

### Account creation
Powershell workflow: 
  - for each user in Pilot OU
    - set targetAddress (username@gmail.mycorp.com)
    - set proxyAddresses
      - SMTP:username@mycorp.com
      - smtp:username@o365.mycorp.com

References:

1) https://docs.microsoft.com/en-us/Exchange/mailbox-migration/perform-g-suite-migration#create-a-google-service-account
2) Co-existence: https://social.technet.microsoft.com/wiki/contents/articles/36118.configure-email-coexistence-between-office-365-google-apps.aspx