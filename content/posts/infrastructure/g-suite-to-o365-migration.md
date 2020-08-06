---
title: "G Suite to O365 Migration"
date: 2019-07-17T16:43:13+10:00
draft: true
---

### Background
Assumptions:

1) Have on-premise AD, and do not want to move to Azure AD as primary.
2) Want Exchange online.
3) AD source of all truth.
4) No existing Exchange server.
5) Migrating from G Suite.
6) Don't want to buy licenses for users that are not ready to be merged.

### Account creation
#### Assumptions
- The user's *primary* email address is `username@domain.com`.
- We have the following domains listed in O365:
  - `domain.com`
  - `o365.domain.com`
- We have the following domains listed in our GSuite environment:
  - `domain.com` (primary)
  - `gsuite.domain.com` (alias)

#### Steps
- Add User to O365, and assign to the correct groups.
- Once user is in Azure AD
  - Remove the user from the GAL, if present
    ```powershell
    Remove-MailContact -Identity 'username@domain.com'
    ```
  - Set proxyAddresses property in AAD
    ```powershell
    $user = ... # [Microsoft.ActiveDirectory.Management.ADUser]
    $addresses = @("SMTP:username@domain.com", "smtp:username@o365.domain.com")
    $user.proxyAddresses = $addresses;

    Set-ADUser -Instance $user
    ```
  - Configure O365 to GMail forwarding in AAD
    ```powershell
    Set-Mailbox -Identity 'username@domain.com' -DeliverToMailboxAndForward $true -ForwardingSmtpAddress 'username@gsuite.domain.com'
    ```
- Start migration from MS Exchange Admin Center
  - Choose 'G Suite to O365' migration option.
  - Migration process will configure forwarders to forward emails from Google to O365.

References:

1) https://docs.microsoft.com/en-us/Exchange/mailbox-migration/perform-g-suite-migration#create-a-google-service-account
2) Co-existence: https://social.technet.microsoft.com/wiki/contents/articles/36118.configure-email-coexistence-between-office-365-google-apps.aspx