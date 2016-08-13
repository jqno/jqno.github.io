---
title: "Generics in EqualsVerifier, part 1: Overcoming type erasure"
tags:
- equalsverifier
- generics
---
This is part 1 of a two-part series. This part deals with overcoming type erasure. In [Part 2]({% post_url 2016-06-26-generics-in-equalsverifier-part-2 %}), we will see what EqualsVerifier can do with this generic type information.

Introduction
---
Since version 2.0, EqualsVerifier is aware of generics. For instance, it can determine, at runtime, whether a class's field is a `List<String>` or a `List<Integer>`. However, type erasure says this is impossible! If we call `getClass()` on a `list` instance, we'll get a `List.class` object, which is unaware of the generic paramter. How does EqualsVerifier do this?

Some background
---
EqualsVerifier works by creating instances of objects, filling their fields with carefully chosen values, and repeatedly calling `equals` on them to see if something unexpected comes up. However, type erasure causes problems. Say we have the following class:

{% highlight java %}
class ListContainer {
    private final List<String> list;
}
{% endhighlight %}

EqualsVerifier can see that this class has a field `list` of type `List`. Versions below 2.0 don't know anything more than that: they can't determine that it's really a `List<String>`, because the generics get erased by the JVM. To work around this, EqualsVerifier simply instantiates a raw `List`, puts in a few values of type `Object`, and hopes nobody notices.

In most cases this works perfectly fine, because in most cases, an `equals` method will just call `list.equals(other.list)`. `List`'s `equals` method then simply calls `equals` on each of its values and it doesn't matter what the type of these values is. After all, every Java class has an `equals` method that you can call.

In a small number of cases, this doesn't work. For example, Android has a type called `SparseArray`. It's a generic type that contains a sequence of values, like an array or a list. However, unlike arrays and lists, it doesn't implement its own `equals` method. Calling `equals` on a `SparseArray` is like calling `==` on it: it doesn't matter if two `SparseArrays` contain exactly the same elements; if they're not the same instance, they're not equal. This means that a class with a `SparseArray` field, has to 'unwrap' it in `equals`:

{% highlight java %}
public class SparseArrayContainer {
    private final SparseArray<String> sparseArray;
    
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
{% endhighlight %}

Now, we get a `ClassCastException` in the line that assigns an element of `sparseArray` to `a`: `a` expects a `String`, but it gets an `Object`. Oops.

Because the generic type is erased at runtime, there's no way around this issue. Or is there?

There is!
---
While it's true for objects at runtime that their generic type gets erased, there is something we can use. In EqualsVerifier, we're always inspecting a class and its fields, and it turns out that Java does retain the fully generic type of all fields in a class. You can use reflection to access this information. So, for our `SparseArrayContainer`, we can do this:

{% highlight java %}
Field f = SparseArrayContainer.class.getDeclaredField("sparseArray");
Type type = f.getGenericType();
if (type instanceof ParameterizedType) {
    ParameterizedType pt = (ParameterizedType)type;
    Type[] genericTypes = pt.getActualTypeArguments();
} 
{% endhighlight %}

The `type` variable has type `java.lang.reflect.Type`, which is an interface with several impementations. The most important implementation, and also the easiest, is good old `java.lang.Class`. However, our `sparseArray` field is parameterized, so we'll get an instance of `ParameterizedType`.

On a `ParameterizedType` (don't forget to cast first; `java.lang.reflect.Type` doesn't declare a lot of useful methods by itself), we can call a `getActualTypeArguments()` method, which gives us an array of `Type`. The elements of this array are instances of `java.lang.Class`. (As said before, `java.lang.Class` implements the `java.lang.reflect.Type` interface). In other words, `genericTypes[0].equals(String.class)`. Yay! We have the generic type we want. The rest is just an exercise of filling in the blanks.

It turns out, there's a lot of blanks to fill in.

TypeTag
---
First, we need to be able to pass around our new generic type information, where EqualsVerifier used to just pass around a `java.lang.Class`. `java.lang.reflect.Type` has all the information we need, but it's very unwieldy. Time then to build our own container: `TypeTag`. It looks something like this:

{% highlight java %}
public final class TypeTag {
    private final Class<?> type;
    private final List<TypeTag> genericTypes;

    public TypeTag(Field field) { ... }
    
    // Getters, equals, hashCode left out for brevity.
}
{% endhighlight %}

In order to construct an instance, we'll need a `java.lang.reflect.Field`, because as we said before, we need that to access the generic types. (There are other ways to get them; for example, if we can make a subclass of a type and instantiate that, we can use that to instance to find the generic types. However, sometimes types are final and in those cases, making a subclass is impossible. Using a field is much more reliable.)

It does mean that we can't get the generic types at the top of the chain, though. Say we have the following class:

{% highlight java %}
public final class Entity<I extends Comparable<I>> {
    private final I id;
    
