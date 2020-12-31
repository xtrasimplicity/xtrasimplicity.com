---
title: "PasswordState - Check-in a password from the database backend"
date: 2020-12-31T14:57:37+11:00
draft: false
tags: ['PasswordState', 'password', 'check-in', 'MS SQL']
categories: ['Infrastructure']
---

For security and auditing reasons, some passwords that we store in PasswordState require a user to check-out the password and then check it back in before others can check it out. As part of this process, users are required to supply a reason why they require access to a specific password, which is very, very useful from an auditing perspective.

The problem, however, is that sometimes users accidentally forget to check the password back in, and unfortunately, there doesn't appear to be any administrative GUI option to check a password back in on another user's behalf.

As a result, accessing specific passwords after-hours can become a little challenging if a user forgot to check the password back in and is now incommunicado.

Historically, I've been lucky in the sense that I didn't _need_ access straight away (though it would be nice!), so I was simply able to wait a few days until my colleagues were back in the office and could check it in for me.

Today, however, I desperately needed access to a password that had been checked-out by a colleague, and I didn't want to bother him as it is New Years Eve and the office is closed for another few weeks. After not having much luck with Google, I decided to take a look at the MS SQL database that our PasswordState deployment uses to see if I could forcefully check the password back in. Thankfully, the database structure is fairly self-explanatory and I was able to quickly identify the correct table and the column values that needed to be changed.

For future reference, I've added some steps below.

1. Log in to the database server using MS SQL Server Management Studio (SSMS) or your preferred client.
2. Search the `Passwords` table for the password you're looking for. To make this easier, you can simply run the following SQL to show all passwords that are currently checked out:
```sql
SELECT [PasswordID], [Title], 
        [UserName], [Description], 
        [GenericField1], [GenericField2], [GenericField3],
        [AccountTypeID], [Notes], [URL], [Password], 
        [ExpiryDate], [AllowExport], [PasswordListID], 
        [Deleted], [LinkID], [GUID], [GenericField4], [GenericField5], 
        [GenericField6], [GenericField7], [GenericField8], [GenericField9], 
        [GenericField10], [EnablePasswordResetSchedule],[PasswordResetSchedule], 
        [AddDaysToExpiryDate], [PasswordResetEnabled], [WebUser_ID], [WebPassword_ID], 
        [ScriptID], [LastUpdated], [AddedFromDiscovery], [PrivilegedAccountID], [HeartbeatStatus],
        [HeartbeatPoll], [HeartbeatSchedule], [RequiresCheckOut], [ChangeOnCheckin], [CheckInSchedule], 
        [CheckInAt], [CheckOutUserID], [CheckOutUserName], [HeartbeatEnabled], [ValidationScriptID], [HostID], 
        [ADDomainID], [ResetStatus], [LastResetDate], [NextAgentSchedule], [ValidatewithPrivAccount], [OTPUri], [FavIcon]
  FROM [passwordstate].[dbo].[Passwords] WHERE RequiresCheckOut = 1;
```
3. Once you know the `PasswordID` of the password that you want to check back in, simply set the `RequiresCheckOut` column to `0`, and the `CheckOutUserID` and `CheckOutUserId` columns to `NULL`. e.g:
```sql
UPDATE [passwordstate].[dbo].[Passwords] 
  SET RequiresCheckOut = 0,
      CheckOutUserID = NULL,
	  CheckOutUserName = NULL
  WHERE PasswordId = YOUR_PASSWORD_ID
```