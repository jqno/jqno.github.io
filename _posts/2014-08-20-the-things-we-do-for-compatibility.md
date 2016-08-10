---
title: "The things we do for compatibility"
tags:
- compatibility
- java
- java8
- equalsverifier
---
For EqualsVerifier's new 1.5 release, I faced a dilemma. EqualsVerifier should support Java 8, but it also should still run under Java 6 and 7. Preferably in a single code base, because maintaining multiple code bases is a hassle (even if it means I get to use lambdas in one of them). Also, there should be unit tests targeting Java 8-specific classes: does EqualsVerifier support classes that contain lambdas? does EqualsVerifier support classes with fields of type, say, `java.time.ZonedDateTime`? These tests should run on Java 8 but should not break on Java 6. Can this even be done?

The answer is: yes. Yes, it can be done. Here's how.


## Compiling

It turns out that Java 6 introduced the `javax.tools.JavaCompiler` interface. You can use it, usurprisingly, to compile Java classes at runtime, like so:

<pre class="prettyprint">
private void compileClass(File sourceFile, File tempFolder) throws IOException {
    JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
    StandardJavaFileManager fileManager = null;
    try {
        fileManager = compiler.getStandardFileManager(null, null, null);
        fileManager.setLocation(StandardLocation.CLASS_OUTPUT, Arrays.asList(tempFolder));
        Iterable&lt;? extends JavaFileObject> javaFileObjects = fileManager.getJavaFileObjectsFromFiles(Arrays.asList(sourceFile));
        CompilationTask task = compiler.getTask(null, fileManager, null, null, null, javaFileObjects);

        boolean success = task.call();
        if (!success) {
            throw new AssertionError("Could not compile the class");
        }
    }
    finally {
        if (fileManager != null) {
            fileManager.close();
        }
    }
}
</pre>

Note that we can't use try-with-resources because of EqualsVerifier's Java 6 compatibility requirement.

This code assumes that `sourceFile` is a `File` reference to a Java source file. It will compile the file and write it to the `tempFolder`. The filename will be identical to the source file, but with a `.class` extension instead of a `.java` extension.

Also, any compile errors are written to the console. Not ideal, but I haven't tried yet to redirect them so I can show the in the `AssertionError` somehow. I might do that for a future revision though.


## Loading

So, now we have a `.class` file somewhere on our filesystem. However, it's not on the classpath yet, the JVM doesn't automagically load it, and we don't have a `Class<?>` variable referencing it. So how do we use it? This is where `java.net.URLClassLoader` comes in:

<pre class="prettyprint">
private URLClassLoader createClassLoader(File tempFolder) {
    try {
        URL[] urls = { tempFolder.toURI().toURL() };
        return new URLClassLoader(urls);
    }
    catch (MalformedURLException e) {
        throw new AssertionError(e);
    }
}
</pre>

Note that, as of Java 7, the `URLClassLoader` implements `Closeable`, which means it has a `close()` method that needs to be called when we're done. It's not `Closeable` yet in Java 6, so we'll have to call `close()` using reflection. I'll leave it as an exercise to you, my esteemed reader, to figure out how to do that.

The important thing is: now we have a class loader that we can use to load our class and pass it to EqualsVerifier:

<pre class="prettyprint">
Class&lt;?> type = createClassLoader(tempFolder);
EqualsVerifier.forClass(type).verify();
</pre>

Note that this code adds the entire contents of the `tempFolder` file to the classpath, so it's wise to create a fresh, empty directory for this. Since I use this code only in unit tests, I use JUnit's `TemporaryFolder` rule to manage this.


## Tying it together

The unit test simply contains a raw `String`:

<pre class="prettyprint">
private static final String JAVA_8_CLASS =
    "\nimport java.util.List;" +
    "\nimport java.util.Objects;" +
    "\n" +
    "\npublic final class Java8Class {" +
    "\n    private final List&lt;Object> objects;" +
    "\n    " +
    "\n    public Java8Class(List&lt;Object> objects) {" +
    "\n        this.objects = objects;" +
    "\n    }" +
    "\n    " +
    "\n    public void doSomethingWithStreams() {" +
    "\n        objects.stream().forEach(System.out::println);" +
    "\n    }" +
    "\n    " +
    "\n    @Override" +
    "\n    public boolean equals(Object obj) {" +
    "\n        if (!(obj instanceof Java8Class)) {" +
    "\n            return false;" +
    "\n        }" +
    "\n        return objects == ((Java8Class)obj).objects;" +
    "\n    }" +
    "\n    " +
    "\n    @Override" +
    "\n    public int hashCode() {" +
    "\n        return Objects.hash(objects);" +
    "\n    }" +
    "\n}";
</pre>

We can write this `String` to a `java.io.File` (in the same `tempFolder` directory mentioned above), making sure that it's name is the name of the class with a `.java` extension. In this case, that would be `Java8Class.java`. Then we pass the `File` reference to the `compileClass` method defined above, and the circle is complete.


## Java 6

