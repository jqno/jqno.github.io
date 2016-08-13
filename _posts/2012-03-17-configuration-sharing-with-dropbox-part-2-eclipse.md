---
title: "Configuration sharing with Dropbox, Part 2: Eclipse"
tags:
- configuration
- dropbox
- eclipse
- how-to
---
This article explains how you can easily share your Eclipse configuration
between workspaces.

Introduction
------------

Since Eclipse's preferences are always to a workspace, you need to reset all
your preferences each time you start a new workspace, which can be a lot of
work. Especially if you have a memory like mine. Therefore, sharing
configurations is useful even without the added cloudiness of Dropbox. Maybe
you have only one development machine anyway. However, it's still worthwile
storing your configuration in Dropbox, if only for the automagic backups it
makes. So I recommend that you read
[Part 0]({% post_url 2012-03-11-configuration-sharing-with-dropbox-part-0-introduction %})
anyway.

One-time set-up
---------------

* Create a folder called `eclipse` inside your `~/Dropbox/config`. That's it. 

Per Eclipse set-up
------------------

For each of your Eclipse installations, whether they be on different computers,
or different versions of Eclipse on the same machine, you will need to do the
following:

* Install the [Workspace Mechanic](http://code.google.com/a/eclipselabs.org/p/workspacemechanic/)
  plugin. You will see a green check box icon somewhere in Eclipse's status
  bar.
* Record some settings!
    1. First, open up Window|Preferences, and press OK. This flushes any
       default settings that might otherwise get cluttered up into your shiny
       new shared preferences.
    2. Right-click the Workspace Mechanic icon in the status bar, pick
       Preference Recorder, and Start Recording.
    3. Open up Window|Preferences again and change a setting.<br>Alternatively,
       right clicking in various places often shows preferences that you can
       change. For example, clicking in the margin of any editor view will let
       you toggle "Show Line Numbers".
    4. Press OK to close the Preferences again.
    5. Right-click the Workspace Mechanic icon and Stop Recording. You will get
       a popup window that will let you describe your new preference.
    6. Check that the list box only contains keys that are relevant to the
       change you just made. Otherwise, try to execute step 0 again.
    7. Fill in a name and a description.
    8. Browse to `~/Dropbox/config/eclipse` and fill in a file name. The
       customary file extension for this kind of files is `.epf`.
    9. Save your setting.

I recommend that you repeat steps 2-8 for each individual preference you have,
rather than lumping them all in one big preferences file. That makes it a lot
easier to clean up unwanted preferences later on, and also to share them with
your friends and co-workers.

Also, I recommend that you choose your names, descriptions and file names
carefully. You will end up with lots of small preference files, and this way it
will be a lot easier to identify them.

Per workspace set-up
--------------------

* In Window\|Preferences, find the Workspace Mechanic node.
* Under Task Sources, click the New button.<br>
  ![Eclipse window preferences](/images/2012-03-17-configuration-sharing-with-dropbox-part-2-eclipse/eclipse-window-preferences.png)
* Navigate to `~/Dropbox/config/eclipse` and press OK.
* Press OK to close Window|Preferences, and you will see a small popup window
  telling you that the Workspace Mechanic has found issues.<br>
  ![Eclipse Workspace Mechanic](/images/2012-03-17-configuration-sharing-with-dropbox-part-2-eclipse/eclipse-workspace-mechanic.png)
* Press View and correct configuration issues.
* You can now review all individual preferences. Or you can just press OK.<br>
  ![Eclipse fix issues](/images/2012-03-17-configuration-sharing-with-dropbox-part-2-eclipse/eclipse-fix-issues.png)

Your preferences are now loaded!

A cool feature of Workspace Mechanic, is that it will periodically scan your
preferences directory to see if new preferences have been added. So if a friend
sends you an `.epf` file and you drop it into the preferences directory, you
will automatically get a notification!

Advanced trick
--------------

Instead of having just one `config/eclipse` folder, you can have several! For
example, if you often switch between Java and Scala development, like I do, you
can create the following folders: `config/eclipse/general`,
`config/eclipse/java` and `config/eclipse/scala` and only load the relevant to
the Workspace Mechanic. That way, I can have Ctrl+N create a new Java class in
a Java workspace, and a new Scala class in a Scala workspace.

