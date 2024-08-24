---
title: "Why are there no decent code formatters for Java?"
tags:
- java
- formatter
excerpt: In which I compare and rate code formatting tools for Java.
---
I haven't found a single Java code formatter that I like. And believe me, I've _looked_.

![Meme - all java code formatters suck, change my mind.jpg](/images/2024-08-24-why-are-there-no-decent-code-formatters-for-java/meme.jpg)

Let's discuss, because I really want my mind to change.

## My history with code formatters

First, some background. Feel free to skip this part if you want to get to the good part.

In my teens, I was working on a hobby project with a friend. We didn't know about version control yet, so every day at school we would exchange diskettes with source code which I would then integrate by hand when I got home. Part of this integration was fixing spacing, indentation and capitalization. My friend was a competent programmer but his code often looked sloppy to my perfectionist eyes. Making the corrections gave me a false sense of productivity, and it was probably at this time that a toxic thought settled in my mind: that a competent programmer is precise enough and cares enough about their craft that they don't need an automated formatter.

Years later, I had to do a code review with an inexperienced coworker. I was unable to read his code because it was so badly formatted. I'm not exaggerating, it was truly unreadable. I thought my toxic thought: this is clearly someone who doesn't care about their craft. I sent him back to fix the formatting, and then we resumed the review.

More years later, the toxic thought settled even deeper when I started working for a well-known Dutch company in a team where a code formatter was mandated. The formatter of choice was "badly-configured Eclipse", and instead of being checked into version control, the configuration was manually copied to every workstation. I asked a coworker what he thought of this formatter. Him: "I like how it's consistently readable." Me: "You mean consistently unreadable." Him: "Well...yeah." Sometimes I would try to make a bit of code more readable by making a manual adjustment, but whenever someone made an edit to the same file, Eclipse would happily reformat everything again. So I made an attempt to change the configuration. Everybody thought it was a good idea, but nobody was willing to reconfigure their Eclipse, so that's where that ended.

This is how I remember what it looked like (exaggerated for dramatic effect):

```java
ExpectedException
    .when(() -> EqualsVerifier.forClass(Foo.class).suppress(Warning.NONFINAL_FIELDS)
    .withPrefabValues(List.class, Arrays.asList(1, 2, 3).stream().map(i -> i + 1).toList(),
    Arrays.asList(1, 2, 3).stream().map(i -> i + 2).toList()).verify()).assertFailure()
    .assertMessageContains("something");
```

