---
title: "Reaction to Cedric Beust's equals challenge"
tags:
- equals
- equalsverifier
- java
---
This week, I got [nerd sniped](http://xkcd.com/356/) by Cedric Beust's latest [coding challenge](http://beust.com/weblog/2013/02/13/coding-challenge-light-edition/):

> A `School` has either a name (“Stanford”) or a nickname (“UCSD”) or both.
>
> Your task is to write `equals()` and `hashCode()` methods for the `School`
> class that satisfy the following requirement: two Schools are identical if
> they either have the same name or the same nickname.

I wrote [EqualsVerifier](http://www.jqno.nl/equalsverifier); I should be able to do this! I came up with the following, which I thought was pretty ironclad:

{% highlight java %}
public final class School {
    private final String name;
    private final String nickname;

    public School(String name, String nickname) {
        this.name = name;
        this.nickname = nickname;
    }

    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof School)) {
            return false;
        }
        School other = (School)obj;
        return nullSafeEqual(name, other.name) ||
                nullSafeEqual(nickname, other.nickname);
    }

    private boolean nullSafeEqual(String a, String b) {
        return a == null ? b == null : a.equals(b);
    }
    
    @Override
    public int hashCode() {
        return 42;
    }

    @Override
    public String toString() {
        return "School: name=" + name + ", nickname=" + nickname;
    }
}
{% endhighlight %}

I even added a `toString()`! Obviously, I wasn't really happy about the `hashCode()` method, but I wasn't really sure how to write it. Essentially, this implementation will turn your efficient hash collection with O(1) lookup into a list with O(n) lookup. So yeah, that's pretty bad. But at least it meets the contract, so I thought I'd fix that later; first I wanted to see if my `equals()` was correct. So I defined some tests with EqualsVerifier:

{% highlight java %}
@Test
public void testEquals() {
    School one = new School("A", "1");
    School two = new School("A", "2");
    School three = new School("B", "2");

    School x = new School("X", "0");

    EqualsVerifier.forRelaxedEqualExamples(one, two)
            .andUnequalExample(x)
            .verify();

    EqualsVerifier.forRelaxedEqualExamples(two, three)
            .andUnequalExample(x)
            .verify();
}
{% endhighlight %}

So far so good! I had to use the slightly verbose `forRelaxedEqualExamples()` mode of EqualsVerifier here, because the regular case is meant for classes that are equal only when all their fields are equal, which is obviously not the case here: the equality relation defined by Beust is more relaxed -- hence the name.

Then I decided to try and combine the two calls to EqualsVerifier:

{% highlight java %}
EqualsVerifier.forRelaxedEqualExamples(one, two, three)
        .andUnequalExample(x)
        .verify();
{% endhighlight %}

And that's where my test failed: `Precondition: not all objects are equal`. And that's true: `one` and `three` aren't equal to each other. But they should be, because [the contract clearly states](http://docs.oracle.com/javase/6/docs/api/java/lang/Object.html#equals%28java.lang.Object%29):

> It is _transitive_: for any non-null reference values
> `x`, `y`, and `z`, if
> `x.equals(y)` returns `true` and
> `y.equals(z)` returns `true`, then
> `x.equals(z)` should return `true`.

So there you have it: this equality relation isn't transitive. Therefore, it's not possible to write an `equals()` method that follows Beust's requirement, and still meet the contract. Was the challenge a trick question?

This also explains why I had trouble with my `hashCode()` method: equal objects must always return the same hashCode. So if `one` and `three` are somehow equal, without having anything in common except the link with `two`, then the only possible hashCode is a constant.

Now you might ask: so what if my `equals()` method isn't transitive? If this is the kind of equality semantics that I want, then why can't I have it? It's a free country!

And that's true: you're free to break the equals contract. But, as with any contract, if you break it, there will be consequences. In this case, the consequences will be subtle, hard-to-track bugs. [One of the commenters](http://beust.com/weblog/2013/02/13/coding-challenge-light-edition/#comment-17073) had a good example of one such bug. Consider the following code:

{% highlight java %}
Set<School> red = new HashSet<School>();
red.add(one);
red.add(two);
red.add(three);

Set<School> black = new HashSet<School>();
black.add(two);
black.add(one);
black.add(three);

System.out.println("Collection red has " + red.size() + " elements.");
System.out.println("Collection black has " + black.size() + " elements.");
{% endhighlight %}

Can you guess what that prints? Actually, you can't, because it depends on the implementation of `HashSet`. When you add a key to a `HashSet` that's already present in the set, the implementation can do two things. Either, it keeps the original one, in which case it will print:

    Collection one has 2 elements.
    Collection two has 1 elements.

Or it replaces the old one with the new one, in which case the order will be reversed.

The moral of the story: writing a good `equals` method is hard; don't try to make it fancy.

If you want to have a non-transitive equality test, that's fine, but don't abuse `equals()` to achieve that. There are other ways. One of them, as Beust points out in his [follow-up post](http://beust.com/weblog/2013/02/16/answer-to-the-school-challenge/), is to add an `id` field to the `School` class and use that to base `equals()` and `hashCode()` on. Another way could be to simply add another method to the class:

{% highlight java %}
public boolean isSameSchool(School other) {
    return nullSafeEqual(name, other.name) ||
            nullSafeEqual(nickname, other.nickname);
}
{% endhighlight %}

It even looks a lot better, because you don't need all that type checking stuff anymore.

P.S.
----
Just a few paragraphs ago I was saying how I needed to use EqualsVerifier's `forRelaxedEqualExamples` mode, because of the relaxed nature of this equality relation, and that the regular mode _obviously_ wouldn't work. I don't know what made me decide to test that, anyway:

{% highlight java %}
@Test
public void obviouslyThisTestShouldNotPass() {   // obviously!
    EqualsVerifier.forClass(School.class).verify();
}
{% endhighlight %}

It passed.

I'm going to have to [fix that](https://code.google.com/p/equalsverifier/issues/detail?id=75)...
