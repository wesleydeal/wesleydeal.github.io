+++
title = "Fix a locked Shoretel/Mitel Director"
date = 2020-09-15
updated = 2023-12-01
author = "Wesley Deal"
[taxonomies]
tags = ["software", "IT"]
[extra]
shorttitle = "Unlock a [Shore/Mi]Tel Director"
+++

Let's say, for the sake of argument, that you're overworked and understaffed. A while ago, something popped up on the top of your Shoretel management page. You noticed it when you were hastily onboarding a new employee:

>Due to license violation, access to Director will be locked in 38 days. Take proper action before the grace period has expired. Note that there are additional charges if you need to recover from a locked Director.

You made a mental note and moved onto the next thing.

Now it's 6 weeks later and you get a ticket to change someone's voicemail PIN. You go into the director but find that you're now locked out entirely.

But you can fix it: Shoretel stores your users' licensing status in a MySQL database that you can edit directly.

A note: this is all bad practice. Don't let your licenses expire. Don't run an old version of Shoretel on an older version of Windows and leave UAC and updates both disabled because that's what your vendor says you have to do. Don't poke around haphazardly in your production database with baby's first SQL Management app from an insecure RDP connection, reading instructions from some random website. Now I'm going to give you instructions to do all of the above.

## The Process

 - Remote in to your Shoretel server. If Shoretel is holding your server hostage on an old version of Windows, you might need to disable CredSSP remediation on your box with `reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\CredSSP\Parameters /v AllowEncryptionOracle /t REG_DWORD /d 2`

 - Install [MySQL Workbench](https://dev.mysql.com/downloads/workbench/) on the server.

 - Add a connection with these details:

   - Hostname: `localhost`

   - Port: `4308`

   - Username: `root`

 - Connect with the password `shorewaredba`

 - Run this query:  
   ```sql
   select UserDNID,UserDN,GuiLoginName,LicenseTypeID,ClientType from users order by licensetypeID
   ```

 - In the table that appears, edit the license type  numbers for LicenseTypeID and ClientType to reduce your license count. Refer to the locked director for which license you're over on. Refer to this table for the meaning of license type numbers.
   ```
   # | LicenseTypeID   | ClientType 
   --+-----------------+------------
   0 | ?               | None
   1 | Mailbox Only    | ?
   2 | Phone Only      | ?
   3 | Phone & Mailbox | ?
   4 | N/A             | ?
   5 | N/A             | Shoretel Connect  
   ```

 - In the bottom right corner, click **Apply**, review the changes, and **Apply** again. Exit MySQL Workbench.

 - Restart the Shoretel-Director service (or, if preferred, the entire server).