    // Constructor, getters, equals, hashCode left out for brevity.
}
{% endhighlight %}

There is no way to figure out that `Entity` has a generic type parameter `I extends Comparable<I>` if we only have an instance of `Entity`. Fortunately, this problem is not as big as it seems, because in all cases, we only need to know the type parameter when it's used. And when the type parameter is used, it's used in a field. And we _can_ get at the type parameters of fields!

However, we're not done yet. Apart from `java.lang.Class` and `java.lang.reflect.ParameterizedType`, our `java.lang.reflect.Type` interface has a bunch more implementations, and we need to consider each one.

TypeVariable
---
Consider once again the `Entity` class above. When we have a `java.lang.reflect.Field` instance of its `id` field, we will get an instance of `java.lang.reflect.TypeVariable`, which is another implementation of the `java.lang.reflect.Type` interface. This gives us the name (in this case a String `"I"`) and its bounds (`Comparable<I>`, wrapped in some other instance of `java.lang.reflect.Type`). The bounds are important, because we can't assign a `java.lang.Object` to `id`, because that's not `Comparable`. Note that type variables can be bound to concrete classes (`T extends Object`), but also to other type variables (`T extends U`), which complicates matters. To make things even worse, bounds are often even recursive: `T extends Comparable<T>`!

And it gets more complicated. Consider the following (not even very contrived) example:

{% highlight java %}
public interface Period { }
public final class Per<T extends Comparable<T>, P extends Period> {
    private final P period;
    private final T value;
    
    // Constructor, getters, equals, hashCode left out for brevity.
}
{% endhighlight %}

When we evaluate the types of `period` and `value`, we'll have to match them up with `Per`'s generic parameters. Note that I switched the order around, to emphasize that we can't simply say that the first field we encounter in the class will match with the first generic parameter, and the second field with the second generic parameter. No, we'll have to match the field's type's name with the generic parameter's name.

Fortunately, there is one thing we _can_ get from a raw `java.lang.Class` object: the `java.lang.reflect.TypeVariable`s it was declared with. `Per.class.getTypeParameters()` returns an array of `TypeVariable`.

So, if we want to determine the precise type of `Per`'s `value` field, we need `value`'s corresponding `java.lang.reflect.Field`, and `Per`'s `TypeVariable`. We'll call the `Per` class `value`'s _enclosing class_. We'll put `Per`'s `TypeVariable`s in a hash map, keyed on the names of the type variables, so we can more easily match them with the types of the fields.

Other implementations of the `Type` interface
---
The next `java.lang.reflect.Type` implementation we have to consider is the wildcard, which can also have bounds. Fortunately, the wildcard only occurs on fields, not on classes, so it's a bit easier:

{% highlight java %}
private final List<? extends Point> points;
{% endhighlight %}
    
We can safely substitute a boundless wildcard with `java.lang.Object`, and a bounded wildcard with the bound itself. In the case of the `List<? extends Point>` above, we can simply pretend it's a `List<Point>`. Note that wildcards, as opposed to `TypeVariable`s, can have upper (`? extends SomeType`) _and_ lower (`? super SomeType`) bounds. Fortunately, we can treat them the same in this case.

Finally, there's the `GenericArrayType`. Fortunately, that one is pretty straightforward compared to the rest. We'll not look at it in detail.

What do we have now?
---
All this effort allows us to determine the complete type of any field, and take from that the information that is relevant to EqualsVerifier. For example, take this class:

{% highlight java %}
public class Container<T extends Comparable<T>> {
    private final List<String> a;
    private final Map<String, List<Integer>> b;
    private final List<T> c;
    private final List<?> d;
    private final List<? super Class<?>> e;
    private final T[] f;
}
{% endhighlight %}

This gives us the following `TypeTag`s for its fields:

    TypeTag(List, TypeTag(String))
    TypeTag(Map, TypeTag(String), TypeTag(List, TypeTag(Integer)))
    TypeTag(List, TypeTag(Comparable))
    TypeTag(List, TypeTag(Object))
    TypeTag(List, TypeTag(Class))
    TypeTag(Comparable[])

`TypeTag` has a factory method that takes a `java.lang.reflect.Field` and another `TypeTag` that represents the enclosing type, which we need to resolve `TypeVariable`s like `T`. Then it determines what kind of `java.lang.reflect.Type` the field is, and recursively resolves it. You can look at the implementation [here](https://github.com/jqno/equalsverifier/blob/d15e4a06242a104da60be19e56849e1cf24759c9/src/main/java/nl/jqno/equalsverifier/internal/prefabvalues/TypeTag.java#L68).

A caveat
---
You might have noticed that I haven't discussed multiple bounds, like `T extends Interface1 & Interface2`, which are also allowed in Java generics. In all honesty, I only thought of them while researching this article. EqualsVerifier 2.0 has been out for several months by now, so I suppose these multiple bounds aren't used often enough for people to have run into this. Nevertheless, it's obviously quite high on my list of future improvements.

Summary
---
We have seen how we can determine the full, generic type of a class's field. In [Part 2]({% post_url 2016-06-26-generics-in-equalsverifier-part-2 %}), we will see how we can put this information to good use.

