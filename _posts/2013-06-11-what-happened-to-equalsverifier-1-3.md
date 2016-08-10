---
title: "What happened to EqualsVerifier 1.3?"
tags:
- equalsverifier
- travis-ci
- maven-central
---
Last weekend, I released two versions of EqualsVerifier. On Sunday afternoon, version 1.3; then version 1.3.1 about an hour later. What happened?

When I uploaded version 1.3 to Maven Central, something caught my eye in the confirmation mail: `"userAgent" = "Java/1.7.0_21"`. That's odd: I had expected to see a 1.6 version there. Just to be sure, I checked my Eclipse settings, and indeed, I had been building against Java 7. After the most recent Ubuntu upgrade, I must have only installed Java 7, without thinking about it.

I know many of EqualsVerifier's users are still using Java 6, so I downloaded that and recompiled. To my horror, there was a compile error. Turns out that Java 7 has added a new constructor overload to `java.lang.AssertionError` that accepts a cause. Turns out such a constructor does not exist in Java 6. Turns out that such a constructor is absolutely vital for my new and improved error reporting feature, which was the main reason for doing this release in the first place. Turns out, also, that Maven Central doesn't let you delete artifacts, even if they contain blatant compile errors.<sup>[[0]](#note0)</sup>

My adrenaline levels rose just about as fast as my heart sank.

Fortunately, my code was easily fixed: all `Throwables` have an `initCause` setter method that lets you initialize a cause. So I simply used that. But what to do with my broken 1.3 release? Fortunately, I hadn't sent out any announcements yet. So I simply pushed a new release (version 1.3.1) and announced that one instead. I think I actually may have gotten away with it.

But there is an underlying problem. Because I had been using a particular JDK (OpenJDK 7), I was unaware of an issue in my code on a different JDK (OpenJDK 6)<sup>[[1]](#note1)</sup>. And this case, though extreme, is not unique: I've had issues where EqualsVerifier would work in OpenJDK 6, but not in Oracle JDK 6<sup>[[2]](#note2)</sup>. Often, I would only find out about this kind of thing after a user was kind enough to report it.

This is where [Travis CI](https://travis-ci.org/) comes in: a continuous integration server in the cloud, free for open source projects. Not only do they support Java; they actually support three separate JDKs: OpenJDK 6, OpenJDK 7 and Oracle JDK 7. Awesome! And it was pretty easy to set up<sup>[[3]](#note3)</sup>. And sure enough, once I got Travis running, my build...failed!?

Turns out that some JDKs add a message to their `AbstractMethodErrors`, while others don't. And of course, one of my tests depended on a message in an `AbstractMethodError`.

Would I have found this without Travis CI? Probably. Eventually. At some point. ...maybe.

Fortunately, this issue only occurred in EqualsVerifier's own test suite, and doesn't affect the runtime behaviour. In other words: EqualsVerifier 1.3.1 was fine, it didn't need _another_ hotfix. The issue was quickly dealt with, and now, I can proudly say that for the first time ever, EqualsVerifier is _confirmed_ to work _and_ build on _three_ different JDKs! And to make sure it stays that way, I put in place some safeguards:

* I will receive an e-mail from Travis CI every time one of the builds breaks;
* I have added an explicit step to the release procedure to actively check the Travis CI build status; and
* I have added a badge to the sidebar of the [EqualsVerifier website]({{ pcurl('equalsverifier') }}), so everybody can see the current build status: <a href="https://travis-ci.org/jqno/equalsverifier"><img src="https://travis-ci.org/jqno/equalsverifier.png" alt="Build Status" style="border:0;"></a>

So from now on, broken EqualsVerifier releases because of JDK-dependent issues will forever be a thing of the past!<sup>[[4]](#note4)</sup>

* * *

<a id="note0"></a>
<small><sup>[0]</sup> Ok, so technically, it's not a compile error, but a `NoSuchMethodError`, and it only happens at runtime. And technically, it's only an issue in Java 6 and below. But still: quite the showstopper!</small>

<a id="note1"></a>
<small><sup>[1]</sup> Of course I could have officially dropped Java 6 support. But that's the kind of decision I want to make consciously (and a long time from now), not at the drop of a hat because of some silly mistake.</small>

<a id="note2"></a>
<small><sup>[2]</sup> EqualsVerifier uses a lot of reflection. While OpenJDK and Oracle JDK must outwardly support the same API, they have small differences of implementation that sometimes cause subtle differences in behaviour when you start reflecting into their classes. See, for example, issues [55](https://code.google.com/p/equalsverifier/issues/detail?id=55), [77](https://code.google.com/p/equalsverifier/issues/detail?id=77) and [81](https://code.google.com/p/equalsverifier/issues/detail?id=81).</small>

<a id="note3"></a>
<small><sup>[3]</sup> EqualsVerifier's build uses ANT with Ivy, which requires a little extra tweaking. Fortunately, that too, was [easy](http://ecmendenhall.github.io/blog/blog/2013/05/28/two-travis-ci-solutions/).</small>

<a id="note4"></a>
<small><sup>[4]</sup> Famous last words...</small>
