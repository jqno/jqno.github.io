---
title: "AnnotationScript: abusing Java annotations to the extreme"
tags:
- java
- lisp
- annotation
- annotationscript
excerpt: In which I take things way too far by implementing LISP using only Java annotations, then using that language to implement another LISP.
---
## TL;DR

I have created a programming language whose syntax is expressed entirely in Java annotations. It's available on [GitHub](https://github.com/jqno/AnnotationScript).

For the why and the how, read on! 

## Table of contents

This story is going to be a rollercoaster, so I've decided to split it up so you can catch your breath between loopings.

* [The introduction of AnnotationScript!](#introduction)
* [The nerd cred of LISP](#lisp)
* [The excessive power of annotations](#annotationpower)
* [The limiting restrictions of anotations](#annotationrestrictions)
* [The inner workings of AnnotationScript](#innerworkings)
* [The limited power of AnnotationScript](#annotationscriptpower)
* [The proof of the pudding](#proof)
* [The ridiculous properties of MetaScript](#metascriptpower)
* [The proof the proof of the pudding](#proofproof)
* [The work thankfully left undone](#futurework)
* [Conclusion](#conclusion)

<a id='introduction'/>
## The introduction of AnnotationScript!

In 2008, a story went viral that the creators of successful programming languages all have beards ([here's a summary about that from 2012](https://www.wired.com/2012/06/beard-gallery/)). So, ever since I decided to [grow one](https://www.youtube.com/watch?v=RXJFBqh6cVc) myself in 2011, naturally, people asked me if I was going to create one, too. My answer has always been "no" -- until now. Now, almost ten years later, I'm finally ready to announce **AnnotationScript**!

So without further ado, here are [Hello World](https://github.com/jqno/AnnotationScript/blob/8806dc7c3bea76d04ab5593e313abfc3085edd2f/src/main/java/nl/jqno/annotationscript/demo/HelloWorld.java) and [FizzBuzz](https://github.com/jqno/AnnotationScript/blob/8806dc7c3bea76d04ab5593e313abfc3085edd2f/src/main/java/nl/jqno/annotationscript/demo/FizzBuzz.java):

{% highlight java %}
package nl.jqno.annotationscript.demo;

import nl.jqno.annotationscript.AnnotationScript;
import nl.jqno.annotationscript.Annotations.Zero;

@Zero("println")
@Zero("'Hello world!'")
public class HelloWorld {
    public static void main(String[] args) {
        AnnotationScript.run(HelloWorld.class);
    }
}
{% endhighlight %}

{% highlight java %}
import nl.jqno.annotationscript.AnnotationScript;
import nl.jqno.annotationscript.Annotations.*;

@Zero("begin")
@Zero(list={@One("define"), @One("fizz-buzz"), @One(list={@Two("lambda"), @Two(list=@Three("n")), @Two(list={
    @Three("cond"),
    @Three(list={@Four("="), @Four(list={@Five("%"), @Five("n"), @Five("15")}), @Four("0")}), @Three("'fizzbuzz'"),
    @Three(list={@Four("="), @Four(list={@Five("%"), @Five("n"), @Five("3")}), @Four("0")}), @Three("'fizz'"),
    @Three(list={@Four("="), @Four(list={@Five("%"), @Five("n"), @Five("5")}), @Four("0")}), @Three("'buzz'"),
    @Three("else"), @Three("n")})})})
@Zero(list={@One("map"), @One("println"), @One(list={@Two("map"), @Two("fizz-buzz"), @Two(list={@Three("range"), @Three("1"), @Three("101")})})})
public class FizzBuzz {
    public static void main(String[] args) {
        AnnotationScript.run(FizzBuzz.class);
    }
}
{% endhighlight %}

For an explanation of how the language works, I refer to the [GitHub repo's README](https://github.com/jqno/AnnotationScript/blob/fac137d719b084b5fc953006886c58516222d791/README.md).

<a id='lisp'>
## The nerd cred of LISP

So, what's with all the weird Java annotations? I mean, the name AnnotationScript kind should be a hint, but still why!? I will get back to that. First, I want to talk about LISP. Please, bear with me.

I decided early on that I wanted to base AnnotationScript on [LISP](https://en.wikipedia.org/wiki/Lisp_%28programming_language%29). You know, that weird language that nobody uses; the one with all those parentheses. The main reason for that is that it's relatively easy to implement. In fact, implementing LISP can be seen as a rite of passage, because it's such a simple yet powerful language.

It's relatively straightforward to write an interesting program in LISP. While a [Turing Machine](https://en.wikipedia.org/wiki/Turing_machine) language like [Brainfuck](https://en.wikipedia.org/wiki/Brainfuck) or [Whitespace](https://en.wikipedia.org/wiki/Whitespace_%28programming_language%29) is certainly easier to implement, it's a lot harder to use them to write an interesting demo program.

LISP's syntax is extremely easy. Everything is a list enclosed by parentheses. The first element of a list is a function, and the rest of the elements are the parameters to pass to that function. BAM. Now you know all you need to know about LISP syntax.

For example, a program in LISP might look like this:

{% highlight lisp %}
(if (> x 0) 1 -1)
{% endhighlight %}

The equivalent code in Java would be:

{% highlight java %}
if (x > 0) {
  return 1;
} else {
  return -1;
}
{% endhighlight %}

Easy, right?

<a id='annotationpower'/>
## The excessive power of annotations

So let's get back to the Java annotations. Because WTF? Why would anyone in their right mind create an entire language with Java annotations?

Well, it's no secret that I dislike Java annotations. In fact, I've even given [a talk](https://jqno.nl/talks/paralleljava/) on that subject:

{% include video id="R0WnUd01f14" provider="youtube" %}

Summarizing that talk; I think annotations are over-used in the Java ecosystem. Originally intended to add metadata to your code, like `@Deprecated` and `@Override`, they are currently used to generate all kinds of code, which has made codebases hard to understand and debug because you can't just use your IDE to navigate to the code that handles the annotations, or put a breakpoint there: all you see is a small interface class with no code. Dependency injection, handling http requests, interacting with databases: it's all much easier using frameworks that don't rely on annotations. Yes, I'm looking at you, Spring.

But the pandemic lockdowns of 2020 did strange things to people, and I started wondering: if annations have become so powerful, why has nobody taken them to their logical extreme? Why has noone created a full-blown, [Turing-complete](https://en.wikipedia.org/wiki/Turing_completeness) _programming language_ using Annotations?

So I decided to do just that.

<a id='annotationrestrictions'/>
## The limiting restrictions of anotations

So let's talk about that strange `@Zero`, `@One`, `@Two` syntax.

When designing a programming language, you have to come up with a good syntax. Fortunately, LISP's syntax is extremely simple, but I still had to find a way to express that in Java annotations. Unfortunately, for all the power they unlock, annotations come with a bunch of restrictions that make that really hard.

At first, I wanted to create a syntax that was very close to LISP:

{% highlight java %}
@Parenthesis("if", @Parenthesis(">", "x", "0"), "1", "-1")
class Source {}
{% endhighlight %}

Here, I ran into the first restriction: you can't nest Java annotations. You just can't. Not allowed. Since LISP programs (or programs in any language, for that matter) would be rather dull if you can't nest code blocks, that was that for this approach.

My second attempt looked like this:

{% highlight java %}
@Open
@Symbol("if")
@Open@Symbol(">")@Symbol("x")@Symbol("0")@Close
@Symbol("1")
@Symbol("-1")
@Close
class Source {}
{% endhighlight %}

This is where I ran into the second restriction: if you want to use the same annotation multiple times on the same syntax element (e.g., a class), you have to make it `@Repeatable`. And if you make an annotation `@Repeatable`, you have to define a new annotation type to contain them (for instance, `@RepeatedSymbolContainer` with a `Symbol[]` property, and `@RepeatedOpenContainer` with a `Open[]` property). Then, the annotations will all get thrown into that container. At run-time, when you want to parse the annotations, you won't have access to the original annotations anymore: they will all be grouped by type into their respective holders. So, all `@Open`s will be in one container, all `@Symbols` in another, and all `@Close`s in a third. In other words, you lose all information needed to determine the structure of your program and build an [abstract syntax tree](https://en.wikipedia.org/wiki/Abstract_syntax_tree) for it.

If that's confusing (it was quite confusing to me), [here](https://docs.oracle.com/javase/tutorial/java/annotations/repeating.html) is a good explanation of repeating annotations.

So, no problem, I thought. I'll just define some interface, and have all of my annotations implement that. Then I can put all of them into the same container, and the structure is preserved. But no: annotations are not allowed to implement interfaces in Java. Bummer.

That's when I came up with the numbered annotations: each number represents a level of nesting, and an annotation can contain either a symbol or a list at a deeper (higher) level of nesting:

{% highlight java %}
@Zero("if")
@Zero(list={@One(">"), @One("x"), @One("0")})
@Zero("1")
@Zero("-1")
class Source {}
{% endhighlight %}

The only drawback is that I had to define an annotation for each level of nesting, which means that there would be a limit to how deeply a program could be nested. Unfortunate, but not a deal-breaker. I decided to start with `@Zero`, because as Dijkstra so eloquently explained, all [numbering should start at zero](https://www.cs.utexas.edu/users/EWD/transcriptions/EWD08xx/EWD831.html). I decided to end at `@Eleven`, because twelve levels of nesting felt like enough to me. You don't want to rack up that cyclomatic complexity, after all. Also, 11 is the 'crazy number' (het [gekkengetal](https://www.encyclo.nl/begrip/gekkengetal)) in Dutch tradition, which seemed appropriate.

<a id='innerworkings'/>
## The inner workings of AnnotationScript

I googled around a bit and found a blog post by [Peter Norvig](https://en.wikipedia.org/wiki/Peter_Norvig) where he explained, step by step, [how to implement LISP in Python](https://norvig.com/lispy.html). Great! I used that as a template. (In fact, it's a great stepping stone for implementing your own LISP and gain that nerd cred rite of passage for yourself as well. It's fun, I recommend it!)

There are two parts to the AnnotationScript interpreter: the parser and the evaluator.

The parser consists of two steps: tokenization, and constructing the abstract syntax tree. Tokenization means reading the input and dividing it into useful parts, such as `+`, `if` and `str/starts-with?`. In AnnotationScript, that's easy: every annotation contains a single token.

The abstract syntax tree (AST) is already defined by the annotation-level: `@Zero`, `@One`, etc. However, to stay closer to Norvig's implementation, I actually flatten the data structure during tokenization and re-parse it into a tree structure later. Kind of pointless, I know, but it felt more 'pure', and it actually came in handy later.

The evaluator also consists of two parts: the environment, and the evaluator itself. The environment is basically a huge HashMap that contains all the identifiers and the values assigned to them. As mentioned before, these values can be constants like `1` and `'Hello World'`, but they can also be functions, in the form of lambda-expressions. I have defined a large 'global' environment with useful constants and functions, and the user can add to it by using `define`.

Evaluating a program now becomes an exercise of traversing the AST and looking up identifiers in the environment.

In reality, it's a little bit more complicated than that, but for the details, I will refer to [Norvig's post](https://norvig.com/lispy.html), because this article is already too long.

However, it is interesting to note where I made a few different choices. For example, I used an immutable datastructure to contain the values of variables and other things, and I used recursion instead of looping. Also, I renamed some primitives (I find that `head` makes more sense than `car`, and `tail` makes more sense than `cdr`, for instance), and I added a bunch of primitives that Norvig's implementation didn't have, to make it a little easier for me to write some demo programs in AnnotationScript. Finally, I've added support for strings.

When it was time to write these demo programs, such as [FizzBuzz](https://github.com/jqno/AnnotationScript/blob/8806dc7c3bea76d04ab5593e313abfc3085edd2f/src/main/java/nl/jqno/annotationscript/demo/FizzBuzz.java) and [Ninety-Nine Bottles of Beer](https://github.com/jqno/AnnotationScript/blob/8806dc7c3bea76d04ab5593e313abfc3085edd2f/src/main/java/nl/jqno/annotationscript/demo/NinetyNineBottlesOfBeer.java), I also decided to write a generator that would translate LISP into AnnotationScript, because ain't nobody got no time to write all them stupid annotations by hand.

The generator is basically a tokenizer for LISP and a function that converts the list of tokens into the corresponding annotations. I kind of cheated on the tokenizer in the same way that Norvig did: by just calling `String::split` on spaces and assuming everything that comes out of that is a token. This means it's impossible to write a string containing spaces, because `'Hello world'` gets split into the tokens `'Hello` and `world'`. Oh well ¯\\\_(ツ)_/¯

<a id='annotationscriptpower'/>
## The limited power of AnnotationScript

Obviously, AnnotationScript is an awful programming language, but it does have some interesting properties. For instance, the fact that all values are Java objects, makes interop surprisingly easy.

Another interesting property is its composability, which comes in handy when writing unit tests. (And boy, do you need unit tests when writing AnnotationScript, because proper error handling is simply not a thing that I paid any attention to.) Say you have an expression like this:

{% highlight java %}
@Zero("if")
@Zero(list={@One("fire-number-of-lasers"), @One("2")})
@Zero("success")
@Zero("failure")
class Example {}
{% endhighlight %}

If you want to call this expression, a function `fire-number-of-lasers` must be in scope. But in a unit test you're free to include a stub function that doesn't actually fire those lasers.

Of course, this is something we're all already used to in Java, thanks to mocking frameworks like Mockito. However, they generally work on the class level. In AnnotationScript, you can swap on the function level, which is much more precise. I suppose this is true for any dynamic, non-object-oriented functional language, but I simply hadn't worked with one yet, so this felt very powerful to me.

<a id='proof'/>
## The proof of the pudding

At this point, I figured I had created something that could convincingly demonstrate the ridiculous power that Java annotations could have if taken too far. But I wanted to go one step further to really drive that point home. So I decided to implement a new LISP interpreter in AnnotationScript.

Let me repeat that.

I decided to implement a new LISP interpreter _in AnnotationScript_.

This time, I took the implementation from the classic computer science book [The Little Schemer](https://www.goodreads.com/book/show/548914.The_Little_Schemer). The final chapter explains how to create a LISP implementation in LISP.

Again, I needed to make some modifications, but only because I wanted to be able to implement FizzBuzz in it. 

The first modification was the hardest. The Little Schemer's implementation doesn't include looping or recursion. To work around that, I added a `define` primitive so I could bind things to names. That way, I could bind a lambda to an identifier and gain the ability to call functions. Unfortunately, there still is an issue with scoping. Because of that issue, the lambda is not bound to the identifier until after the definition is evaluated. In other words, inside of the lambda, the binding is not available. That means I have to pass it along as an extra parameter to the recursion. So, when you would normally write a recursive function like this:

{% highlight lisp %}
(define
  (recurse
    (lambda (n)
      (cond
        ((eq? n 0) 0)
        (else (add1 (recurse (sub1 n)))))))
  (recurse 10))
{% endhighlight %}

It would have to look like this instead:

{% highlight lisp %}
(define
  (recurse
    (lambda (n rec)
      (cond
        ((eq? n 0) 0)
        (else (add1 (rec (sub1 n) rec))))))
  (recurse 10 recurse))
{% endhighlight %}

You might ask, since I've referred to The Little Schemer already: why not use the [Y Combinator](https://en.wikipedia.org/wiki/Fixed-point_combinator#Fixed-point_combinators_in_lambda_calculus) instead? Well, simply put, I was unable to get it to work, which was mostly due to the fact that I don't understand it, even after having read and re-read the chapter. There are, apparently, limits to my folly.

The second modification was easy: I had to add a modulo operator, because I was too lazy to build one using only `add1`, `sub1`, and recursion, which were the only tools available to me otherwise.

<a id='metascriptpower'/>
## The ridiculous properties of MetaScript

Again, like AnnotationScript, in MetaScript all values are Java objects. This is something that I, again, leveraged in the unit tests (because did I mention how much we need unit tests to make sure things are working properly in this crappy language?)

For instance, like AnnotationScript, MetaScript needs a tokenizer to split a program into a list of tokens, and a parser to build an AST. It turns out that MetaScript's tokenizer needs to work exactly the same way as the one I wrote for the AnnotationScript generator, and it turns out that MetaScript's parser needs to work exactly the same way as AnnotationScript's parser. The only difference is that the input looks a little bit different.

Therefore, it was pretty easy to re-use some of AnnotationScript's unit tests. I could just move the tests into an abstract superclass [like this](https://github.com/jqno/AnnotationScript/blob/main/src/test/java/nl/jqno/annotationscript/abstract_test/StringTokenizerTest.java), add an abstract method that calls the correct tokenizer, and have two implementations: [one for AnnotationScript](https://github.com/jqno/AnnotationScript/blob/main/src/test/java/nl/jqno/annotationscript/language/LanguageStringTokenizerTest.java) and [one for MetaScript](https://github.com/jqno/AnnotationScript/blob/main/src/test/java/nl/jqno/annotationscript/meta/MetaStringTokenizerTest.java), and boom! I can now guarantee that they both work exactly the same way!

I'll be honest, I was a little giddy at how well this works.

<a id='proofproof'/>
## The proof of the proof of the pudding

Anyway, without further ado: [FizzBuzz in MetaScript](https://github.com/jqno/AnnotationScript/blob/main/src/main/java/nl/jqno/annotationscript/demo/MetaFizzBuzz.java)!

As you can see, I've defined the program in LISP and shoved it into a plain old Java string. I pass that to the `MetaScript::run` method which, [as you can see](https://github.com/jqno/AnnotationScript/blob/dc7962eea26dffb39d4eea799f56345b40d21151/src/main/java/nl/jqno/annotationscript/meta/MetaScript.java), calls `AnnotationScript.run` with the `Runner` class which is an AnnotationScript program defined in the same class, and passes the LISP string as a parameter to that. If you run it, you'll see that it actually works!

<a id='futurework'/>
## The work thankfully left undone

One could say that I've taken this idea way too seriously, but there's still lots of things that can be improved in AnnotationScript. These include, but are not limited to:

* Have less StackOverflowErrors. I chose to implement it functionally with a lot of recursion. AnnotationScript also requires recursion because it has no loops. In MetaScript, it's the same thing. This means that the stack builds up extremely quickly. In MetaScript's [FizzBuzz](https://github.com/jqno/AnnotationScript/blob/dc7962eea26dffb39d4eea799f56345b40d21151/src/main/java/nl/jqno/annotationscript/demo/MetaFizzBuzz.java), you get a StackOverflowError if you make it run past `60`. Given that FizzBuzz traditionally runs up to `100`, this is kind of a problem.
* Proper error handling. I touched on this before, but every time there is an error, the program crashes with a very unhelpful message. Working in MetaScript compounds that issue, because that has no error handling either, so when a MetaScript program fails, there are now two levels where the program can fail. Adding `println`s is basically the only option, but that's hard too, because if you want to add a `println` somewhere, you have to wrap some expression in a `begin` block and insert the print statement. That means you have to increment all the annotations in the wrapped expression with 1, which is quite annoying.
* Improve the generator's tokenizer (and, consequently, MetaScript's tokenizer as well), so that it supports strings containing spaces and parentheses. Though the generator helped a lot with translating the code from The Little Schemer to AnnotationScript, it was kind of annoying I couldn't use these symbols, and had to later edit them in manually.Also, it would be nice if the generator's pretty printer could preserve line-breaks somehow, because I later had to edit those in manually as well.
* The whole point of this exercise was to show that annotations can be abused by giving them too much power. Therefore, it would be nice to have proper Spring integration. Perhaps the return value from AnnotationScript programs can be `@Autowired` into Java properties. The sky is really the limit here!

<a id='conclusion'>
## Conclusion

First of all, sorry for the wall of text. I wanted to take you along for a ride with me, because it certainly has been a ride for me. It has taken me several months of giddiness and frustration to get from the initial, ridiculous idea to the actually working, equally ridiculous result: I have implemented a fully-functioning implementation of LISP, which can be expressed only using Java annotations, and then I have used that to implement an embedded, fully-functioning implementation of LISP.

If that isn't proof that annotations can be abused to do things that are way too powerful, I don't know what is.

I would like to leave you with [Greenspun's tenth rule](https://en.wikipedia.org/wiki/Greenspun%27s_tenth_rule), which I think sort of applies here:

> Any sufficiently complicated C or Fortran program contains an ad hoc, informally-specified, bug-ridden, slow implementation of half of Common Lisp.

Thanks for going along with me on this rollercoaster ride of ridiculousness.
