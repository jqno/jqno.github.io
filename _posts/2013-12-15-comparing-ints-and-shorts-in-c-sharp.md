---
title: "Comparing ints and shorts in C#"
excerpt: In which I find a possible bug in the language.
tags:
- c-sharp
- equals
---
My colleague Ralph and I recently discovered an interesting bit of C# equality behaviour. Consider the following piece of code:

{% highlight c# %}
[Test]
public void Equals()
{
    int i = 42;
    short s = 42;

    Assert.IsTrue(i == s);
    Assert.IsTrue(s == i);
    Assert.IsTrue(i.Equals(s));
    Assert.IsTrue(s.Equals(i)); // fails
}
{% endhighlight %}

One would expect the last assert to pass, just like the others. The fact that it doesn't, tells us two things:

* The behaviour of `==` and `Equals` is different on C#'s primitive integral types. At least, when comparing objects of (slightly) different types.
* `Equals` is not symmetric when comparing `int`s with `short`s.

Both are surprising to me. While I don't like pointing out bugs in things like the .NET Framework (it's like farting: he who points it out, is usually at fault himself), these do seem to qualify; especially the a-symmetry in Equals, which [violates the contract](http://msdn.microsoft.com/en-us/library/bsc2ak47%28v=vs.90%29.aspx).

There's probably some arcane piece of .NET legacy at work here, so of course, I sent a question to [Eric Lippert](http://ericlippert.com/2013/07/17/ask-the-bug-guys/). I hope I'll get a response; I'll post an update if and when I do.

In the mean time: can you, dear readers, offer an explanation?
