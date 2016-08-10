---
title: "How to: make a Maven build script for Android"
tags:
- maven
- android
- how-to
---
I've written before about [writing Android apps in Scala]({{ pcposturl(2011, 10, 16, "howto-android-development-in-scala") }}). Sometimes, though, Scala isn't the answer, and you need to fall back on Java. That doesn't necessarily mean you should also fall all the way back to Ant, though. This post is about creating a Maven build script for an Android app written in Java.

But...why?
----------
In order to have a repeatable build, one that you could also run from a build server, you need a build script. Building and deploying from Eclipse might work fine if you're a single person on a fire-and-forget project, but once you're working with more than one person, or if you need to go back to a project you were doing a few months ago on a different version of Eclipse, you're going to want to have a proper build script.

But Android already provides an Ant script, why would you want to use Maven if you need to install and configure lots of stuff to it? Isn't Ant good enough?

Yeah, for a simple project, it probably is. But once your project gets more complicated, it's nice when Maven has your back. Especially if you have to manage dependencies (other than Android), Maven can be a life-saver. And this is coming from someone who has no particular love for Maven. Between Ant and Maven, Maven is, quite simply, the lesser of two evils.

Installing all the stuff
------------------------
First, download Eclipse. I recommend Eclise Juno (4.2). If you use Indigo (3.7), you will need to install the [M2Eclipse](http://www.eclipse.org/m2e/download/) plugin as well.

**NOTE**: are you running Linux on a 64 bit machine? Then do this first:
    sudo apt-get install ia32-libs
It's somewhere in Google's documentation that you should do this, but it's easy to miss and it took me some googling before I figured this out.

Next, install the [Android ADT plugin](http://developer.android.com/sdk/installing/installing-adt.html). It comes with a nice wizard that downloads the SDK for the platform(s) of your choice and helps you set up an emulator.

Then, you need to install m2e-android. The install process is described on [m2e-android's website](http://rgladwell.github.com/m2e-android/). Don't have the Eclipse Marketplace installed? You can get it [here](http://www.eclipse.org/mpc/).

A new project
-------------
Instead of creating a new Android Project, you should create a new Maven Project. The first time, you need to configure a Maven archetype. Don't worry, this is super easy and the [m2e-android website](http://rgladwell.github.com/m2e-android/) explains very well how to do it. It also explains how you can convert an existing Android project to Maven.

Congratulations! You're good to go now.

Running Maven from the command-line
-----------------------------------
As I explained above, you might want to run your build on a build server. If you do, or even if you simply want to run your build from the command-line, you need to do a couple extra things.

First, you need to install [Maven 3](http://maven.apache.org/download.html), since it won't work with Maven 2. Also, make sure it's on your `PATH`. Check this by running `mvn -version`; it should say something like `Apache Maven 3.0.4`; not `Apache Maven 2.2.1` (or some other version in the 2.x.y range).

And while you're fiddling with environment variables, make sure that the `ANDROID_HOME` variable is set correctly as well. If you don't know where your SDK is located, open Eclipse's _Preferences_ screen from the _Window_ menu. You can find the SDK location in the _Android_ tab.

To make things easier (even though, technically, it's cheating), you can add your ADV to the Maven `pom` file. That way, Maven will automatically deploy your app to the given ADV, so you don't have to provide it manually every time you run Maven. In the `<build><plugins>` section, find the `android-maven-plugin`. It has a `<configuration>` section, where you can add the following lines:

<pre class="prettyprint">
&lt;emulator>
  &lt;adv>my_adv_name&lt;/adv>
&lt;/emulator>
</pre>

Obviously, you need to replace `my_adv_name` with the name of an ADV that you have actually configured.

Once all this is out of the way, you can finally run Maven from the command-line! Useful commands include:

    mvn clean install
    mvn android:deploy
    mvn android:run

Be sure to run `mvn clean install` at least once before running `mvn android:deploy` or `mvn android:run`, or else you might start to see weird error messages about "trying to figure out package name from inside apk file".

Conclusion
----------
That's it! Good luck! And if I missed something, please let me know. This kind of instructions tends to change over time, so while this works at the time of writing (which is September 16th, 2012), it might no longer work at the time of reading (which is, well, now).
