---
title: "How to: Android development in Scala"
excerpt: In which I explain how I wrote an Android app in Scala.
tags:
- android
- code
- how-to
- scala
---
I recently decided to write an Android app, and I decided to do it in Scala.
Android uses its own VM (Dalvik), but Dalvik bytecode is generated from Java
bytecode, so it should be possible to write Android apps using any language
that compiles to Java bytecode. However, the tooling that Google has provided
is geared pretty much to Java, so you may have to jump through a couple of
hoops to get things working for other languages. In this article, I describe
the toolchain that I use for developing Android apps in Scala. I have only 3
disclaimers.

* Disclaimer 1: this process works at the time of writing, which is  October
  16th, 2011. By the time you read this, things may have changed.  For the
  better, I would hope, but still, this article might be out of  date.
* Disclaimer 2: I was unable to get Eclipse to work, unfortunately. So continue
  only if the command-line doesn't scare you.
* Disclaimer 3: I use Ubuntu 11.10, so you might have to adjust for your OS if
  you use another. I'll assume you already know how to do things like add
  folders to the path.

Ok, those were the disclaimers. Let's go:

Preliminaries
-------------

* In your home folder, create the folders `dev` and `bin`. In the `dev` folder,
  we'll be putting all the development tools. Choose a different name or
  location if you prefer. The `bin` folder will contain some small executables.
  One of the steps will automatically generate this folder, so you might as
  well use it for the rest as well.
* Put `~/bin` on your path.

Android
-------

* First, to install the Android SDK, follow
  [these instructions](http://developer.android.com/sdk/installing.html).
  You can skip the steps related to Eclipse.
* You can download the SDK to the `~/dev` folder you just created. The unzipper
  will automatically extract the contents to a subfolder called
  `android-sdk-linux_x86`.
* Run `~/dev/android-sdk-linux_x86/tools/android` to download a platform
  (Android 2.2 or 2.3, etc.) Don't download all platforms at once, or be
  prepared to wait a long time.
* Also, create a Virtual Device. You will need it later to deploy. I called
  mine `avd`.
* Add `~/dev/android-sdk-linux_x86/tools` and
  `~/dev/android-sdk-linux_x86/platform-tools` to your path.
* If you run 64-bit Ubuntu (as I am), run `sudo apt-get install ia32-libs`. I
  know, it says to do that at the bottom of the Android installation
  instructions, but who reads that shit? I certainly didn't... which is why I
  mention it here.)

Scala
-----

* Make sure you have a recent JDK installed on your system.
* Install Scala. I'm using Scala 2.9 at the moment. Your system may have a
  package for it; if not, you can download it
  [here](http://www.scala-lang.org/downloads).
* Next, we need SBT. Every self-respecting language needs its own build tool
  nowadays, and SBT is Scala's. To install, follow
  [these instructions](https://github.com/harrah/xsbt/wiki/Setup).
  I'm using [version 0.11](http://typesafe.artifactoryonline.com/typesafe/ivy-releases/org.scala-tools.sbt/sbt-launch/0.11.0/sbt-launch.jar).
* I did one thing differently, which you may want to do as well:
    * I downloaded `sbt-launch.jar` to a new folder called `~/dev/sbt-0.11`
      instead of in `~/bin.`
    * This meant that the shell script needs to be adjusted, too:<br />`java
      -Xmx512M -jar ~/dev/sbt-0.11/sbt-launch.jar "$@"`

Scala + Android
---------------

* Next, we need the SBT Android plugin. This plugin adds the necessary Android
  capabilities to SBT.
  [The plugin's GitHub page](https://github.com/jberkel/android-plugin)
  contains installation instructions.
* The `giter8` thing looks a bit weird, but I recommend going along with it
  anyway: it will keep things easier. BTW, this is the step that creates the
  `~/bin` folder which I mentioned earlier. Do create the project using the `-b
  sbt-0_11` switch.
* At the time of writing, it's still necessary to `git clone` android-plugin's
  repository and `sbt publish-local` it. I recommend cloning it into our trusty
  `~/dev` folder, to keep things clean.
* Once you run SBT the first time, it will start downloading and compiling teh
  internets, so this might take a while.

Code!
-----

* While SBT is running, you can start the emulator. It, too, needs some time to
  start up. Assuming you called your Virtual Device avd, like I did, you can
  use the following command-line:

    `emulator -avd avd &`

* Add the following folders to your `.gitignore` file (or equivalent thing for
  your favourite SCM tool):

      lib_managed/
      src_managed/
      project/boot/
      project/build/target/
      project/plugins/target/
      project/plugins/lib_managed/
      project/plugins/src_managed/

* I like to keep SBT running in a terminal window, so I can quickly type
  instructions into the interactive prompt:
    * `compile` compiles the code
    * `test` runs the unit tests
    * `android:package-debug` creates an Android APK file with debug
      information.
    * `android:package-release` creates an Android APK without debug
      information.
    * `android:start-emulator` deploys the APK file to the emulator.
    * `android:start-device` deploys the APK to your device, if you have it
      connected.

* Note that `android:start-emulator` and `android:start-device` don't
  automatically compile and package your code. It happened to me a couple of
  times that I made a change and deployed it without packaging it first, and I
  would wonder why I couldn't see the changes...
* If you run `android:start-emulator` and SBT says "`error: device offline`",
  even though the emulator is running, just wait a few seconds and try again.
  It needs some time to warm up or something.

That's it! Enjoy! And if you find out how to do all this in Eclipse, please let
me know in the comments.
