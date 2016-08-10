---
title: "Get to grips with your build, with Gradle"
tags:
- gradle
- java-magazine
---
Last month, the Dutch [Java Magazine](http://www.nljug.org/magazine/edities/editie-2015-3) published an article written by [Hanno Embregts](https://twitter.com/hannoloeloe) and myself. In it, we describe how we used Gradle in our project at the [Dutch Railways](http://www.ns.nl).

It's a big, monolithic project with &plusmn; 25.000 lines of Ant scripts, which we wanted to break up into separate components that can be built and released separately. Doing this with Ant would add more complexity to an already much too complex system of scripts. Maven wasn't a good fit either, because being built with Ant since its inception, our project didn't follow Maven's conventions, and shoe-horning it into these conventions would be impractical. Gradle provided the right mix of structure and flexibility.

We started our migration on a small subcomponent with no dependencies to other parts of our codebase, to work our way up. That way we wouldn't have to deal with deploying the moving target of our project, which was actively in development during the migration, into Nexus so our Gradle scripts could consume it. We tried it; it didn't work. It's much easier to go the other way round, to consume artifacts from Nexus that were already migrated to Gradle and put under a strict regimen of [Semantic Versioning](http://semver.org/).

Still, our existing Ant scripts needed to deal with our new Gradle-built artifacts. We decided it would treat these like any other 3rd party dependency, which worked fine for the most part. However, we found we had to be careful that the content of artifacts was identical to the ones built by the original Ant scripts, especially in the case of MANIFEST.MF files and various kinds of XML files embedded in the JARs. We had tools to test this, and we even went so far as to change Gradle's unit test output layout to match that of Ant, so we could compare those easily using a diff viewer.

In the end, we were very happy with the result, even if it took more effort than we expected initially. If you want to know more about the reasoning behind some of the choices we made, or how we solved some of the other issues we ran into, you can read the full article in the [physical magazine](http://www.nljug.org/magazine/edities/editie-2015-3), or you can read it [online](http://www.nljug.org/databasejava/grip-op-je-buildproces-met-gradle/). Note: it's in Dutch.
