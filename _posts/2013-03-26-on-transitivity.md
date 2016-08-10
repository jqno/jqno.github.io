---
title: "On transitivity"
tags:
- equals
- equalsverifier
- java
---
[Last time]({{ pcposturl(2013, 02, 17, "reaction-to-cedric-beusts-equals-challenge") }}), I discussed a transitivity issue for `equals` methods that also affects [EqualsVerifier](http://www.jqno.nl/equalsverifier). To recap, consider this class:

<pre class="prettyprint">
class School {
    private final String name;
    private final String nickname;

    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof School)) {
            return false;
        }
        School other = (School)obj;
        return name.equals(other.name) || nickname.equals(other.nickname);
    }
}
</pre>

This class violates the transitivity requirement: if `x` equals `y`, and if `y` equals `z`, then `x` must equal `z`. It's easy to see why:

* `new School("A". "1")` is equal to `new School("B", "1")`
* `new School("B", "1")` is equal to `new School("B", "2")`
* `new School("A", "1")` is **NOT** equal to `new School("B", "2")`

It turned out that EqualsVerifier was unable to detect this case. At the end of my post, I resolved to fix [this issue](https://code.google.com/p/equalsverifier/issues/detail?id=75) quickly. Since I raised the issue on this blog, I felt I should publish the solution here as well. It took me a little while, but here's the solution! I won't bore you with the various detours I took while finding it and get straight to the point.

Approach
----
EqualsVerifier works by creating intances of classes the same way mocking frameworks like Mockito and EasyMock do: using bytecode trickery library [Objenesis](http://objenesis.googlecode.com/svn/docs/index.html). Using reflection, it then 'fills' these instances with certain values. How this works exactly is a topic for another day, but it is important to know that for most types, EqualsVerifier either has 2 pre-fabricated values it can use for this, or it can construct these two values. In some cases, though, EqualsVerifier can't construct any values, and the user has to supply them using the `withPrefabValues` method.

EqualsVerifier uses carefully chosen combinations of values to create instances, and calls `equals` and `hashCode` on these instances to find out whether certain properties of the contract are met. The idea, therefore, is to construct instances of a class in such a way that they expose our transitivity issue.

A partial solution
----
Let's try to solve the case for 2 fields first. This relatively easy. Above, we created three instances of the `School` class that I called `A1`, `B1` and `B2`, partly because I was too lazy to come up with actual names of schools, but also partly because this nicely shows how the issue can be approached for any class with two fields: we could have chosen any two values valid for the types at hand, as long as they are distributed according to the `A1`, `B1`, `B2` pattern.

As mentioned above, transitivity requires that if `x.equals(y) && y.equals(z)`, then also `x.equals(z)`. The key insight here is that the order doesn't matter: if `y.equals(z) && z.equals(x)` then `y.equals(x)` and if `z.equals(x) && x.equals(y)` then `z.equals(y)`, etc. In other words, it doesn't matter in which order we perform the `equals` checks.

So there are three checks we need to perform: `x.equals(y)`, `y.equals(z)`, and `x.equals(z)`. (We're going to assume that symmetry holds; if not, there's another EqualsVerifier test that will catch that.) So, if two of these checks are true, the third one has to be true as well, or it's non-transitive. In other words: if exactly two out of these three checks are true, `equals` is non-transitive. If all three are true, it's ok; if only one or zero are true, it's also ok. You can verify this easily in the following table:

    A1==B1 && B1==B2 && A1==B2
    A1==B1 && B1==B2 && A1!=B2  ← not ok
    A1==B1 && B1!=B2 && A1==B2  ← not ok
    A1==B1 && B1!=B2 && A1!=B2
    A1!=B1 && B1==B2 && A1==B2  ← not ok
    A1!=B1 && B1==B2 && A1!=B2
    A1!=B1 && B1!=B2 && A1==B2
    A1!=B1 && B1!=B2 && A1!=B2

The full solution
----
Now that we have the two-field solution, a more general solution is within reach: we simply have to pretend that the class only has two fields. How do we do that? Simple: we apply the two-field solution separately for each field in the class. We can iterate over the fields, where the current field is the 'first' value, and all other fields together represent the 'second' value. So if the second value changes, all fields (except the current one) change.

Let's say we have a class with four fields, `f`, `g`, `h` and `i`. We'll represent them with A/B, 1/2, α (alpha)/β (beta) and ا (alif) /ب (ba), respectively. Then we have to solve the following cases:

           fghi     fghi     fghi
    f:  A1=A1αا  B1=B1αا  B2=B2βب
    g:  A1=A1αا  B1=A2αا  B2=B2βب
    h:  A1=A1αا  B1=A1βا  B2=B2βب
    i:  A1=A1αا  B1=A1αب  B2=B2βب

Note how `A1` and `B2` are the same in all cases, and how the "`B`" ripples through the `B1` values in the middle column.

If we perform the two-field solution for each of these four cases, we're done. All of these cases must be transitive, as per the partial solution. If one of them isn't, the `equals` method is bad and EqualsVerifier will say so.

Conclusion
----
I had no idea that Cedric Beust's coding challenge would be such a challenge. But I'm glad that it was; EqualsVerifier benefitted from it, not just because a subtle bug was finally fixed, but also because I discovered several other areas for improvement along the way, which I will address in future releases.

In the mean time, I would like to urge you to update your `pom` files to version 1.2, and see if your `equals` methods are truly transitive! :)
