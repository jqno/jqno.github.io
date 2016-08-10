---
title: "Configuring my NAS"
tags:
- synology
- how-to
---
The other day, I had to re-install my NAS, and of course, I'd forgotten exactly how I had configured it. That meant finding back lots of websites, some of which apparently didn't exist anymore. This time, I've written down all the steps I've taken, just in case, for some sad, unforeseen reason, I ever have to do it again.

What do I want from my NAS? Not much, actually:

* I want it to store my files (duh)
* I want to use an rsync script to copy my photos to my NAS
* I want filenames with special characters (éüå) to be treated sanely by said rsync script
* I want it to run [CrashPlan](http://www.code42.com/crashplan/) so all my data is automatically backed up to the cloud

I don't need it to be a media server; I have Spotify and Netflix for that.

My NAS is a Synology DS213j running DSM 5.1.


Starting out
---
After choosing a server name, a user name and a password, make sure to first install all the updates from the Control Panel. Add the user account to the administrators group, then disable admin and guest account. Next, create a shared folder: in File Station, click Create, Create New Shared Folder. Give it a name and click OK. Give your user Read/Write permissions to the folder.


Set up SSH
---
Once the preliminaries are out of the way, you can set up SSH, which is needed for rsync.

* Go to Control Panel, "Terminal &amp; SNMP" and check "Enable SSH service"
* Go to Control Panel, User, Advanced, and check "Enable user home service". This is needed to be able to log in as a different user than root later.

You now have root SSH access though the Admin account. (You disabled that account earlier, but root is still active, and its password is the same as the NAS's admin account's password.) Let's try it from the command-line: `ssh root@diskstation`. You have to type 'yes', then the admin password, and you're in. Nice.

You want to be able to log in using the new user account you defined earlier. From a root SSH session:

* Open `/etc/passwd` with vi.
* Find the line for your user, and at the end of that line, replace `/sbin/nologin` with `/bin/sh`.
* Run `chmod u+s /bin/busybox`, or else you won't be able to gain root access later. (You'll get errors like "must be suid to work properly" when you try to run `su`.) In fact, I've lost root once after an update because the permissions of this file were reset, so I invite you to add it to the crontab as well:
* `vi /etc/crontab`, then add the line `0 0 * * * root chmod u+s /bin/busybox`. That will restore the permission every day at midnight.
* Restart the diskstation to make sure that the new crontab is loaded. You may want to test this to convince yourself it works, before you proceed, because if something goes wrong, you'll be locked out of your NAS.

Now, you really should disable direct root access. Perform the following steps carefully, or you might get locked out of root forever.

* `vi /etc/ssh/sshd_config` and change the line `#PermitRootLogin yes` to `PermitRootLogin no`.
* Let's also change `#MaxAuthTries 6` to `MaxAuthTries 3`.

If you want to beef up security a bit more with SSH keys, read [this article](http://www.eldemonionegro.com/blog/archivos/2012/08/19/how-to-securely-activate-ssh-into-your-synology-diskstation-with-ssh-keys-and-no-root-login).


Enable special characters
---
Not all Synology diskstations support UTF-8 on the terminal, which is where you want to run rsync. So if you have files or folders with non-ASCII characters in them, you may want to make sure that your diskstation does support them, or else you'll run into trouble later.

SSH into your diskstation and type `locale`. If it looks like this, you're good:

    LANG=en_US.utf8
    LC_CTYPE="en_US.utf8"
    LC_NUMERIC="en_US.utf8"
    LC_TIME="en_US.utf8"
    LC_COLLATE="en_US.utf8"
    LC_MONETARY="en_US.utf8"
    LC_MESSAGES="en_US.utf8"
    LC_PAPER="en_US.utf8"
    LC_NAME="en_US.utf8"
    LC_ADDRESS="en_US.utf8"
    LC_TELEPHONE="en_US.utf8"
    LC_MEASUREMENT="en_US.utf8"
    LC_IDENTIFICATION="en_US.utf8"
    LC_ALL=en_US.utf8

I.e., `en_US.utf8` should be everywhere. If it's not, go to [this blog post](http://www.chainsawonatireswing.com/2012/01/08/set-up-the-synology-diskstation-ds411j-to-support-utf-8/) and follow the instructions to fix it. It's a bit of work, but it's worth it, I promise.


Enable rsync
---
* Go to the diskstation's main menu and start the app "Backup & Replication".
* Go to the Backup Services tab.
* Check "Enable network backup service".
* Click Apply.

Now you can rsync from the pc to the NAS like this (assuming your shared folder is called `documents`): 

    rsync --delete -av --exclude ".DS_Store" --exclude ".fseventsd"
          --exclude ".Spotlight-V100" --exclude ".TemporaryItems"
          --exclude ".Trashes" --exclude "@eaDir"
          /some/directory/ user_name@diskstation:/volume1/documents


Install CrashPlan
---
* Open the Package Center's Settings, and change the Trust Level to "Any publisher".
* Just follow [Scott Hanselman's instructions](http://www.hanselman.com/blog/UPDATED2014HowToSetupCrashPlanCloudBackupOnASynologyNASRunningDSM50.aspx). He explains it much better than I could. Note that, if you're on a Mac, CrashPlan's configuration file can be found in `/Applications/CrashPlan.app/Content/Resources/Java/conf/ui.properties`

The CrashPlan client prevents the diskstation from going into hibernation mode. Therefore, it may be a good idea to schedule it to run only for a few hours a day. Log in via SSH and gain root via `su`. Then `vi /etc/crontab` and add the following two lines:

    0 1 * * * root /var/packages/CrashPlan/scripts/start-stop-status start
    0 5 * * * root /var/packages/CrashPlan/scripts/start-stop-status stop

That starts up the service at 01:00 and shuts it back down at 05:00.


Wrapping up
---
Restart the diskstation one final time, to make sure all settings have taken effect.

That's it! We're done.