The change, for me, came a few years later still, when I was working on a Scala project and was introduced to [Scalafmt](https://scalameta.org/scalafmt/). A colleague insisted we use it and I grudgingly agreed. It was the first time I encountered a formatter that actually worked, and worked _well_. The configuration could be checked into git along with the project itself, and it integrated well with all of the tools. What's more, I found it actually caught inconsistencies that I had introduced myself. I guess I didn't care enough about my craft, after all! I noticed it improved my colleagues's code as well. And it would have solved that code review problem with the mere press of a button.

That was the end of the toxic thoughts about formatting: my mind was changed.

I decided to adopt a formatter in EqualsVerifier as well. I tried a few, started using one ([google-java-format](https://github.com/google/google-java-format)), switched to another ([Prettier Java](https://github.com/jhipster/prettier-java)), and have come to the bold conclusion that all code formatting tools for Java are inadequate.

## So what do I want from a formatter?

Basically, I want what formatters from other language ecosystems have, such as Scala, Rust and Go. Which is:

- 🏗: Integration with Maven. Anything that's not enforced by CI is merely a suggestion, and may as well not exist. I want CI to check formatting and fail the build if it's not good, and I want Maven to be able to reformat code too, so contributors don't have to install anything. As a corollary, if a binary must be installed, Maven must handle that too.
- 🏃‍♂️: It must be fast (or: there must be a fast way to format a file). That way, I can invoke it from [Neovim](https://jqno.nl/post/2020/09/09/my-vim-setup). A command-line tool is ideal, but Maven is too slow. Even [mvnd](https://github.com/apache/maven-mvnd) is not fast enough for this. In lieu of a command-line tool, I'll accept some kind of [Language Server Protocol](https://langserver.org/) integration. I realise that this may not be a concern for many people, but it is for me and it seems to me like a reasonable thing to want: it's standard practice for all other language ecosystems.
- ✨: It must format well. It may seem obvious, but I want the formatted code to look _nice_. See the code sample in the previous paragraph for an example of a formatter not doing a good job.
- 🚀: It must get out of my way. I don't want to get bogged down with version incompatibilities that I have to debug or jump through weird hoops in order to configure the thing.

If a formatter fails on one of these criteria, it fails as a whole. Some criteria may be subjective; in those cases I will be the judge and executioner. Again, all of the formatters for all of the languages have all of these features, so it shouldn't be strange to want the same things from a Java formatter, right? It's not like Java is a niche language with historically bad tooling, after all.

There are other factors to consider that, for me at least, don't weigh on the final verdict, but are interesting nonetheless:

- `IJ`: A plugin for IntelliJ would be nice. I don't use IntelliJ myself, but many potential contributors do, and I want things to be easy for them.
- ⚙️: Some formatters have many configuration options while others take inspiration from Go and provide none. I'm fine either way (as long as the results are good enough), but it's interesting to keep track of.

## Review of formatters

So, let's discuss all the formatters (that I know of) one by one. We'll review each criterion with one of the following emoji: 

- 🤩: Does exceptionally well
- 👍🏻: Is acceptable
- 👎🏻: Fails
- 0️⃣: Has no configuration options
- ✅: Has some configuration options
- 💯: Has many configuration options

Let's dive in.

### IntelliJ's built-in formatter

- 🏗: 👎🏻
- 🏃‍♂️: 👎🏻
- ✨: 👍🏻
- 🚀: 👍🏻
- `IJ`: 🤩
- ⚙️: 💯

It's a decent formatter, but it suffers from Kotlin-syndrome: there's no way you can use it from anything that isn't IntelliJ. You can't call it from Maven, and you _certainly_ can't use it from the command-line.

Also, if left unconfigured, it won't touch newlines. That can be a good thing as you have a lot of control over how your code is formatted, but has some edge cases. The results are not 100% predictable, and these are all possible results:

```java
EqualsVerifier.simple().forClass(Foo.class).verify();
```

```java
EqualsVerifier
        .simple()
        .forClass(Foo.class)
        .verify();
```

```java
EqualsVerifier.simple()
        .forClass(Foo.class).verify();
```

For me, that's not a deal-breaker. The lack of proper non-telliJ tooling, however, is.

Final verdict: 👎🏻

Example, with inconsistent line breaking:
```java
ExpectedException.when(
                () ->
                        EqualsVerifier.forClass(Foo.class).suppress(Warning.NONFINAL_FIELDS)
                                .withPrefabValues(
                                        List.class,
                                        Arrays.asList(1, 2, 3).stream().map(i -> i + 1)
                                                .toList(),
                                        Arrays.asList(1, 2, 3).stream()
                                                .map(i -> i + 2).toList())
                                .verify())
        .assertFailure()
        .assertMessageContains("something");
```

### [google-java-format](https://github.com/google/google-java-format)

- 🏗: 👍🏻
- 🏃‍♂️: 🤩
- ✨: 👎🏻
- 🚀: 🤩
- `IJ`: 👍🏻
- ⚙️: 0️⃣

This should have been the obvious choice. Integration with tooling is excellent, and the quality of the formatting would have been good if not for one _extremely weird_ design decision.

I guess Google decided that the tabs-vs-spaces discussion wasn't heated enough, and wanted to throw the _number_ of spaces into the mix. Java has had the convention of 4-space indents [since at least 1999](https://www.oracle.com/java/technologies/cc-java-programming-language.html), if not longer, so of course, Google decides they want to do 2 spaces instead.

Sure, other languages use 2 spaces. Who cares. If you want to participate in an ecosystem, you need to follow the established conventions for that ecosystem.

Thankfully, the _one_ configuration option that they have, allows you to use 4 spaces instead. This option is called AOSP for Android Open Source Project because when Google acquired it, Android had been using 4 spaces like any sane Java shop. Google doesn't document this option on their website, though.

Anyway, the AOSP option really accentuates google-java-format's only other weakness: it _loves_ to double-indent, and sometimes even quadruple-indent, especially with nested lambdas and expressions. This really pushes a lot of code to the right hand side of your editor, making the code look quite bad, as you can see in the examples below.

Final verdict: 👎🏻

Example, with default configuration - note that there actually aren't any 2-space indentations in this example; they only happen after `{`:
```java
ExpectedException.when(
        () ->
            EqualsVerifier.forClass(Foo.class)
                .suppress(Warning.NONFINAL_FIELDS)
                .withPrefabValues(
                    List.class,
                    Arrays.asList(1, 2, 3).stream().map(i -> i + 1).toList(),
                    Arrays.asList(1, 2, 3).stream().map(i -> i + 2).toList())
                .verify())
    .assertFailure()
    .assertMessageContains("something");
```

Example, with AOSP - note how everything gets pushed to the right:
```java
ExpectedException.when(
                () ->
                        EqualsVerifier.forClass(Foo.class)
                                .suppress(Warning.NONFINAL_FIELDS)
                                .withPrefabValues(
                                        List.class,
                                        Arrays.asList(1, 2, 3).stream()
                                                .map(i -> i + 1)
                                                .toList(),
                                        Arrays.asList(1, 2, 3).stream()
                                                .map(i -> i + 2)
                                                .toList())
                                .verify())
        .assertFailure()
        .assertMessageContains("something");
```

### [Prettier Java](https://www.jhipster.tech/prettier-java/)

- 🏗: 👍🏻
- 🏃‍♂️: 👍🏻
- ✨: 🤩
- 🚀: 👎🏻
- `IJ`: 👎🏻
- ⚙️: ✅

Prettier Java, on the surface, is also a very nice formatter. It has sane defaults and produces beautiful code out of the box. It's the one I'm currently using for EqualsVerifier.

It has a lot of peripheral problems though:

- It's an extension for [Prettier](https://prettier.io/), the JavaScript formatter. This means that it requires a full NodeJS runtime to work, which is ... unfortunate.
- It can be kind of annoying to install, because it needs to be installed as a plugin to Prettier. If you want to use [prettierd](https://github.com/fsouza/prettierd), to speed things up, it becomes downright _hard_ to set up.
- The formatting isn't stable between versions of Prettier Java, so you might have to re-format your code base every time you upgrade.
- Because of this, you might want to pin your Prettier Java to a specific version, which makes managing your prettierd installation _even harder_, because now you can't blindly update your system anymore. Of course, if you allow the Prettier Java in your build script and the one in your editor to diverge, they will start working against each other, which is no fun.
- There's no IntelliJ plugin.

Final verdict: 👎🏻

Example with Prettier Java 2.4.0:
```java
ExpectedException
    .when(() ->
        EqualsVerifier
            .forClass(Foo.class)
            .suppress(Warning.NONFINAL_FIELDS)
            .withPrefabValues(
                List.class,
                Arrays.asList(1, 2, 3).stream().map(i -> i + 1).toList(),
                Arrays.asList(1, 2, 3).stream().map(i -> i + 2).toList()
            )
            .verify()
    )
    .assertFailure()
    .assertMessageContains("something");
```

Example with Prettier Java 2.6.0 - note how the `() ->` gets its own line now, how `.when` and `.forClass` don't, and what the heck is going on with that single `)`!?
```java
ExpectedException.when(
    () ->
        EqualsVerifier.forClass(Foo.class)
            .suppress(Warning.NONFINAL_FIELDS)
            .withPrefabValues(
                List.class,
                Arrays.asList(1, 2, 3).stream().map(i -> i + 1).toList(),
                Arrays.asList(1, 2, 3).stream().map(i -> i + 2).toList()
            )
            .verify()
)
    .assertFailure()
    .assertMessageContains("something");
```

### Eclipse JDT formatter

- 🏗: 👍🏻
- 🏃‍♂️: 👍🏻
- ✨: 👍🏻
- 🚀: 👎🏻
- `IJ`: 👍🏻
- ⚙️: 💯

The formatter that turned me off on formatters so long ago. But if configured properly, it's actually very decent.

And that's immediately where the problem lies. Even though this formatter can be used as a more-or-less stand-alone tool, unlike IntelliJ's formatter, it's still very integrated with the Eclipse IDE, and you'll need to run it to configure it. And you _need_ to configure it. You can export your settings to an xml file of the kind that you're not going to want to edit by hand. In other words, you can't adopt this formatter without installing Eclipse, which is ... _sigh_

While researching this post, I found that IntelliJ is able to import and export Eclipse formatter configurations. However, I don't know how well this works.

Also, despite the fact that there are _two_ different Maven plugins to run this formatter, there's no proper command-line tool. The loop-hole here is that the LSP I use for Java is based on Eclipse and has the formatter built-in, so I can still run it from Vim. But if I ever want to try out Oracle's shiny new LSP, I'm out of a formatter again.

Since the Eclipse JDT formatter is basically a Java library that can be used from various tools, it shouldn't be too hard to make a GraalVM command-line tool to invoke it. But that adds quite a lot to the burden of adopting it, which was already high due to the need to configure it from Eclipse.

Final verdict: 👎🏻

Example with no configuration:
```java
ExpectedException
    .when(() -> EqualsVerifier.forClass(Foo.class).suppress(Warning.NONFINAL_FIELDS)
        .withPrefabValues(List.class, Arrays.asList(1, 2, 3).stream().map(i -> i + 1).toList(),
            Arrays.asList(1, 2, 3).stream().map(i -> i + 2).toList())
        .verify())
    .assertFailure().assertMessageContains("something");
```

Example with configuration:
- Wrapping settings → Function calls → Arguments → "Wrap all elements, except first element if not necessary"
- Wrapping settings → Function calls → Qualified invocations → "Wrap all elements, every element on a new line"
```java
ExpectedException
    .when(() -> EqualsVerifier
        .forClass(Foo.class)
        .suppress(Warning.NONFINAL_FIELDS)
        .withPrefabValues(List.class,
            Arrays.asList(1, 2, 3).stream().map(i -> i + 1).toList(),
            Arrays.asList(1, 2, 3).stream().map(i -> i + 2).toList())
        .verify())
    .assertFailure()
    .assertMessageContains("something");
```

### [Palantir Java Format](https://github.com/palantir/palantir-java-format)

- 🏗: 👍🏻
- 🏃‍♂️: 👎🏻
- ✨: 🤩
- 🚀: 👍🏻
- `IJ`: 👍🏻
- ⚙️: 0️⃣

Palantir Java Format is based on google-java-format, and somehow they manage to make Google's bad parts good, and Google's good parts bad, at the same time.

On the one hand, they make google-java-format's crappy formatting much, _much_ better. But at the same time, they don't offer a command-line tool. Yes, there's Maven integration, and yes, there's an IntelliJ plugin. But no official command-line tool. Weird.

There's a way to work around that. If you use Arch, BTW, you can download [a script from the AUR](https://aur.archlinux.org/cgit/aur.git/tree/palantir-java-format?h=palantir-java-format) that you can run from the command-line, and of course you can easily borrow and adapt that script if you use a more sensible OS. You'll have to gather and package the dependencies somehow, though. If you want to go down that path, here's the updated ratings:

- 🏗: 👍🏻
- 🏃‍♂️: 👍🏻
- ✨: 🤩
- 🚀: 👎🏻
- `IJ`: 👍🏻
- ⚙️: 0️⃣

I give 🚀 a 👎🏻 in this case because you have to figure out how to share the script and its dependencies with other people, and how you want to update them.

This script does expose that Palantir is probably a wrapper around google-java-format, rather than a fork of it. I'm not sure what that implies for the long-term stability of the project, and I don't know if I should let myself be bothered by that.

Also, there's the business activities of Palantir, the company behind this formatter. I don't know if I should let myself be bothered by that, either. But I might. I think I will.

Either way, the messy business around the command-line tool is a deal-breaker for me.

Final verdict: 👎🏻

Example:
```java
ExpectedException.when(() -> EqualsVerifier.forClass(Foo.class)
                .suppress(Warning.NONFINAL_FIELDS)
                .withPrefabValues(
                        List.class,
                        Arrays.asList(1, 2, 3).stream().map(i -> i + 1).toList(),
                        Arrays.asList(1, 2, 3).stream().map(i -> i + 2).toList())
                .verify())
        .assertFailure()
        .assertMessageContains("something");
```

### [Spring Java Format](https://github.com/spring-io/spring-javaformat)

- 🏗: 👍🏻
- 🏃‍♂️: 👎🏻
- ✨: 👍🏻
- 🚀: 👎🏻
- `IJ`: 👍🏻
- ⚙️: 0️⃣

This seems to be a wrapper around the Eclipse JDT formatter (and also Checkstyle) with a hard-coded configuration. The formatting looks nice enough, and it provides plugins for all the build tools and IDEs ... but no command-line tool.

For some reason, it needs a modification in your `.m2/settings.xml` file, which is weird.

Final verdict: 👎🏻

Example:
```java
ExpectedException
    .when(() ->
        EqualsVerifier
            .forClass(Foo.class)
            .suppress(Warning.NONFINAL_FIELDS)
            .withPrefabValues(
                List.class,
                Arrays.asList(1, 2, 3).stream().map(i -> i + 1).toList(),
                Arrays.asList(1, 2, 3).stream().map(i -> i + 2).toList()
            )
            .verify()
    )
    .assertFailure()
    .assertMessageContains("something");
```

### Cross-language code formatters

- 🏗: 👎🏻
- 🏃‍♂️: 🤩
- ✨: ???
- 🚀: 👎🏻
- `IJ`: 👍🏻
- ⚙️: 💯

There are many languages that look like Java (because they all descend from C), so it makes sense that there exist various tools that can format all of these languages: [ClangFormat](https://clang.llvm.org/docs/ClangFormat.html), [Artistic Style (astyle)](https://astyle.sourceforge.net/), [Uncrustify](https://github.com/uncrustify/uncrustify)...

They are all similar in a way, in that they all have excellent command-line support. Some have IntelliJ plugins, some don't, but all of them are hard to use from Maven: you'll have to pre-install them and then use `exec-maven-plugin` to run them.

Also, since there are so many configuration options, and the tools are focused on other languages than Java, they don't look particularly well out of the box, so expect to invest a lot of time tweaking. For this reason, I won't include examples, since they don't mean much without configuration anyway.

Final verdict: 👎🏻

### [EditorConfig](https://editorconfig.org/)

For completeness's sake, I'll give a mention to EditorConfig. It's a nice editor-agnostic and language-agnostic tool that ensures consistent use of tabs/spaces, EOL characters, and indentation. But nothing more.

It's a very nice and useful tool that's good at what it does, but since it's not a full formatter, I won't rate it here.

### [Spotless](https://github.com/diffplug/spotless)

I'll also give a shout-out to Spotless, since many people mention it on Reddit whenever somebody asks what Java formatter they use.

Spotless isn't a formatter. What it is, is a plugin for both Maven and Gradle that you can use to run most of the formatters mentioned above, and it's very good at that!

### Summary

If I compile the final verdicts for all the formatters I discussed, there's a clear trend:

- IntelliJ's built-in formatter: 👎🏻
- google-java-format: 👎🏻
- Prettier Java: 👎🏻
- Eclipse JDT formatter: 👎🏻
- Palantir Java Format: 👎🏻
- Spring Java Format: 👎🏻
- Cross-language code formatters: 👎🏻

None of the formatters that I could find provide a decent solution to all of the stated requirements.

This makes the Java ecosystem very different from the ecosystems of most other (modern) languages, where there's a single canonical tool that satisfies all the requirements.

Why can't Java have that?

## Wrapping up

I'm currently using Prettier Java for EqualsVerifier, but I'm very open to switching to another tool. However, I have a hard time picking one, because they all have problems that make me not want to use them. I've already switched formatters before (once from nothing to google-java-format, and once from google-java-format to Prettier Java), so if I'm going to switch again, I want to switch to something _really good_, and that simply doesn't exist right now.

On the other hand, inertia means that I have to stay with Prettier Java, which causes annoying problems every time it updates.

And then there's the Git history pollution that occurs when you switch formatters (or update Prettier Java). Fortunately, Git has a clever way to deal with that: you can add a file with Git commit hashes to your repo, and then Git is able to ignore those commits when doing a Git blame. Read [this](https://moxio.com/blog/ignoring-bulk-change-commits-with-git-blame/) for more info on that.

## Conclusion

If you don't care about running the formatter from the command-line, and you are ethically OK with it, I think Palantir is probably your best option. Otherwise, there is no formatter I can currently recommend. Maybe Eclipse is the lesser of all evils.

Is there a formatter I missed? Or some configuration or feature that could change my assessment for a formatter? Please let me know!
