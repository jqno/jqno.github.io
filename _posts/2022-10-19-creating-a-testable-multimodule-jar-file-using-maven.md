---
title: "Creating a testable multi-module jar file using Maven"
excerpt: "In which I describe how I implemented EqualsVerifier's multi-release jar file build using Maven, and discuss the issues I had to solve to get there."
tags:
- equalsverifier
- java
- maven
- multi-release-jar-file
---
_Disclaimer: this post is not a tutorial, although I will link to relevant pieces of code. It's more of a retrospective that could provide a starting point for discussing multi-release jar files in Maven, or (more likely) for fixing my crappy scripts 😅. I'm writing this as a result of the Apache Maven BoF session at Devoxx 2022, where we discussed all of the below, and decided a blog post would be a nice jumping-off point for future improvements._

## Introduction

EqualsVerifier targets JDK 8, but has to support certain features from more recent JDKs as well, most importantly records and sealed types. For records, I can get away with using reflection to check whether a class is a subtype of `java.lang.Record` (pre-JDK 16, that check would simply return `false`), but to properly support sealed types, EqualsVerifier needs to do a bunch of calls to new-to-JDK 17 methods on the `Class` type. This can be done through reflection as well, but the resulting code would be slow and extremely ugly. Also, it needs to be tested thoroughly. The build already runs on a nice selection of JDKs via GitHub Actions, but I do need to be able to write tests targeting JDKs both with and without the feature.

Fortunately, JDK 9 introduced a feature called multi-release jar files, where a class's implementation can differ depending on the JDK version it runs on. So, EqualsVerifier can have a `SealedTypesHelper` class that returns `false`s and `null`s on JDKs 8 through 16, and that has a proper implementation on JDK 17. The thing is, how to set this up with Maven? (And no, I won't switch to Gradle.)

Note that, while I have been "the build guy" in many teams, I was certainly no expert at Maven when I started this project: a lot of the resources on Maven's website went way over my head when. This includes, unfortunately, their [official page on multi-release jar files](https://maven.apache.org/plugins/maven-compiler-plugin/multirelease.html). Coming back to that page now that I've finished this project, I understand it a lot better, but now it's too late 😉.

I've asked [a question on StackOverflow](https://stackoverflow.com/q/70541340/127863) and even got [Karl Heinz Marbaise](https://twitter.com/khmarbaise) (now chairman of the Apache Maven Project) to help me out, for which I'm very grateful. I went through three approaches, eventually settling on the last one, but they each have drawbacks. I have come away with the feeling that all this is more complicated than perhaps it could be, although I freely admit that my use case is perhaps not a typical one.

Let's discuss each of the approaches:

