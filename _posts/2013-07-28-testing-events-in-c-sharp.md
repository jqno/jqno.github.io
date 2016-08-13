---
title: "Testing events in C#"
tags:
- c-sharp
- event-handling
- unit-testing
---
## TL;DR

This is a story about the pain of testing events in C#. If you don't have time to read the whole story, but you _are_ looking for an easy, reliable way to test (asynchronous) events, you can skip straight to [the end](#solution).


## The story

So I've been unit-testing events in C# lately, and it's been kind of a pain. Let's say we have this code:

{% highlight c# %}
public delegate void AnswerHandler(object sender, AnswerEventArgs e);
public class AnswerEventArgs : EventArgs
{
    public int Answer { get; set; }
}

public class AnswerFinder
{
    public event AnswerHandler AnswerEvent;

    public void FindTheAnswer()
    {
        int? answer = DeepThought.Compute();
        if (answer.HasValue)
        {
            var eventArgs = new AnswerEventArgs { Answer = answer.Value };
            AnswerEvent(this, eventArgs);
        }
    }
}
{% endhighlight %}

And we want to know whether we got the correct answer. We could write a unit test that looks something like this:

{% highlight c# %}
[Test]
public void Attempt1()
{
    var finder = new AnswerFinder();
    finder.AnswerEvent += (sender, e) =>
    {
        Assert.That(e.Answer, Is.EqualTo(42));
    };
    finder.FindTheAnswer();
}
{% endhighlight %}

Although--doesn't this cause a [memory leak](http://stackoverflow.com/q/4526829/127863)? Actually, no, because both the producer of, and the subscriber to the event, have the same life cycle and can be garbage collected together. Still, one could argue (and many people do) that it's good practice to explicitly unsubscribe from the event anyway: that way, you've got your bases covered if you do run into a situation where such a memory leak would occur. Or even worse: you could refactor your code, change the scope of either the `finder` or the event handler, and you forget to update all the places where either of them is used, and voilà: you've introduced a memory leak and you didn't even know it. Better safe than sorry, right?

So now we end up with something like this:

{% highlight c# %}
[Test]
public void Attempt2()
{
    var finder = new AnswerFinder();
    EventHandler<AnswerEventArgs> handler = (sender, e) =>
    {
        Assert.That(e.Answer, Is.EqualTo(42));
    };
    finder.AnswerEvent += handler;
    finder.FindTheAnswer();
    finder.AnswerEvent -= handler;
}
{% endhighlight %}

That will do the trick, right? Wrong. There's no guarantee an answer will be found. `DeepThought.Compute()` may simply give up at some point and return `null`, and the event will never be raised. Unfortunately, in that case, our assert is never executed, and the test will happily report success. So we have to guard against that:

{% highlight c# %}
[Test]
public void Attempt3()
{
    var finder = new AnswerFinder();
    int answer = -1;
    AnswerHandler handler = (sender, e) =>
    {
        answer = e.Answer;
    };
    finder.AnswerEvent += handler;
    finder.FindTheAnswer();
    Assert.That(answer, Is.EqualTo(42));
    finder.AnswerEvent -= handler;
}
{% endhighlight %}

Ouch. Also, what does that `-1` mean? Assuming that the answer is always an integer, are we sure it can't be negative? I'm certainly not. Also, we're conflating two properties here: whether or not the event was raised, and the actual answer. Let's fix that.

{% highlight c# %}
[Test]
public void Attempt4()
{
    var finder = new AnswerFinder();
    bool eventWasCalled = false;
    AnswerHandler handler = (sender, e) =>
    {
        eventWasCalled = true;
        Assert.That(e.Answer, Is.EqualTo(42));
    };
    finder.AnswerEvent += handler;
    finder.FindTheAnswer();
    Assert.That(eventWasCalled, Is.True);
    finder.AnswerEvent -= handler;
}
{% endhighlight %} 

That's an awful lot of boilerplate code for such a simple test! In fact, I'm having trouble seeing the actual test logic through all the boilerplate. Now, imagine if you have a comprehensive test suite with many tests like these. Or actually: please don't. It makes me sad.

What makes me even sadder, is that `DeepThought.Compute()` is quite an expensive operation. We don't want it to freeze the GUI; we should off-load it to a background thread. Great: now we have an asynchronous event. Let's replace the `bool` with a `ManualResetEventSlim`:

{% highlight c# %}
[Test]
public void Attempt5()
{
    var finder = new AnswerFinder();
    var resetEvent = new ManualResetEventSlim(false);
    AnswerHandler handler = (sender, e) =>
    {
        resetEvent.Set();
        Assert.That(e.Answer, Is.EqualTo(42));
    };
    finder.AnswerEvent += handler;
    finder.FindTheAnswer();
    Assert.That(resetEvent.Wait(TimeSpan.FromMilliseconds(500)), Is.True);
    finder.AnswerEvent -= handler;
}
{% endhighlight %}

OK, that's not so bad, right? But wait: `ManualResetEventSlim` implements `IDisposable`!

{% highlight c# %}
[Test]
public void Attempt6()
{
    var finder = new AnswerFinder();
    using (var resetEvent = new ManualResetEventSlim(false))
    {
        AnswerHandler handler = (sender, e) =>
        {
            resetEvent.Set();
            Assert.That(e.Answer, Is.EqualTo(42));
        };
        finder.AnswerEvent += handler;
        finder.FindTheAnswer();
        Assert.That(resetEvent.Wait(TimeSpan.FromMilliseconds(100)), Is.True);
        finder.AnswerEvent -= handler;
    }
}
{% endhighlight %}

Of course we could lessen that load by turning `resetEvent` into a field, and using `[SetUp]` and `[TearDown]` methods to initialize it and clean it up. But still a lot of boilerplate remains.

**EDIT** Are we done? I thought so, but as Ralph correctly pointed out in the comments, this still doesn't work. If the assert fails, it will throw an exception, yes--but it will do so in the background thread, not in NUnit's main thread. This means that NUnit doesn't pick up on it, and the behaviour that follows is undefined. It could report success, or it could fail to terminate at all. The only thing you can be certain of, is that the test won't actually fail in the way you want. So we have to move the assertion back into the main thread:

{% highlight c# %}
[Test]
public void Attempt7()
{
    var finder = new AnswerFinder();
    using (var resetEvent = new ManualResetEventSlim(false))
    {
        int answer = -1;
        AnswerHandler handler = (sender, e) =>
        {
            answer = e.Answer;
            resetEvent.Set();
        };
        finder.AnswerEvent += handler;
        finder.FindTheAnswer();
        Assert.That(resetEvent.Wait(TimeSpan.FromMilliseconds(100)), Is.True);
        Assert.That(answer, Is.EqualTo(42));
        finder.AnswerEvent -= handler;
    }
}
{% endhighlight %}

And now, we're stuck with that nasty `-1` again.

Thankfully, we can do better. A lot better.

<a name='solution'></a>

## A solution

I've written a small class that uses some clever reflection tricks to handle most of the boilerplate I've shown above. Behold:

{% highlight c# %}
[Test]
public void FinalAttempt()
{
    var finder = new AnswerFinder();
    AnswerHandler handler = (sender, e) =>
    {
        Assert.That(e.Answer, Is.EqualTo(42));
    };
    using (new EventMonitor(finder, "AnswerEvent", handler))
    {
        finder.FindTheAnswer();
    }
}
{% endhighlight %}

Neat, isn't it? You can find the code [below](#code). Maybe I'll do a proper version on GitHub some day, with its own test suite and proper documentation. Until then, this blog post will have to do :).

Here's a few things `EventMonitor` can do:

* It works both for synchronous and asynchronous events.
* You can give it a custom timeout in an optional constructor parameter. By default, it'll wait for 500 milliseconds.
* Even if you decide not to use the conventional `object sender, EventArgs e` delegates, it will work with any delegate with up to 4 parameters, as long as they have a `void` return type.
* If you want to check that the same event is fired twice, you can do that as follows:

{% highlight c# %}
[Test]
public void TheSameEventTwice()
{
    var finder = new AnswerFinder();
    AnswerHandler handler = (sender, e) =>
    {
        Assert.That(e.Answer, Is.EqualTo(42));
    };
    using (var monitor = new EventMonitor(finder, "AnswerEvent",
                                          handler, Mode.MANUAL))
    {
        finder.FindTheAnswer();
        monitor.Verify();

        // Let's do it again!
        finder.FindTheAnswer();
        monitor.Verify();
    }
}
{% endhighlight %}

Here's a few things `EventMonitor` unfortunately can't do:

* Unfortunately, it doesn't seem possible in C# to pass an event as a parameter. That's why I opted for the "stringly typed" option of passing the event's name as a string. If you know of a better way to handle this, I would very much like to hear about it!
* It's not fully thread-safe. If you have asynchronous events, and several of them are raised at the same time, `EventMonitor`'s behaviour will be undefined. If you raise only one asynchronous event, or if you ensure that they aren't raised simultaneously, it will be fine. This issue could be solved by employing a lock in the `Handle` method.

### Update, October 16th, 2013
Fixed the code, so that it will handle event delegates with primitive type arguments, as well as object type arguments, by adding generics to the solution, as suggested in [this StackOverflow question](http://stackoverflow.com/q/19346023/127863).

<a name='code'></a>

## Full code of `EventMonitor`

{% highlight c# %}
using System;
using System.Linq;
using System.Reflection;
using System.Threading;
using NUnit.Framework;
using NUnit.Framework.SyntaxHelpers;

namespace Test
{
	public enum Mode
	{
		MANUAL,
		AUTOMATIC
	}

	public class EventMonitor : IDisposable
	{
		private static readonly int MaximumArity = 4;

		private readonly object objectUnderTest;
		private readonly Delegate handler;
		private readonly TimeSpan timeout;
		private readonly Mode mode;

		private readonly ManualResetEventSlim resetEvent;
		private readonly EventInfo eventInfo;
		private readonly Delegate wrappedHandler;

		private Exception exception = null;

		public EventMonitor(object objectUnderTest, string eventName, Delegate handler, Mode mode = Mode.AUTOMATIC)
			: this(objectUnderTest, eventName, handler, TimeSpan.FromMilliseconds(500), mode)
		{ }

		public EventMonitor(object objectUnderTest, string eventName, Delegate handler, TimeSpan timeout, Mode mode = Mode.AUTOMATIC)
		{
			this.objectUnderTest = objectUnderTest;
			this.handler = handler;
			this.timeout = timeout;
			this.mode = mode;

			this.resetEvent = new ManualResetEventSlim(false);
			this.eventInfo = objectUnderTest.GetType().GetEvent(eventName);
			Assert.That(eventInfo, Is.Not.Null, string.Format("Event '{0}' not found in class {1}", eventName, objectUnderTest.GetType().Name));

			this.wrappedHandler = GenerateWrappedDelegate(eventInfo.EventHandlerType);
			eventInfo.AddEventHandler(objectUnderTest, wrappedHandler);
		}

		public virtual void Dispose()
		{
			if (mode == Mode.AUTOMATIC)
			{
				Verify();
			}
			eventInfo.RemoveEventHandler(objectUnderTest, wrappedHandler);
			resetEvent.Dispose();
		}

		public void Verify()
		{
			if (exception != null)
			{
				throw exception;
			}
			Assert.That(resetEvent.Wait(timeout), Is.True, string.Format("Event '{0}' was not raised!", eventInfo.Name));
			resetEvent.Reset();
		}

		private Delegate GenerateWrappedDelegate(Type eventHandlerType)
		{
			var method = eventHandlerType.GetMethod("Invoke");
			var parameters = method.GetParameters();
			int arity = parameters.Count();
			Assert.That(arity, Is.LessThanOrEqualTo(MaximumArity), string.Format("Events of arity up to {0} supported; this event has arity {1}", MaximumArity, arity));
			var methodName = string.Format("Arity{0}", arity);
			var eventRegisterMethod = typeof(EventMonitor).GetMethod(methodName, BindingFlags.NonPublic | BindingFlags.Instance);
			if (arity > 0)
			{
				eventRegisterMethod = eventRegisterMethod.MakeGenericMethod(parameters.Select(p => p.ParameterType).ToArray());
			}
			return Delegate.CreateDelegate(eventHandlerType, this, eventRegisterMethod);
		}

		private void Handle(Action action)
		{
			try
			{
				action();
			}
			catch (Exception e)
			{
				exception = e;
			}
			resetEvent.Set();
		}

		private void Arity0()
		{
			Handle(() => handler.DynamicInvoke());
		}

		private void Arity1<T>(T arg1)
		{
			Handle(() => handler.DynamicInvoke(Convert.ChangeType(arg1, arg1.GetType())));
		}

		private void Arity2<T1, T2>(T1 arg1, T2 arg2)
		{
			Handle(() => handler.DynamicInvoke(
					Convert.ChangeType(arg1, arg1.GetType()),
					Convert.ChangeType(arg2, arg2.GetType())));
		}

		private void Arity3<T1, T2, T3>(T1 arg1, T2 arg2, T3 arg3)
		{
			Handle(() => handler.DynamicInvoke(
					Convert.ChangeType(arg1, arg1.GetType()),
					Convert.ChangeType(arg2, arg2.GetType()),
					Convert.ChangeType(arg3, arg3.GetType())));
		}

		private void Arity4<T1, T2, T3, T4>(T1 arg1, T2 arg2, T3 arg3, T4 arg4)
		{
			Handle(() => handler.DynamicInvoke(
					Convert.ChangeType(arg1, arg1.GetType()),
					Convert.ChangeType(arg2, arg2.GetType()),
					Convert.ChangeType(arg3, arg3.GetType()),
					Convert.ChangeType(arg4, arg4.GetType())));
		}
	}
}
{% endhighlight %}