Now what about Java 6? We haven't used any API calls that aren't available in Java 6, so that's good, but obviously the `Java8Class` string won't compile. We don't want our test to fail on that. We can solve this by simply detecting if the test is running on a Java 8 JVM, and if it's not, simply return. How we do this? Well...

<pre class="prettyprint">
public boolean isTypeAvailable(String fullyQualifiedTypeName) {
    try {
        Class.forName(fullyQualifiedClassName);
        return true;
    }
    catch (ClassNotFoundException e) {
        return false;
    }
}

// ...

if (!isTypeAvailable("java.util.Optional")) {
    return;
}
</pre>

`java.util.Optional` was introduced in Java 8, so if it's on the classpath, we know we're running Java 8 (or higher). It's a bit of a hack, I know, but to me it felt more reliable than checking Java system properties. And the whole thing is obviously a huge hack anyway, so what's one more, right? :)


## Java 8 API classes

So that takes care of classes containing lambdas, streams, and other Java 8 language features. But we're not done yet, because what about classes containing fields of a type that wat introduced in the Java 8 API? For example, Java 8 introduced the new Java Time API, and some other new classes as well (such as `Optional` which we abused above). Some of these are defined recursively, meaning EqualsVerifier can't instantiate them without a little help, so we need to find a way to instantiate these classes and add them to EqualsVerifier's prefabValues.

We know in advance which classes we need to add to EqualsVerifier's prefab values (we can simply try them out and make an inventory list), and we also know how to instantiate them (that's part of the API, after all). we just can't call the constructor directly, because the class may or may not be on the classpath, depending on the JVM version currently running. Reflection to the rescue!

It turns out there are 3 main ways an instance of a class can be retrieved: through calling its constructor, through calling a static factory method defined on the same class, or through referencing a static constant defined on the class. Since reflection is even more verbose than vanilla Java, I've hidden all this away in a nice helper class that allows me to do things like this:

<pre class="prettyprint">
ConditionalPrefabValueBuilder.of("java.lang.Integer")
        .callFactory("valueOf", classes(int.class), objects(42))
        .callFactory("valueOf", classes(int.class), objects(1337))
        .addTo(prefabValues);
</pre>

The `ConditionalPrefabValueBuilder` contains similar methods for calling constructors or referencing constants. Behind the curtains, it calls things like `Class.forName()`, `Constructor.newInstance()` and `Method.invoke()`. It contains a lot of `try/catch` blocks, too. The `classes` and `objects` methods are static imports for methods that I wrote that convert a vararg into an array. They just look a lot nicer than `new Class<?>[] { int.class }` would.


## Joda-Time and Google Guava

While I was at it, I also added prefab values for some commonly used classes from Joda-Time and Google Guava, such as `LocalTime` and `ImmutableList`. Because, why not?

(Please note that I'm not going to add prefab values for every library out there. But since Joda-Time and Guava are so ubiquitous, I think this has real added value.)


## Unit tests

In order to test all this, we can simply write a class containing some of these types, put it in a string, and run it through the compiler, much like we did with the Java 8 class above.

However, this changes one thing quite dramatically. Now that the tests are platform-dependent, they need to be run on each platform before I can release it. After all, if I run the tests only on Java 7, how will I know that I called `ConditionalPrefabValueBuilder` correctly with my Java 8 `java.time.ZonedDateTime`? I don't, that's how.

But, [TravisCI](http://travis-ci.org) to the rescue! TravisCI is a continuous integration service which is free for open source projects such as EqualsVerifier. I have configured it in such a way that, whenever I push something to GitHub, it triggers a build on OpenJDK 6, OpenJDK 7, Oracle JDK 7, and Oracle JDK 8. Whenever something fails, I'll receive an e-mail within mere minutes. It's a life-saver.


## Conclusion

If you want to take a look at the full source: it's all on [GitHub](https://github.com/jqno/equalsverifier). Here are some of the classes I discussed: [ConditionalCompiler](https://github.com/jqno/equalsverifier/blob/79229ca2a072ee9c985ff8f01670e68625feabf7/test/nl/jqno/equalsverifier/testhelpers/ConditionalCompiler.java), [Java8ClassTest](https://github.com/jqno/equalsverifier/blob/79229ca2a072ee9c985ff8f01670e68625feabf7/test/nl/jqno/equalsverifier/integration/operational/Java8ClassTest.java), [ConditionalPrefabValueBuilder]() and [ConditionalInstantiator](https://github.com/jqno/equalsverifier/blob/79229ca2a072ee9c985ff8f01670e68625feabf7/src/nl/jqno/equalsverifier/util/ConditionalInstantiator.java). Fork away!

So there you have it: quite possibly the biggest hack in my career so far. I'm still not sure whether to be proud or ashamed. But I've been working with this for several weeks now, and it works quite well! And it certainly adds value to EqualsVerifier: for me, because I don't need to maintain separate code bases for different versions of Java. And for you, the user, because you don't need to worry about which version of EqualsVerifier to use with your version of Java, and because you even get prefab values for Joda-Time and Google Guava as an added bonus.

