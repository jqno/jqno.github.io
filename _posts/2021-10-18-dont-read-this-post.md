---
title: "Don't read this post! ☠️💣💥"
tags:
- java
- hack
- unicode
- reflection
- dependencies you already probably have on your classpath
excerpt: In which I provide ammo to annoy your coworkers.
---
_This is a partial transcription of [one of my talks](https://jqno.nl/talks/dont-hack-the-platform)._

Are you reading this post anyway? Wasn't the title clear enough? ಠ\_ಠ

Oh well. It would be unfortunate if JAX filled this slot with a blog post on blockchain, so let's do this anyway. Here's a few ways to upset your coworkers. Especially Martin: he's just _asking_ for it.

(If your name is Martin, I can imagine the surprise on your face. But realistically, almost everybody has a coworker named Martin. It's a well-known fact.)

Note that everything in this post has been tested on Java 17.

## True lambda

Let's start with something small. Everybody knows that we can use lambda's since Java 8. But a lambda isn't _truly_ a lambda if you don't call it λ:

{% highlight java %}
var ints = List.of(1, 2, 3);
Consumer<Integer> λ = i -> System.out.println(i);
ints.forEach(λ);
{% endhighlight %}

It turns out that many unicode symbols are allowed in Java identifiers. Z̗̹͎͚̊̈͋̄͆͆̒͠Á̳̞̻͚̌̃ͪ̈́͋̾͝L͙͔̼̗̭͖̇̔͊͛ͪ͠G̲̦̭̘͕̍ͫͨ̌̀ͤͬ̐͟Ô͕͓͍̓͢ is a valid name for a variable. It's kind of [hard to type](https://www.zalgotextgenerator.com), but Java is just fine with it.

## Unicode escape sequences

Most Java developers will know that you can use the `\uXXXX` notation in Strings to express Unicode symbols. However, after having passed their Certified Programmer exam, most will have repressed that this doesn't just work in strings, but in all Java code. This test will pass:

{% highlight java %}
@Test
public void falseIsTrue() {
    // Flaky test! \u000A if (true) return;
    assertEquals("false", "true");
}
{% endhighlight %}

`\u000A` is the Unicode representation of a new-line symbol, and the Java compiler will treat it as such. This is a handy way to fix a flaky test!

But you can take it further: you can hide entire programs inside Unicode escape sequences! Both the compiler and Maven are fine with that. For example:

```
\u0070\u0075\u0062\u006c\u0069\u0063\u0020\u0063\u006c\u0061\u0073\u0073\u0020\u0058\u007b
\u0070\u0075\u0062\u006c\u0069\u0063\u0020\u0073\u0074\u0061\u0074\u0069\u0063
\u0020\u0076\u006f\u0069\u0064\u0020\u006d\u0061\u0069\u006e\u0028
\u0053\u0074\u0072\u0069\u006e\u0067\u005b\u005d\u0061\u0029\u0074\u0068\u0072\u006f\u0077\u0073
\u0020\u0045\u0078\u0063\u0065\u0070\u0074\u0069\u006f\u006e\u007b
\u006a\u0061\u0076\u0061\u002e\u0061\u0077\u0074\u002e\u0044\u0065\u0073\u006b\u0074\u006f\u0070
\u002e\u0067\u0065\u0074\u0044\u0065\u0073\u006b\u0074\u006f\u0070\u0028\u0029
\u002e\u0062\u0072\u006f\u0077\u0073\u0065\u0028\u006e\u0065\u0077\u0020
\u006a\u0061\u0076\u0061\u002e\u006e\u0065\u0074\u002e\u0055\u0052\u0049\u0028
\u0022\u0068\u0074\u0074\u0070\u003a\u002f\u002f\u0079\u006f\u0075\u0074\u0075\u002e\u0062\u0065\u002f\u0022\u002b
\u0022\u0064\u0051\u0077\u0034\u0077\u0039\u0057\u0067\u0058\u0063\u0051\u0022\u0029\u0029\u003b\u007d\u007d
```

Note: the position of the line endings matters. Save this snippet as `X.java` and run it with `javac X.java && java X`. (Whoever tweets me `@jqno` with hash tag `#irefusedtolisten` and a screenshot that contains this snippet _and_ the result, will get a free retweet!)

## Reflection

Reflection enables some nice tricks too. A short disclaimer before I show these: all code in this article was tested on Java 16 and should be future-proof. Of course they also work in older versions of Java.

And by the way, if you get `illegal reflective access` warnings, you can suppress them by adding `--add-opens java.base/java.lang=ALL-UNNAMED` to the JVM's startup parameters. It would be a shame if Martin catches you because of such an obtrusive message in your application's logs.

## Loopy

Here's an example of a nice reflection trick:

{% highlight java %}
public class Loopy {
    public static void main(String... args) throws Exception {
        var field = Integer.class.getDeclaredField("value");
        field.setAccessible(true);
        field.set(5, 4);

        for (Integer i = 0; i < 10; i++) {
            System.out.println(i);
        }
    }
}
{% endhighlight %}

This program prints `0`, `1`, `2`, `3`, and then an infinite amount of `4`s. This happens because most JVMs cache the first 128 boxed Integers. The code takes `5` from that cache and changes that Integer's `value` field to `4`. So when the loop reaches `4` and adds `1` to that, the JVM takes the object for `5` from the cache. But that now has a value of `4`! So `i` becomes `4`. Next, we add `1` to that and end up with `5`. The JVM takes `5` from the cache, sees a `4`, and prints that. Then we add `1`, and the cycle continues ad nauseam.

If you have an annoying coworker who insists on always using boxed primitives instead of regular ones (yes, these people _actually_ exist), it's very easy to mess up his unit tests by polluting the JVM's integer cache somewhere in an unrelated unit test. 💣!

## Constructors are hard

If you happen to have [Objenesis](http://objenesis.org) on your class path (or module path!), you can take it a step further. Objenesis is a library that can instantiate objects without calling the constructor. This can be useful when a constructor is private, when it has a lot of tricky parameters, or when it just neatly checks its preconditions. Bah. This is why Objenesis is used in many mocking frameworks. Chances are that you already have it!

So, did you ever want to instantiate `Void`? Now you can!

{% highlight java %}
var objenesis = new ObjenesisStd();
Void avoid = objenesis.newInstance(Void.class);
System.out.println(avoid);
{% endhighlight %}

## Enums

Objenesis is so good at instantiating objects, that it even works for enums!

Enums? But didn't Joshua Bloch say in Effective Java (2nd _and_ 3rd edition&mdash;and yes, I bought the 3rd edition especially for this blog post to see if the quote was still there) that the JVM gives an "ironclad guarantee" against "sophisticated [...] reflection attacks" to sneakily instantiate enums, and that this is why they're so effective for defining singletons?

Yes. Yes, he did. And what's more, even [the Java Language Specification says it](https://docs.oracle.com/javase/specs/jls/se10/html/jls-8.html#jls-8.9). And still it works. Check it out:

{% highlight java %}
public static <T> void setPrivateFieldValue(Class<T> type, String fieldName, T receiver, Object newValue)
        throws NoSuchFieldException, IllegalAccessException {
    var field = type.getDeclaredField(fieldName);
    field.setAccessible(true);
    field.set(receiver, newValue);
}

enum Singleton { INSTANCE }

Singleton copy = objenesis.newInstance(Singleton.class);
setPrivateFieldValue(Enum.class, "ordinal", copy, 0);
setPrivateFieldValue(Enum.class, "name", copy, "INSTANCE");
{% endhighlight %}

Child's play. Say bye-bye to your singleton!

If you're still on Java 11, you can take it even further. It's easy to add a new element to an existing enum by fiddling with an enum instance's `ordinal` and `name` fields and updating the private `$VALUES` field inside the enum's class. You'll have to clear its `final` modifier, too. Now we can add a new suit to our deck of cards:

{% highlight java %}
enum Suits { DIAMONDS, CLUBS, HEARTS, SPADES };

Suits trumps = objenesis.newInstance(Suits.class);
setPrivateFieldValue(Enum.class, "ordinal", trumps, 4);
setPrivateFieldValue(Enum.class, "name", trumps, "TRUMPS");

Field valuesField = Suits.class.getDeclaredField("$VALUES");
valuesField.setAccessible(true);

setPrivateFieldValue(Field.class, "modifiers", valuesField, valuesField.getModifiers() & ~Modifier.FINAL);
Suits[] allSuits = { DIAMONDS, CLUBS, HEARTS, SPADES, TRUMPS };
valuesField.set(null, allSuits);

for (Suits kleur : Suits.values()) {
    System.out.println(kleur);
}
{% endhighlight %}

Unfortunately, Java 12 made this trick impossible by disallowing reflection into Java's own reflection classes.

## Agents

Java Agents allow you to confuse Martin in a very different way. They are regular Java programs that can be attached to a running JVM, which allows them to inject code there. Agents are a standard feature in Java and don't require anything special. Once an agent is attached to a Java process, the `agentmain` method is called. This method is similar to the well-known `public static void main` method.

Whoah, you might think. That's pretty nasty! Why is this a standard feature in Java?

Well, there are many Java Agents with legitimate use cases. Profilers, debuggers, monitoring tools: just three applications that would be impossible without agents.

But we're not interested in legitimate use cases right now. We just want to make Martin's life hard. Let's do that by fouling up `System.currentTimeMillis()` in a running process.

Let's say you're working on a high-speed trading system, or a planner for timetables for trains. In those cases, it's very important to know exactly what time it is. Let's simplify that kind of application to a program that simply prints the time every second:

{% highlight java %}
public class Victim {
    public static void main(String... args) throws Exception {
        System.out.println("Current PID: " + ProcessHandle.current().pid());
        while (true) {
            // It's super important that this time ↓ is correct!
            System.out.println(System.currentTimeMillis());
            Thread.sleep(1000);
        }
    }
}
{% endhighlight %}

An agent can help us make sure that this program prints the number `1337` every second. We'll use an open source library called [Byte Buddy](http://bytebuddy.net), which makes it easy to generate bytecode. As with Objenesis, chances are pretty good that Byte Buddy already on your class path, because it's a transitive dependency of often-used tools like Hibernate and Jackson.

Our agent will print a short line of text and then gets to work. The transformer (which is a lambda and hence is called λ) searches for a method called `currentTimeMillis`, intercepts it, and makes it return `1337`:

{% highlight java %}
public class Attack {

    public static void agentmain(String arguments, Instrumentation instrumentation) {
        System.out.println("Let's install some malicious code...  ᕕ( ᐛ )ᕗ ");

        AgentBuilder.Transformer λ =
                (builder, typeDescription, classLoader, javaModule) -> builder
                                .method(named("currentTimeMillis"))
                                .intercept(value(1337));

        new AgentBuilder.Default()
                .ignore(none())
                .disableClassFormatChanges()
                .with(AgentBuilder.RedefinitionStrategy.RETRANSFORMATION)
                .type(named("java.lang.System"))
                .transform(λ)
                .installOn(instrumentation);
    }
}
{% endhighlight %}

The `AgentBuilder` takes the transformer and links it to `java.lang.System`. Then it installs the agent on `instrumentation`, a variable that refers to the running process that we want to wreck.

Compile this class and stick it in a jar file, for instance `attack.jar`. Make sure that its `MANIFEST.MF` file contains at least these lines:

{% highlight http %}
Agent-Class: the.package.name.that.youre.using.Attack
Can-Retransform-Classes: true
{% endhighlight %}

The final piece of the puzzle is to actually link the agent to the process. Make sure you know the victim process's PID. The `Victim` program conveniently prints it out for us, but on a POSIX system you can also find it using `ps` on the command-line. With Byte Buddy, it's now a one-liner that you can even run from JShell:

{% highlight java %}
ByteBuddyAgent.attach("/path/to/attack.jar", pid);
{% endhighlight %}

Victim's output then looks more or less like this:

{% highlight plaintext %}
Current PID: 63157
1528464432723
1528464433725
1528464434726
Let's install some malicious code...  ᕕ( ᐛ )ᕗ 
1528464435728
1337
1337
1337
1337
1337
1337
{% endhighlight %}

The fun thing is, if you have access to your production server, you can do this trick there too. Martin won't know what hit him! 👍

By the way, you can also use agents to repair the enum hackery in the previous section for more recent versions of Java. If you want to know how, come to my talk at JAX London!

## Awareness

You can also go for the double bluff: fight coworkers who started reading this article, but got distracted halfway through so they could try one of the pretty hacks in this article and never returned to read the rest of it. You can annoy them by making sure that hack won't work.

Most Unicode tricks are simply neutralized by a few strategically selected SonarQube rules.

The reflection hacks are a bit trickier, but can be stopped as well: use a security manager. Add this line to your program somewhere:

{% highlight java %}
System.setSecurityManager(new SecurityManager());
{% endhighlight %}

You don't even have to import anything! The JVM will now throw a `SecurityException` on every reflection hack. And also when you use Spring, because Spring is really not much more than a large mush of dirty, _dirty_ reflection hacks. 👹

Sadly, the security manager was deprecated for removal in Java 17. So what else can you do?

Well, you can modularize your code base and use the `--illegal-access=deny` JVM parameter. This will make sure you can only perform reflection on classes within the same module. Because all Java API classes reside in a different module (`java.base`), you can't get to them anymore.

And really, what's a better way to upset Martin than forcing him to completely modularize all the code?
