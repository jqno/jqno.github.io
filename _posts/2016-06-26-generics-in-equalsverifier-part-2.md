---
title: "Generics in EqualsVerifier, part 2: Instantiating generic classes"
tags:
- equalsverifier
- generics
---
In [Part 1]({% post_url 2016-06-23-generics-in-equalsverifier-part-1 %}) of this two-part series, we have explored how to determine the full, generic type of a class's fields. Now it's time to do something with this information: instantiate an object of any given type.

Introduction
---
We want to dynamically create objects of the given, generic type. In other words, if we have a TypeTag(ArrayList, TypeTag(String)), we want to construct an object that is an `instanceof ArrayList`. Also, we want it to contain an element of type `String`. EqualsVerifier uses these instances to do 'black box' testing: it assigns certain values to the fields of an object, calls `equals` on the object, and checks if the result was expected.

To instantiate an object, EqualsVerifier uses a library called [Objenesis](http://objenesis.org). This lets us instantiate an object without calling the constructor, leaving the fields uninitialized. This is necessary, because determining the required parameters for the constructor, and cobbling together values for them, can be hard. Also, a constructor might have side-effects which are undesirable in the context of testing `equals`. The purpose of a constructor is to make sure that an object is in a 'consistent state' after creation, but we're not usually interested in that either (unless the class actually enforces that consisten state; more about that later): we're going to alter the values of the fields anyway.

To alter the fields, EqualsVerifier simply uses reflection. This allows us to change the value of a field to whatever value we desire, as long as it has a compatible type. It doesn't matter if the field is private; reflection can easily bypass that.

This process works well for simple types, but with generics added to the mix, some interesting challenges present themselves. Let's first take a look at instantiating non-generic classes, though.

Non-generics classes
---
Let's say we want to instantiate the following class:

<pre class="prettyprint">
public class Point {
    private final int x;
    private final int y;

    // Constructor, getters, equals, hashCode left out for brevity.
}
</pre>

It's easy: with Objenesis, we get an instance of `Point` with `x` and `y` set to their default values (`0`). Using reflection, we can give `x` and `y` a value.

Sometimes, this doesn't work. For example, when there is a recursion. The simplest example of that is the `Node`:

<pre class="prettyprint">
public class Node {
    private final Node n;

    public Node(Node n) {
        this.n = n;
    }
}
</pre>

This would result in an infinite loop of trying to instantiate `Node`. To avoid this, EqualsVerifier detects these infinite loops and aborts when it encounters them. To get around this, EqualsVerifier keeps a collection of so-called 'prefab values', where the user can manually add pre-fabricated instances of these classes. Before trying to instantiate a type, EqualsVerifier always checks the prefab values to see if there's a value it can use.

After successfully instantiating a type, it adds the newly created instance to the collection, so it doesn't have to instantiate the same type twice. EqualsVerifier also keeps a list of standard Java API classes that are known to have recursions (or are hard to construct for other reasons), which saves the users a lot of manual prefab value adding.

But there are other reasons why using Objenesis and reflection to create instances might not work. For instance, when a class is abstract we will get `AbstractMethodError`s. Also, some classes might have strict class invariants. `ArrayList`, for example, might throw `ConcurrentModificationExceptions` when elements aren't added in the proper way. Reflection obviously bypasses this 'proper way'.

Since Java's Collections API contains many recursive types, abstract methods and class invariants, EqualsVerifier keeps a list of prefab values for all of Java's collections, and also for most of Google Guava's collections.

Generic classes
---
Keeping track of generics makes maintaining prefab values a lot harder. In the past, EqualsVerifier could simply instantiate an `ArrayList` and add a bunch of `java.lang.Object` instances. Because of type erasure, at runtime every `ArrayList` is essentially an `ArrayList<Object>` anyway.

Most of the time that would work. It would only fail when the list is 'unpacked', like in the `SparseArray` example in [Part 1]({% post_url 2016-06-23-generics-in-equalsverifier-part-1 %}).

Now that we do keep track of generics, it doesn't work anymore. An `ArrayList<String>` is no longer the same thing as an `ArrayList<Integer>`. That means a single instance of `ArrayList` is no longer sufficient. We need an instance of _all_ the `ArrayLists`, which is clearly impossible.

Factories
---
So we have to add an extra layer on top of the collection of prefab values: we need a collection of _factories_ for prefab values. Whenever an `ArrayList<Integer>` is encountered, EqualsVerifier now determines its `TypeTag` and gives it to the factory. The factory instantiates a raw `ArrayList` and finds a value for `Integer` which it adds to the `ArrayList`.

This is actually a recursive process. If EqualsVerifier encounters an `ArrayList<HashSet<Integer>>`, it will instantiate the `ArrayList` and then recursively hand over a `TypeTag` for `HashSet<Integer>` to the factory, which will instantiate a `HashSet`, recursively find an instance for `Integer` (which is easy), and add that `Integer` to the `HashSet`. Then it adds the new instance of `HashSet<Integer>` to the `ArrayList`, and return that.

How do we instantiate these collections? For `ArrayList`, this is easy: we can call its parameterless constructor. This is also true for many collections, but not for all of them. `EnumSet`, for instance, can only be instantiated with a factory method which requires very specific arguments. Therefore, we need a separate factory for `EnumSet` which knows about the way it can be instantiated.

Also, there's the problem of adding values to the collection. For lists and sets it's easy: we just call the `add` method. For maps it's also easy: we call the `put` method (although now we need two values instead of just the one). But it means that here too, we now need two different factories: one for lists and sets, and one for maps.

In other words, we need a lot of different factories for different kinds of classes.

Finally, every value created by the factory is added to the original collection of prefab values. Therefore, it could happen that it contains instances for `ArrayList<Integer>` and `ArrayList<HashSet<String>>`, but not `ArrayList<String>`, for example. It depends on the specific generic types that were encountered.

Other considerations
---
In a [previous post]({% post_url 2014-08-20-the-things-we-do-for-compatibility %}), I have explained how EqualsVerifier uses reflection to create prefab values for classes that it wasn't compiled with, and that may or may not be available at runtime. Of course there have to be separate factories for these types as well. I will not discuss these in this article, but you're welcome to take a look at [the code](https://github.com/jqno/equalsverifier/blob/master/src/main/java/nl/jqno/equalsverifier/JavaApiPrefabValues.java#L274).

Putting it all together
---
We have a bunch of different factories for a bunch of different purposes, and we need to manage which `TypeTag`s can be handled by which factory. Managing this isn't too hard (we just keep a `Map<Class<?>, Factory>`), but as we've seen, the number of factories quickly adds up.

Still, it frequently happens that EqualsVerifier encounters a `TypeTag` for which it has no factory. In this case, it will attempt the old strategy of instantiating the class with Objenesis and then filling its fields, either by using instances in its cache, or by recursively trying to instantiate those.

When this doesn't work, the only alternative is to ask the user to supply instances using EqualsVerifier's `withPrefabValues` method. These instances go directly into the cache. However, this doesn't scale to generic types that are defined by the user; the user can currently only supply one instance per type. That means if a class has a field of type `MyType<String>` as well as one of type `MyType<Integer>`, and both the `String` and the `Integer` are 'unpacked' inside `equals` or `hashCode`, EqualsVerifier can't deal with that.

This is a known limitation which fortunately doesn't come up a lot in practice. However, it will still be addressed in a future version of EqualsVerifier.

Summary
---
In [Part 1]({% post_url 2016-06-23-generics-in-equalsverifier-part-1 %}), we have explored how we can determine the full generic type of a field in a class. In this part, we have seen how we can leverage this information to build instances of objects that respect these generics.

Because of this, users of EqualsVerifier have more freedom to implement their `equals` and `hashCode` methods, since it is now possible to directly refer to the generic components of fields, which was not possible before. For instance:

<pre class="prettyprint">
public class SparseArrayContainer {
    private final SparseArray&lt;String> sparseArray;
    
    // ...
    
    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof SparseArrayContainer)) {
            return false;
        }
        SparseArrayContainer other = (SparseArrayContainer)obj;
        for (int i = 0; i < sparseArray.size(); i++) {
            String a = sparseArray.get(i);
            String b = other.sparseArray.get(i);
            if (!a.equals(b)) {
                return false;
            }
        }
        return true;
    }
}
</pre>

When tested by older versions of EqualsVerifier, this class would throw a `ClassCastException` on the line `String a = sparseArray.get(i);`, because `sparseArray` would contain instances of `java.lang.Object` instead of `String`. As of version 2.0, EqualsVerifier recognises that `sparseArray` is a `SparseArray` of `String`, and fills it with `String`s instead of `Object`s, thus preventing the `ClassCastException`.

All this leads to up to the final conclusion of this series:

Conclusion
---
Java generics are kinda hard.

