---
title: "Looking back on 15 years of EqualsVerifier"
tags:
  - equalsverifier
excerpt: In which I take a trip down memory lane.
---
Today, I celebrate EqualsVerifier's 15th birthday (observed) with this retrospective blog post. It's a long read, but then, 15 years is a long time too!

In those 15 years, I met and married my wife, had a daughter who now goes to school, moved house three times, visited twelve countries across three continents, lived through a global pandemic, changed jobs four times...a lot happened.

I'd like to take a moment to look back, and write down some of my memories from the project. I'll start with EqualsVerifier itself, and finish by talking a bit about the people around the project.

## Table of contents

This is a long post; feel free to skip to the sections that sound most interesting to you!

- [The evolution of EqualsVerifier](#evolution)
    - [The early beginning](#beginning)
    - [0.x era](#0x-era)
    - [1.x era](#1x-era)
    - [2.x era](#2x-era)
    - [3.x era](#3x-era)
- [Java and other languages](#languages)
- [People](#people)
    - [Contributors and users](#contributors-users)
    - [Conference interactions](#conference-interactions)
    - [Honorable mentions](#honorable-mentions)
- [Personal life](#personal-life)
- [What would I have done differently?](#different)
- [What's next](#whats-next)
- [Conclusion](#conclusion)

<a id='evolution'/>
## The evolution of EqualsVerifier

<a id='beginning'/>
### The early beginning

15 years ago, I was working for [TOPdesk](https://www.topdesk.nl), and I was trying to introduce the practice of unit testing in my team. I noticed that testing an `equals` method was pretty hard to do right. You either had to write lots of screens of code, or skip over things.

I was reading a lot of programming books at the time, and J.B. Rainsberger's [JUnit Recipes](https://www.amazon.com/JUnit-Recipes-Practical-Methods-Programmer/dp/1932394230) pointed me to [GSBase](https://gsbase.sourceforge.net/index.html)'s EqualsTester. It was very nice, but it required that you implement `equals` with `getClass()` instead of `instanceof`. This is technically a violation of the [Liskov Substitution Principle](https://www.baeldung.com/cs/liskov-substitution-principle) and I was still young and idealistic enough to care about that. Also, it wasn't very thorough. There are plenty of ways in which the `equals` can be broken, and while it tested the most important ones, it skipped over many of them too.

In 2008, I attended Devoxx Belgium for the first time, and saw a talk about this cool new programming language, Scala. I bought the book and started reading it cover to cover, and promised myself I'd do a Scala side project once I'd finished reading the book. However, near the end of the book, for some reason there was a chapter on how you can write an `equals` method that could be overridden to add state while still fulfilling both the `equals` contract _and_ the Liskov Substitution Principle (you can read it online [here](https://www.artima.com/articles/how-to-write-an-equality-method-in-java)). Josh Bloch, in his [Effective Java](https://www.amazon.com/Effective-Java-Joshua-Bloch/dp/0134685997), said this was impossible, so this tickled me.

So one weekend, I started coding (in Java, not Scala) to see if I could apply what I'd learned from the Scala book to write a replacement for EqualsTester, maybe even applying some reflection tricks to make the API a bit nicer. And indeed, in two weekends I had something working. I called it EqualsVerifier, because the name EqualsTester was obviously already taken and I couldn't come up with something better.

I don't remember the exact date when I first started coding on EqualsVerifier, but [I first uploaded it on June 1st, 2009](https://github.com/jqno/equalsverifier/commits/main/?since=2009-06-01&until=2009-06-02), which is why I observe its birthday on that date.

I did, eventually, get around to that [Scala side project](https://jqno.nl/post/2012/09/11/foobal-predicting-soccer-matches-with-scala-and-drools/) I promised myself too, but it took a while. I also completely blew off a friend who wanted to create a game with me. I kept promising him that I'd get to it when EqualsVerifier was done, but here we are. Harald, if you're reading this, I'm sorry!

<a id='0x-era'/>
### 0.x era

What I uploaded on June 1st, 2009 was the source, dependencies, and binaries of version 0.1. It ran on Java 6. I used [Google Code](https://en.wikipedia.org/wiki/Google_Developers#Google_Code) to host the source code and the binaries, because it was the popular and obvious choice at the time. Google was still a reliable and exciting company where you could be sure that if it launched a product, it would be there [forever](https://killedbygoogle.com/). It was very modern, as it natively supported SVN for source control.

In the meantime, Roel, my co-worker and mentor at the time, was working with a friend on a small open source Java library of his own which you may have heard of, called Lombok. They were using this weird new GitHub thing that nobody had heard of yet. We had many conversations about our respective projects, which were incredibly helpful to me. Roel, if you're reading this: thanks for helping me when I was just starting out!

A typical call to EqualsVerifier, at the time, looked like this:

```java
Foo first = ...
Foo second = ...

EqualsVerifier.forExamples(first, second)
        .verify();
```

There was `.forClass()` too, but I wasn't confident enough yet that it was good enough to be able to generate all the necessary instances that EqualsVerifier needed. So I kept `.forExamples()` around as a fallback for a long time.

I was particularly proud of my Ant script that automatically uploaded artifacts to Google Code, so people could download the JAR file directly. However, people quickly started pestering me to add support for Maven, which was this new thing that started to get momentum. And the [very first contribution from somebody I didn't know](https://github.com/jqno/equalsverifier/issues/8) solved exactly that: it added a custom Maven repository to the SVN repository that people could add in their pom file's `<repositories>` section, and then they could automatically download EqualsVerifier from there.

<a id='1x-era'/>
### 1.x era

In 2011, I changed jobs and moved back to my home town. EqualsVerifier was already pretty stable at the time, so I decided it was time for the big 1.0. In this version, EqualsVerifier was able to detect annotations for the first time: there was support for `@NotNull`, `@Immutable`, and rudimentary support for JPA.

After some more pestering from people, I finally uploaded an EqualsVerifier release to Maven Central for the first time: version 1.0.2, in August of 2011, and every release since then was uploaded there. Including the [botched 1.3 release](https://jqno.nl/post/2013/06/11/what-happened-to-equalsverifier-1-3/), which will remain available for as long as Maven Central will be around since Maven Central is immutable.

I set up continuous integration with Travis CI in 2013, because I thought it would be nice if I could test EqualsVerifier against a bunch of different Java versions before each release.

In the meantime, EqualsVerifier was slowly spreading. It's very difficult to keep track of usage of an open source Java library. Maven Central provides some download numbers, but they don't say a lot: a company might download the artifact and put it in their cache and a hundred people could use it without me knowing about it; or a single person using three computers might download it on each, and it would count for three downloads. However, one obvious milestone was the first time someone [asked a question on StackOverflow](https://stackoverflow.com/q/19926486/127863) (they even introduced the `[equalsverifier]` tag for it!), I was pretty excited when I found it, until I discovered it had been posted a year earlier and I'd never even noticed...

Late in the 1.x era, I made some major changes to the build and distribution. In 2014, I finally switched from Ant to Maven, and I also began distributing EqualsVerifier as an "uberjar" (a single JAR file that contains all dependencies), after receiving several issues from people with version conflicts in EqualsVerifier's dependencies. The tool I used to create it, had the best name of any open source project I have ever seen: _Jar Jar Links_. Get it? It links jars!

Around this time, I also discovered a couple of really dirty hacks: [compiling at runtime](https://jqno.nl/post/2014/08/20/the-things-we-do-for-compatibility/) (which I used to support multiple Java versions from a single codebase) and [creating enum entries at runtime](https://jqno.nl/post/2015/02/28/hacking-java-enums/) (which I discovered by accident while debugging an EqualsVerifier issue). Both ended up in my [Don't hack the platform? ☠️💣💥](https://jqno.nl/talks/dont-hack-the-platform/) talk.

In 2015, Google announced that they were retiring Google Code. I was still pretty upset about the death of [Google Reader](https://en.wikipedia.org/wiki/Google_Reader) two years prior. Wait, who am I kidding? I'm still upset today, and since the demise of these two services (as well as Google Music and Google Wave), I refuse to start using any new service Google introduces, since they'll probably kill it off anyway by the time I'm really invested in the product.

Anyway, I moved EqualsVerifier over to GitHub. I'd moved the code there earlier, but I'd kept Google Code around for the issue tracker. Fortunately, I found a script that was able to migrate all the issues, so I was able to keep them for reference. This also meant the end of my cool script to upload artifacts to Google Code, but it didn't matter, because Maven Central was serving very well.

<a id='2x-era'/>
### 2.x era

In 2016, EqualsVerifier jumped to 2.0. There were a couple of breaking changes: I finally dropped support for Java 6, I changed the default behaviour for EqualsVerifier to always check that all fields are used unless disabled explicitly (before, this was an opt-in behaviour), and I replaced CGLib, the bytecode magic library, with Byte Buddy, which could do the same things but more and better and with much better support for new Java versions (turns out one of EqualsVerifier's users liked using Early Access versions and was not afraid to open issues 😄). I also introduced a lot of static analysis, using [Checkstyle](https://checkstyle.org/), [FindBugs](https://en.wikipedia.org/wiki/FindBugs) (later SpotBugs), and of course [PIT](https://pitest.org/) for mutation testing.

The main reason for the jump to 2.0, however, was the support for generics. Someone had an edge case where they would do this:

```java
public class Foo {
    private final List<Bar> bars;

    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Foo)) return false;
        Foo other = (Foo)obj;
        for (int i = 0; i < bars.size(); i++) {
            Bar a = bars.get(i);
            Bar b = other.bars.get(i);
            // Compare Bars
        }
    }
}
```

That would lead to a `ClassCastException`, because due to type erasure, EqualsVerifier didn't know the generic type of the `List`, and would just treat it like a raw `List` and put strings in that list which obviously can't be assigned to a variable of type `Bar`. This was generally ok though, because normally you would just call `bars.equals(other.bars)`, and `List`'s `equals` method would figure it out. There's no good reason to unwrap lists in an `equals` method like this, except with Android's `SparseArray` class which, for some bizarre reason, didn't implement `equals()`.

So I over the course of 4 months and 165 commits (yes, I kept track), I made two attempts (the first one failed), and finally, EqualsVerifier could overcome type erasure and figure out the appropriate types to populate the `List` (and any other generic data structure). I wrote about it in two parts: [part 1](https://jqno.nl/post/2016/06/23/generics-in-equalsverifier-part-1/) and [part 2](https://jqno.nl/post/2016/06/26/generics-in-equalsverifier-part-2/).

In this period, I also made huge improvements to the documentation. For instance, I added a full [manual](https://jqno.nl/equalsverifier/manual/) to the website while on vacation in Paris. And I got to give [a talk about EqualsVerifier at Devoxx Belgium](https://www.youtube.com/watch?v=pNJ_O10XaoM)! The recording is from 2017, but it's still relevant today.

[![Me presenting EqualsVerifier at Devoxx 2017](/images/2024-06-01-looking-back-on-15-years-of-equalsverifier/devoxx2.png)](https://www.youtube.com/watch?v=pNJ_O10XaoM)<br/>Me presenting EqualsVerifier at Devoxx 2017

<a id='3x-era'/>
### 3.x era

2018 already saw the release of EqualsVerifier 3.0. The main reason for this were some new features, like re-usable configurations and the `report()` function that allows integration with other tooling. I took the opportunity to drop Java 7 support (and replace _a lot_ of anonymous inner classes with nice lambdas) and improve error messages.

At some point I found out that Travis CI wasn't able to cache Maven artifacts, so every build required downloading the entire internet. That's time-consuming, polluting and pointless, so I decided to switch to Circle CI. Around that time, GitHub Actions was introduced too, and not long after, I moved my pipelines again, so I would have everything in a single place on GitHub. Later on, I also started using the excellent [JReleaser](https://jreleaser.org/) to run releases directly from GitHub Actions instead of from my local machine, as I was doing before.

In 2019, I introduced a code formatter, since I no longer wanted to have to thing about spacing and making everything look consistent. I was working with Scala at the time, and its excellent [Scalafmt](https://scalameta.org/scalafmt/) made me want to have similar capabilities for Java. At first, I picked the [Google Java formatter](https://github.com/google/google-java-format) for this task, but I never really liked it. True, it integrates nicely with Maven, and I like its philosophy of being unconfigurable so people can't bikeshed formatting settings. However, its decision to enforce a two-space indent in an ecosystem that has used 4 spaces since the 90s, is just _weird_. And while it does have an Android flag (which I used) to make it use 4 spaces, this also makes everything look really cramped. I found myself changing the logic of the code to make it look better after formatting. So I quickly switched to [Prettier Java](https://github.com/jhipster/prettier-java), a plugin to [Prettier](https://prettier.io/), which has saner defaults and results in code that's more readable. Its biggest downside is that it requires NPM to run, so I'm still looking out for something better. Please let me know if you know something! (And no, IntelliJ's built-in formatter is not better, since you can't run it from Maven.)

I did learn about [.git-blame-ignore-revs](https://www.michaelheap.com/git-ignore-rev/) though, which you can use to skip commits when doing a `git blame`. Very handy when you need to reformat all your code...

In 2022,  while doing maintenance on the build script, I noticed a missing entry in the `MANIFEST.MF` file. I don't remember if this actually got released or not, but I decided I needed to be able to "unit test" my build. Also, with the advent of new language features like records and sealed types, I wanted to step up my game of supporting multiple Java versions from a single code base, and start using multi-release JAR files. This resulted in a huge overhaul of the build scripts and the introduction of Maven submodules. I actually received a lot of help from [Karl Heinz Marbaise](https://twitter.com/khmarbaise) (current chairman of the Apache Maven project). Karl, if you're reading this: thank you!

The last big feature I added was improved JPA support, mainly because I started using JPA myself for the first time. Yes, this happened in 2022.

<a id='languages'/>
## Java and other languages

The funny thing is, for most of these past 15 years, I didn't even use Java for my main job. I had a long stint (more than 6 years) as a Scala developer (buying that book turned out a life-changing event!), I did C# for two years, and for one year my main languages were Ruby and [Jess](https://en.wikipedia.org/wiki/Jess_(programming_language)), which is a Prolog-like language with a Lisp syntax.

There was a time where I was seriously considering doing an EqualsVerifier.NET together with a colleague, since C#'s `Equals()` method has basically the same contract as the Java one. But then I found out that [someone already beat me to it](https://github.com/magicmonty/equalsverifier-net), although sadly this project didn't get very far.

While I was doing Scala, I could of course have been using EqualsVerifier. However, I never did—in all those years, I only had to write an `equals` method twice, because Scala's case classes made it virtually unnecessary to do so.

However, having EqualsVerifier as a side project did keep me grounded in the Java community, which was nice.

<a id='people'/>
## People

<a id='contributors-users'/>
### Contributors and users

In its 15 years of existence, 32 people (and 2 bots) have contributed to EqualsVerifier, which seems small if you compare it to other open source projects. But let's not fall prey to survivorship bias. This is actually really amazing for a project that I made to scratch my own itch, and that I never even actively advertised—at least, not until I first started speaking at conferences.

It's even more amazing when you start considering the people I interacted with along the way, who didn't necessarily make code contributions to the project.

136 people were thoughtful enough to open an issue on GitHub when they found a problem with EqualsVerifier, and this doesn't even include the issues that were raised back in the Google Code days. Some of these issues were easily solved. Some of these were a pain in the behind to deal with. But all of them made EqualsVerifier better, and I'm grateful for each of them. Especially for the hard ones!

But the most fun thing for me is finding out who are using EqualsVerifier, and where. As I mentioned before, there's no good way to track this, so I have to rely on people to tell me, which they sometimes do. GitHub issues are a nice source for this: people who open an issue, often have their employer and/or their country of origin in their profile. Another great source is friends and (former) coworkers who will sometimes let me know that their new company is using EqualsVerifier. The most fun occurrence of this happened recently, when a colleague started working for my first employer's biggest competitor and sent me a screenshot of a package statement and a few import statements, proving that EqualsVerifier was used there.

![Me presenting EqualsVerifier at Devoxx 2017](/images/2024-06-01-looking-back-on-15-years-of-equalsverifier/devoxx.jpg)<br/>Me presenting EqualsVerifier at Devoxx 2017

<a id='conference-interactions'/>
### Conference interactions

And then there's conference interactions. At Devoxx Belgium 2014, I saw a talk about [PIT](https://pitest.org/). I was immediately excited to try it out in EqualsVerifier, and said so on Twitter. I quickly got a reply from its creator, [Henry Coles](https://twitter.com/_pitest), who commented that would be a nice symmetry since he uses EqualsVerifier in PIT! Henry, if you're reading this: thank you for that tweet, for allowing me to test-drive Arcmutate, and for the chats at JavaZone in Oslo!

In 2016, when I was a Scala developer at Codestar, I gave [my first and only talk at a Scala conference](https://www.youtube.com/watch?v=W37Mp3mBYLw), the Typelevel Summit in Olso. In the introduction, I casually mentioned that I must be the only attendee who did Scala for their day job and Java as a hobby, which got a good laugh.

While at that same conference in Oslo, I was thinking about something a former colleague once said: don't be afraid of your heroes; they're people too! I remembered that [Rafael Winterhalter](https://twitter.com/rafaelcodes), creator of Byte Buddy, lived in Oslo, so I sent him a message and we ended up having lunch together. It was a nice experience. Rafael, if you're reading this, thank you for that, and also for helping me integrate Byte Buddy!

I also won't forget how, at Devoxx Belgium 2023, I was talking with two acquaintances in the exhibit hall, and somebody walked past, slowed down for a moment as he looked at me, yelled "EqualsVerifier!", and continued walking. I still have no idea who that was...

EqualsVerifier also comes up occasionally in conferences that I don't attend, because several kind people have included it in their talks. For example, [Johan Janssen](https://twitter.com/johanjanssen42)'s "[Java Hidden Gems](https://youtu.be/pH_mHWQmtDst?t=1160)", [Tom Cools](https://twitter.com/TCoolsIT)'s and [Elien Callens](https://twitter.com/Elien_Callens)'s "[Leaving a Legacy](https://youtu.be/rJ08SZEe36g&t=2795)" and [Michael Vitz](https://twitter.com/michaelvitz)'s "[Beyond Built-in: Advanced Testing Techniques for Spring Boot Applications](https://www.youtube.com/watch?v=vn9P38o03TQ&t=1346s)".

I found out about these, because friends who did attend have sent me pictures. In fact, I only found out about the last one _yesterday_ when Paco sent me a picture from Spring I/O 2024! So, thanks Jaap, Jan-Hendrik, Paco, and more, for sending me these pictures!

Johan, Tom, Elien and Michael, a special thanks to you for the shout-outs! It makes me really happy to know that I've built something that you like so much that you include it in your talks.

<a id='honorable-mentions'/>
### Honorable mentions

And then there are some other nice moments that happened as a result of EqualsVerifier:

- I was invited for an interview by the [BarCoding Podcast](https://anchor.fm/barcoding/episodes/Episode-20---The-tales-of-the-EqualsVerifier-project-e1993ev). Paulien, Arnout, if you're reading this: thanks for having me on the show.

- Tom Cools made a slick [getting-started video about EqualsVerifier](https://www.youtube.com/watch?v=ivRjf8yvVMk), which he graciously allowed me to link to. It's a great addition to the wall of text that is EqualsVerifier's website. Tom, if you're reading this, thank you for the video, for being a woefully underpaid de-facto devrel for EqualsVerifier, and for being great company at conferences!

- I also liked when J.B. Rainsberger, the very person who wrote the book that introduced me to EqualsTester, posted this tweet:<br/>[![J.B. Rainsberger: "All right. Goodbye. EqualsVerifier. I cannot handle your needles constraints."](/images/2024-06-01-looking-back-on-15-years-of-equalsverifier/needless-constraints.png)](https://twitter.com/jbrains/status/661556171166392320)<br/>It inspired me to find ways to improve the user experience for EqualsVerifier. One thing I did was to write a [manual](https://jqno.nl/equalsverifier/manual/), where I attempt to explain the reasoning behind the contraints and how to deal with them. Another is the introduction of the `simple()` method, which makes it easy to side-step some of them. J.B., if you're reading this, thanks for speaking up!

- Henry Coles named [his verifier test pattern](https://blog.arcmutate.com/the-verifier-test-pattern/) after EqualsVerifier, which I found amusing, because as I mentioned, I picked 'Verifier' only because the name EqualsTester was already taken. I never actually liked the word... Anyway, Henry, thank you for that one too!

- [Johan Janssen](https://twitter.com/johanjanssen42) wrote an [InfoQ article](https://www.infoq.com/news/2023/11/equalsverifier-jpa-entities/) about EqualsVerifier's improved JPA support. Johan, if you're reading this: thanks!

- Somehow, EqualsVerifier also made it to [Baeldung](https://www.baeldung.com/java-equals-hashcode-contracts).

- Finally, most of my past and current employers allowed me to work on EqualsVerifier on company time, for example during 10% time or R&D days. [TOPdesk](https://www.topdesk.nl), [Sioux](https://www.sioux.eu/), [Codestar](http://www.codestar.nl) (Ordina), [Yoink](https://yoink.nl/): thank you, I appreciate it!

<a id='personal-life'/>
## Personal life

For the first half of EqualsVerifier's existence, I was single and living alone, so I had a lot of free time to hack on EqualsVerifier. And I did. Of course after I met my wife and had a daughter, things started to change. I managed to clear every single GitHub issue, for the first time ever, a few days before our daughter was born, and for about a year I actually managed to keep up. EqualsVerifier was already pretty stable, there weren't a lot of issues, and the ones that did come in were relatively easy to solve.

Of course, when you have a family, your priorities shift (as they should!), and unfortunately so did my energy levels. As a result, the past years I haven't been able to spend as much time on EqualsVerifier as before. I haven't been able to add a lot of new features or do big refatorings recently. I still try to keep up with the issues, but I have some hard ones that have been there for more than a year. I do intend to get to them some day though!

<a id='different'/>
## What would I have done differently?

I've enjoyed my run of making EqualsVerifier, and I think the project has found its niche of happy users. Also, I'm not someone who regrets things. In life and work, I make decisions. Some of them go well, some of them don't, and every decision is an opportunity for learning and experiencing thing that I wouldn't have had if I'd made a different decision.

If I really have to name one thing, it's the mutable nature of EqualsVerifier's architecture. When EqualsVerifier instantiates an object, it mutates the state of that object for each test it performs. Then it has to be careful to restore the state for the next test. This has caused some gnarly debugging sessions over the years. If I had to rewrite EqualsVerifier today, I'd create a new instance for every test and then discard it.

Ironically, the Scala book that triggered it all already emphasized immutability. I just hadn't internalized that message yet when I started EqualsVerifier, and I've been stuck with that decision ever since.

<a id='whats-next'/>
## What's next

EqualsVerifier has been reasonably stable for some years now. There haven't been a lot of big new features, and most issues that people open pertain to prefab values, which I can deal with quickly. Every six months, I need to update the build pipeline to ensure everything still works on the latest and greatest Java version.

Still, part of the reason why I still enjoy working on EqualsVerifier, is that it's a place where I can do experiments and gain some deep knowledge about Java. That's why I plan to fully adopt the Java module system for the 4.0 release, which I'm hoping to start working on soon-ish. And if I'm doing 4.0, it's also time to bump the Java version again. I'm not sure yet if I'll go to 11, or all the way up to 17. I'm open to opinions on this matter.

Another thing I'd like to do, is to reduce the need for adding prefab values, both for the end user and internally, because that would save me a bunch of GitHub issues every year. I have some ideas on how to do that, but they're hard to implement and might not make it into 4.0 yet. They require a more immutable architecture, and in fact I've already started some refactorings in that area. But given my time constraints, we'll have to see how that goes.

<a id='conclusion'/>
## Conclusion

It's been fun looking back on these past 15 years. EqualsVerifier has gone through a lot of changes, and has matured a lot. And so have I.

I might write another post like this in 15 years, so if you liked this, please check back in 2039!

