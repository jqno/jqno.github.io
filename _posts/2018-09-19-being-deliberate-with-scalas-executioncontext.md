---
title: "Being deliberate with Scala's ExecutionContext"
excerpt: In which I present a way to make sure Scala's implicit global ExecutionContext isn't used accidentally
tags:
- scala
- archunit
- executioncontext
---
## The problem

In Scala we often use `Future` to deal with concurrency, especially when working with the Play! Framework. To operate, `Future` needs an implicit `ExecutionContext`, which manages the concurrency. Fortunately, if you forget one, the compiler is very helpful with its error message:

    [error] /path/to/some/File.scala:7:16: Cannot find an implicit ExecutionContext. You might pass
    [error] an (implicit ec: ExecutionContext) parameter to your method
    [error] or import scala.concurrent.ExecutionContext.Implicits.global.
    [error]   println(fut.map(_ + 1))
    [error]                  ^

This might prompt a developer to, I don't know, import `scala.concurrent.ExecutionContext.Implicits.global`. If he does, the error goes away and all seems well.

But all is not well. The global `ExecutionContext` that Scala provides, which is [configurable and backed by a work-stealing thread pool](https://www.scala-lang.org/api/current/scala/concurrent/ExecutionContext$.html#global:scala.concurrent.ExecutionContextExecutor), will be adequate in many cases, but not all. In fact, the Play! Framework defines its own implicit `ExecutionContext` called `play.api.libs.concurrent.Execution.defaultContext`, also accessible implicitly through `play.api.libs.concurrent.Execution.Implicits.defaultContext`, which is [a very different thing](https://stackoverflow.com/questions/30805337/plays-execution-contexts-vs-scala-global). In Play! applications, it makes sense to use it instead of Scala's global one. That means you need to be careful to import the right one; if you decide to use Play!'s `ExecutionContext`, you don't want to import Scala's by mistake. This is a bad thing, because then you have two different `ExecutionContext`s competing for the same resources. The compiler's helpfully intended suggestion to import the global `ExecutionContext` is quite dangerous here!

## Scalastyle

Fortunately, it's quite easy to add a rule to [Scalastyle](http://www.scalastyle.org/rules-1.0.0.html#org_scalastyle_scalariform_IllegalImportsChecker) that forbids the import of `scala.concurrent.ExecutionContext.Implicits.global`. This solves much of the problem, but not all of it, because of course there's other ways you can still (accidentally or not) use the global `ExecutionContext`. For instance, you could declare an `implicit val ec = scala.concurrent.ExecutionContext.Implicits.global` in your class. Also, there's still the non-implicit alias `scala.concurrent.ExecutionContext.global` that you could use. But doing that assumes a level of intent that goes beyond blindly following the compiler's suggestion, and it should be dealt with easily enough in a code review. So if we use Scalastyle and do proper code reviews, we're done, right?

Well, no.

In my previous project, our team wasn't aware of the difference between Scala's `ExecutionContext` and Play!'s `ExecutionContext`. This led to some developers importing one, and others the other `ExecutionContext`. Code reviews don't matter if reviewers don't know what to look for.

## Dependency injection

When we finally did figure it out, it was an easy thing to globally search and replace one `ExecutionContext` import with another, but upon reflection, importing an implicit global `ExecutionContext` seems like a strange thing to begin with. An `ExecutionContext` is a _dependency_ of a class, and should be _injected_ as such. That way it becomes much easier to change the details of your application's `ExecutionContext` later, for instance when performance issues come to light after running in production for a month and you need to make a tweak.

However, whatever your chosen dependency injection method is (using a framework like Guice or MacWire, or doing it manually), it's an awful lot of work to refactor a large codebase from global imports to constructor parameters. Therefore, in my current project we decided to be very deliberate about `ExecutionContext` from the beginning so we wouldn't have to do a big rewrite later on.

We used Scalastyle to forbid the import of _both_ Play!'s `ExecutionContext` _and_ the global one. Of course we made an exception for the place where the actual wiring takes place. But we felt that didn't go far enough, as it's still very easy to get hold of the global `ExecutionContext` even with the import restrictions in place, either through `scala.concurrent.ExecutionContext.Implicits.global` or `scala.concurrent.ExecutionContext.global` (which refer to the same `ExecutionContext` under the hood). We wanted something stronger.

## ArchUnit

In the end, we found [ArchUnit](https://www.archunit.org), a tool that lets you unit-test your architecture using a fluent and very readable interface. We settled on the following ScalaTest code:

{% highlight scala %}
it should "never use the global ExecutionContext outside WiredApplication" in {
  val importedClasses = new ClassFileImporter()
      .withImportOption(ImportOption.Predefined.DONT_INCLUDE_TESTS)
      .withImportOption(ImportOption.Predefined.DONT_INCLUDE_JARS)
      .importPackages("com.example.app")

  val rule =
    noClasses.that.dontHaveFullyQualifiedName(classOf[WiredApplication].getCanonicalName)
      .should.callMethod(scala.concurrent.ExecutionContext.Implicits.getClass, "global")
      .orShould.callMethod(scala.concurrent.ExecutionContext.getClass, "global")
      .orShould.callMethod(play.api.libs.concurrent.Execution.Implicits.getClass, "defaultContext")
      .orShould.callMethod(play.api.libs.concurrent.Execution.getClass, "defaultContext")
      .as(s"No-one except WiredApplication should use the global ExecutionContext, implicitly or explicitly")
      .because("we want to be in full control, through dependency injection, of which ExecutionContext is in use")

  rule.check(classes)
}
{% endhighlight %}

This test test checks every class file in the application for references to the method `global` in either `ExecutionContext`'s companion object or `ExecutionContext.Implicits`.

ArchUnit is designed with Java in mind, not Scala, so we had to make a few adjustments. For instance, in `ExecutionContext.Implicits`, `global` is defined as a `lazy val`. Under the hood, that gets compiled into a method, so we have to use ArchUnit's `callMethod`.

A very nice thing about ArchUnit is the fact that you can override its default error messages (which are already pretty good) with your own messages using the `as` method. Another nice thing: with `because`, you can specify a reason for the ArchUnit rule, which is also included in the error message.

Of course there are also disadvantages to this approach. First, ArchUnit will slow down your test suite. It can cache things to make subsequent rules much faster, but if you only have one rule (like we do at the time of writing), that doesn't help. Also, it's a heavy tool to use for (arguably?) little benefit, as Scalastyle can get you to 90% of where you need to be.

But we originally chose Scala partly because its tools and culture are very focused on letting the compiler and the tools do as much of the work as possible. We use its type system to prevent as many bugs as possible, and complement that with tests to check for things that the compiler can't. Confusion around `ExecutionContext` has bitten me hard before, so for us it was worth it to invest in a tool that helps us be more deliberate. In that sense, ArchUnit fits our team very well, and we'll be looking out for more ways to leverage it in —pardon the pun— future.