- [Tweaking the Compiler plugin](#compilerplugin)
- [Leveraging the Failsafe plugin](#failsafeplugin)
- [A multi-module build](#multimodule)

<a id='compilerplugin'/>
## Tweaking the Compiler plugin

In this approach, all I need to do is configure `maven-compiler-plugin` as described by [this Baeldung post](https://www.baeldung.com/maven-multi-release-jars).

I've tried this approach before asking my StackOverflow question. It's easy to set up, but has two downsides. First, IDEs don't recognise the different JDK versions, resulting in lots of false compilation errors and red squigglies in the code. The second, more important issue is that it's impossible to get a similar multi-module thing going for my unit tests, which means that I'm unable to unit test the EqualsVerifier features related to records and sealed types: a dealbreaker.

<a id='failsafeplugin'/>
## Leveraging the Failsafe plugin

Robert Scholte (Marbaise's predecessor as Apache Maven chair) was first to [answer to my StackOverflow question](https://stackoverflow.com/a/70581719/127863). He suggested testing the jar using `maven-failsafe-plugin` during the `verify` phase instead of `maven-surefire-plugin`. Using annotations like `@EnabledOnJre`, I can select the right tests for the right JDK.

My knowledge of Maven failed me here: when I read this answer, I didn't understand it at all, probably because I didn't understand the difference between Surefire and Failsafe. (They're both for running tests, but Surefire is for unit tests and Failsafe for integration tests, but you can run integration tests with Surefire, too, so...)

Coming back to it now, I think the idea is that Failsafe operates on the final multi-release jar file, rather than a directory of `.class` files, which is what Surefire does. However, [the docs](https://maven.apache.org/surefire/maven-failsafe-plugin/index.html) don't really say if this is indeed the case. If Scholte says it is, it's probably true though, and of course I could run an experiment to find out. In that case, this would indeed be a fairly easy solution.

However, it still has the disadvantage that IDEs don't recognise it, and will produce false compiler errors.

<a id='multimodule'/>
## A multi-module build

Karl Heinz Marbaise suggested to go multi-module. This will fix the IDE issue once and for all, because I can make a Maven submodule for each JDK version, and have the JDK-specific code in the corresponding submodule.

I have created [profiles](https://github.com/jqno/equalsverifier/blob/5fff957394ca8348257cba96bd4f5ef6cc5b1599/pom.xml#L523-L574) for each JDK version that includes the appropriate submodules for that JDK, so it's easy to test everything.

I think this is the cleanest and 'purest' solution, but it has some significant drawbacks:

1. The biggest one is that I have some test helpers that I need to share between submodules. The dependencies flow like this: `core <- testhelper <- {core-tests, 11-tests, 16-tests, 17-tests}`. This means that I had to extract them to a new submodule and move the main test suite into a separate submodule as well. When I had done this, it turned out that [PITest](http://pitest.org/) (which I use) [doesn't support this](https://github.com/hcoles/pitest/issues/323#issuecomment-267029825). I ended up folding core, test and testhelpers back together and living with the fact that some test helpers [end up in the final jar](https://github.com/jqno/equalsverifier/tree/5fff957394ca8348257cba96bd4f5ef6cc5b1599/equalsverifier-core/src/main/java/nl/jqno/equalsverifier/internal/testhelpers), since I was unable to break the circular dependencies otherwise. (Note that there's a [Maven plugin to work around this](https://github.com/STAMP-project/pitmp-maven-plugin), but it requires that all submodules be published; see point 4 below.)
2. Another [new submodule](https://github.com/jqno/equalsverifier/tree/5fff957394ca8348257cba96bd4f5ef6cc5b1599/equalsverifier-release-main) must be introduced to build the final jar file using the `maven-assembly-plugin`.
3. This approach does [not automatically produce `-sources.jar` and `-javadoc.jar`](https://github.com/jqno/equalsverifier/issues/598): I have to [copy them over](https://github.com/jqno/equalsverifier/blob/5fff957394ca8348257cba96bd4f5ef6cc5b1599/equalsverifier-release-main/pom.xml#L105-L159) using the `copy-rename-maven-plugin` from the core submodule's `target` folder if I want to publish to Maven Central.
4. I also have to deal with having a bunch of submodule jar files that should _not_ be published to Maven Central. I ended up [skipping that for all submodules by default](https://github.com/jqno/equalsverifier/blob/5fff957394ca8348257cba96bd4f5ef6cc5b1599/pom.xml#L66-L67), and [enabling it again](https://github.com/jqno/equalsverifier/blob/5fff957394ca8348257cba96bd4f5ef6cc5b1599/equalsverifier-release-main/pom.xml#L25-L26) in the submodule that builds the final jar file.
5. I also decided that this would be a good time to start releasing EqualsVerifier both as a regular jar file and a 'fat' jar, i.e. a jar with no dependencies. This is really out of scope for this post, but it explains why there are three `*-release-*` submodules, rather than one: the regular jar ([`-release-main`](https://github.com/jqno/equalsverifier/tree/5fff957394ca8348257cba96bd4f5ef6cc5b1599/equalsverifier-release-main)), the 'fat' jar ([`-release-nodep`](https://github.com/jqno/equalsverifier/tree/5fff957394ca8348257cba96bd4f5ef6cc5b1599/equalsverifier-release-nodep)), and one with a shared configuration for the `maven-assembly-plugin` ([`-aggregator`](https://github.com/jqno/equalsverifier/tree/5fff957394ca8348257cba96bd4f5ef6cc5b1599/equalsverifier-aggregator)), which also merges code coverage reports. Because all of this is rather fiddly, I also added [a module that actually unit tests the produced jar files](https://github.com/jqno/equalsverifier/tree/5fff957394ca8348257cba96bd4f5ef6cc5b1599/equalsverifier-release-verify); more on that [here]({% post_url 2022-03-30-unit-testing-a-maven-build %}). This was actually a lot of fun to do 😄!
6. Finally, there's the small matter of the `flatten-maven-plugin`. Because I use `maven-assembly-plugin` to produce a jar file, I can now use `flatten-maven-plugin` to [massage the pom file](https://maven.apache.org/surefire/maven-failsafe-plugin/index.html) that is uploaded to Maven Central a little bit, and remove everything that's only relevant for building EqualsVerifier, but not for end-users consuming it from Maven Central. This works [beautifully](https://search.maven.org/artifact/nl.jqno.equalsverifier/equalsverifier-nodep/3.10.1/pom) for the 'fat' jar, but [not so much](https://search.maven.org/artifact/nl.jqno.equalsverifier/equalsverifier/3.10.1/pom) for the regular pom: I'm unable to configure it in such a way that I can remove the dependencies on the unpublished JDK-specific submodules, without also removing the actual required dependencies. Fortunately, by adding `<scope>provided</scope>` and `<optional>true</optional>`, it's possible to make Maven _not_ try to download any unpublished, intermediate jar files for submodules such as `equalsverifier-16` in consuming projects.

## Conclusion

In the end, I was able to make everything work as I want, but it took a lot of after-hours work _and_ the help of some world-renowned Maven experts 😅, and still I had to make some sacrifices, such as the shared testhelper module and the clean-but-still-messy published pom file for the regular jar release.

I can't help but wonder if there shouldn't (couldn't?) be some easier way to achieve what I want: to use Maven to produce a multi-release jar file that I can test, without false-positive compiler errors. Maybe that way already exists, but I didn't find it because of a lack of documentation. If so, I would love to help create that documentation. Maybe this post can be a first step in that direction.
