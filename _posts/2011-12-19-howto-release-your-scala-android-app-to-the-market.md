---
title: "How to: Release your Scala Android app to the Market"
excerpt: In which I show how I added my app to the Android Market.
tags:
- android
- code
- how-to
- scala
---
This post is a follow-up to my previous one,
[How to: Android development in Scala]({% post_url 2011-10-16-howto-android-development-in-scala %}).
I'm sure it has inspired you to write an awesome Android app in Scala. In this
post, I'll explain how to release your app to the Android Market.

Generate a key
--------------

First of all, you need to sign your app. For this, you will need a private key.
You can generate one using the `keytool` program which is supplied with
the JDK, as follows:

    keytool -genkey -v -keystore ~/.keystore -alias alias_name -keyalg RSA -keysize 2048 -validity 10000

This will save a `.keystore` file in your home directory, which contains all
kinds of super-secret private key stuff. Make a secure copy of this somewhere,
because you'll need it throughout your Android career!

Note that SBT will assume that the file is called `~/.keystore`. If you decide
to name it differently, SBT will not be able to sign your app!

Also, remember the alias that you choose on the command-line. You'll need it in
the next section.

For more information about keys and signing, see the
[Android Developers site](http://developer.android.com/guide/publishing/app-signing.html).

Update your build script
------------------------

Next, you have to update your build script, which is located in
`project/build.scala`. Find the line:

    keyalias in Android := "change-me",

and change it into

    keyalias in Android := "alias_name",

If you used a different `alias_name` when generating the keystore, modify this
line accordingly. If you still have an open SBT session at this point: exit
and restart, because it doesn't refresh your new build settings automatically.

Prepare your release
--------------------

In SBT's interactive prompt, run the following commands:

* `android:package-release` -- this will generate an unsigned, release-mode APK
  file.
* `android:prepare-market` -- this will generate a signed APK file, based on
  the one that was just generated.

Note: your password will be displayed in plain text on your screen, so don't do
this in a crowded room or anything.

In the text that `android:prepare-market` outputs to the console, you will see
the path to the APK file that you can upload to the Android market. It's in the
`target/` folder and it ends with `-market.apk`.

Release your release
--------------------

Now that you have a properly signed APK file, you can release your app. Go to
[Android Developer Console](http://market.android.com/publish) and follow the
instructions from there.

You will be asked to upload your APK file, and also to write a description,
upload some screenshots (at least 2), and upload some graphics to display in
the Market, so make sure you have these handy!

Once the app is uploaded, it's available (almost) immediately, although it
might take a while until it finally shows up in the search. If you're
impatient, you can access it directly using the following url:
`https://market.android.com/details?id=<package_name>` where `<package_name>`
is the Java package name for your app.

And that's it! You are now able to publish your app in the Android Market!

Additional note: updating your app
----------------------------------

So, your app has been on the Market for some time, and you're ready to release
version 2. Awesome! Here's how you do it.

* Write version 2, obviously.
* Next, you should enable
  [automatic manifest generation](https://github.com/jberkel/android-plugin/wiki/android-manifest-generation)
  if you haven't done so already: 
    * Remove the android:version and android:versionCode attributes from your
      AndroidManifest.xml
    * In `project/build.scala`, add a `versionCode` setting just below the
      `version` setting.
    * In the `AndroidBuild` object, add `AndroidManifestGenerator.settings` to
      the main project.
* Bump the `version` and `versionCode` settings. Remember that `versionCode` is
  for internal use only and must increase by at least one, each time you
  release. `version` is the number that users may see, and could even remain
  the same (although I don't recommend that).
* Then, re-build and sign your app as described above, and upload it!


Comments from my previous blog
------------------------------

#### [Stefan de Boey](http://posterous.com/users/5AB6ij4aTMSB) responded:

> cool, thanks<br>
> maybe you can contribute this to the wiki of the android-plugin? this kind of info seems to be missing on the wiki.

#### [Jan Ouwens](http://www.jqno.nl) responded:

> Hi Stefan, I'm glad you liked my post. I'll send the android-plugin maintainers a message. Thanks!
