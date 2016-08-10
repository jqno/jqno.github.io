---
title: "Hacking Java enums"
tags:
- java
- enum
- equalsverifier
- objenesis
---
The other day, I was debugging some enum related code in EqualsVerifier. I had this enum:

<pre class="prettyprint">
enum CallMe { YES, NO, MAYBE }
</pre>

And two variables, `original` and `clone`, containing a value of said enum. Here's what the bug looked like in Eclipse's debugger:

![Call Me Maybe]({{ assets['CallMeMaybe'] }})

So, what do we see here? We see two variables of type `EnumHack$CallMe` (the enum was an inner class, so that makes sense). Both enums have the same `name` and `ordinal`, so they are equal. They also have different ids (33 and 34, respectively), so they're not the same object.

Wait, what!?

That's right: two different instances of the same enum constant. In other words, `original.equals(clone)` returns `false`, even though both variables are set to `CallMe.MAYBE`. How is that even possible? I thought it wasn't. In the words of [Joshua Bloch](http://www.amazon.com/Effective-Java-Edition-Joshua-Bloch/dp/0321356683):

> This approach [of using a one-element enum to implement a singleton, JO] provides an ironclad guarantee against multiple instantiation, even in the face of sophisticated serialization or reflection attacks. 

The [Java Language Specification, section 8.9](http://docs.oracle.com/javase/specs/jls/se7/html/jls-8.html#jls-8.9) says this:

> An enum type has no instances other than those defined by its enum constants. It is a compile-time error to attempt to explicitly instantiate an enum type (§15.9.1).
> 
> The final clone method in Enum ensures that enum constants can never be cloned, and the special treatment by the serialization mechanism ensures that duplicate instances are never created as a result of deserialization. Reflective instantiation of enum types is prohibited. Together, these four things ensure that no instances of an enum type exist beyond those defined by the enum constants. 

Clearly, despite the fact that an enum constant can never be cloned, I had a clone on my hands.

So what happened? I traced the problem back to this:

<pre class="prettyprint">
@Test
public void hackAnEnum() throws Exception {
    CallMe original = CallMe.MAYBE;
    CallMe clone = ObjectAccessor.of(original).copy();
    
    assertEquals(original.name(), clone.name());
    assertEquals(original.ordinal(), clone.ordinal());
    assertFalse(original.equals(clone));
    assertFalse(original == clone);
}
</pre>

I added the asserts so you can see for yourself what's going on: if you copy/paste this to your IDE and put EqualsVerifier on the classpath, you'll be able to run it. This test passes, which means that both `original.equals(clone)` and `original == clone` are, indeed, false.

So what does this `ObjectAccessor` do? It's part of EqualsVerifier's reflection library, and as you probably guessed, it makes a copy of the given object. If I factor out all EqualsVerifier code, I end up with this:

<pre class="prettyprint">
CallMe clone = new ObjenesisStd().newInstance(CallMe.class);
for (Field f : Enum.class.getDeclaredFields()) {
    f.setAccessible(true);
    f.set(clone, f.get(original));
}
</pre>

Apart from the reference to `ObjenesisStd`, this is all standard Java reflection code. So, what is this Objenesis thing, then? From their [website](http://objenesis.org):

> Objenesis is a small Java library that serves one purpose: To instantiate a new object of a particular class.

In other words, it can instantiate any object, without calling its constructor. So how does Objenesis work, exactly? Well, that depends. It uses the Strategy pattern to choose from several different ways of instantiating objects, depending on what kind of JVM you're running, and probably some other factors, too. In my case, it expanded to this:

<pre class="prettyprint">
Constructor&lt;Object> objectConstructor =
    Object.class.getConstructor((Class[]) null);
Class&lt;?> reflectionFactoryClass =
    Class.forName("sun.reflect.ReflectionFactory");
Method method = reflectionFactoryClass.getDeclaredMethod("getReflectionFactory");
Object reflectionFactory = method.invoke(null);
Method newConstructorForSerializationMethod =
    reflectionFactoryClass.getDeclaredMethod("newConstructorForSerialization", Class.class, Constructor.class);
Constructor&lt;CallMe> ctr = (Constructor&lt;CallMe>)
    newConstructorForSerializationMethod.invoke(reflectionFactory, CallMe.class, objectConstructor);
CallMe clone = ctr.newInstance((Object[]) null);
</pre>

That's some incredibly hairy scary code. Let's pretend we never saw this. The only thing we need to remember is that all this can be done without resorting to actually changing the bytecode at runtime; it's all reflection.

While Objenesis maybe a relatively unknown library, it is actually pretty widely used. The most famous libraries that use it are [Mockito](http://mockito.org) (and basically all mocking frameworks), and [Spring Framework](http://projects.spring.io/spring-framework/). But there are more. Many more. EqualsVerifier uses it to instantiate values for the fields of the class it's testing.

OK, now we that know this, can we go one further? It turns out we can:

![Call Me Sometime]({{ assets['CallMeSometime'] }})

We can add our own enum constants. Here's how:

<pre class="prettyprint">
CallMe sometime = new ObjenesisStd().newInstance(CallMe.class);
Field ordinal = Enum.class.getDeclaredField("ordinal");
ordinal.setAccessible(true);
ordinal.set(sometime, 4);
Field name = Enum.class.getDeclaredField("name");
name.setAccessible(true);
name.set(sometime, "SOMETIME");
</pre>

It's actually pretty simple. I guess the JVM's guarantees aren't ironclad enough for this particular sophisticated reflection attack. And to think I actually found it by accident! I fixed the bug in EqualsVerifier before it ever got released into production, so all's well that ends well, I guess.

P.S. Oh and don't try this at home kids! ...or at least, not in production.
