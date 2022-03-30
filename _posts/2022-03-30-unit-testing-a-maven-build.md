---
title: "Unit testing a Maven build"
tags:
- equalsverifier
- java
- maven
- unit-testing
excerpt: In which I test the artifacts produced by a Maven build.
---
A few weeks ago, I asked a [question on StackOverflow](https://stackoverflow.com/q/70861878/127863): how can you test a Maven build?

EqualsVerifier a complex Maven build, and I want to have some automated checks that the jar files and pom files produced by the build actually contain what I expect them to contain.

For instance, I want to check for the presence of an Automatic-Module-Name entry in the manifest file. It's a multi-release jar, so I also want to check that the proper class files exist inside of the jar file's `META-INF/versions` directory. Finally, EqualsVerifier is published to Maven Central, so I also want to check that the produced pom file contains the dependencies the project needs, but that the produced pom file for the fat jar that I also publish, doesn't contain these dependencies.

Unfortunately, it's hard to google for this, because of the words I would use to describe this ("test", "verify") already have specific different meanings in Maven.

I got a nice response from [Karl Heinz Marbaise](https://twitter.com/khmarbaise), one of the Maven devs, who suggested I create an additional Maven submodule, use a plugin to copy the relevant artifacts to a directory, and go from there.

So I created the `equalsverifier-release-verify` submodule in the project and used `copy-rename-maven-plugin` to copy the files, as follows:

{% highlight xml %}
<plugin>
    <groupId>com.coderplus.maven.plugins</groupId>
    <artifactId>copy-rename-maven-plugin</artifactId>
    <version>${version.copy-rename-maven-plugin}</version>
    <executions>
        <execution>
            <id>copy-artifacts</id>
            <phase>compile</phase>
            <goals>
                <goal>copy</goal>
            </goals>
            <configuration>
                <fileSets>
                    <fileSet>
                        <sourceFile>${artifact.src.main}/.flattened-pom.xml</sourceFile>
                        <destinationFile>${artifact.dst}/equalsverifier-main.pom</destinationFile>
                    </fileSet>
                    <fileSet>
                        <sourceFile>${artifact.src.main}/equalsverifier-${project.version}.jar</sourceFile>
                        <destinationFile>${artifact.dst}/equalsverifier-main.jar</destinationFile>
                    </fileSet>
                    <!-- more fileSets here -->
                </fileSets>
            </configuration>
        </execution>
    </executions>
</plugin>
{% endhighlight %}

Now the poms and jars are in the `src/test/resources` folder, I can work with them. For the pom files, I used Java's built-in [XPath](https://docs.oracle.com/en/java/javase/17/docs/api/java.xml/javax/xml/xpath/XPath.html) API, because it's simple, and my needs are simple as well. But you can use whatever you want.

For the jar files, I used NIO to access their content as a [FileSystem](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/nio/file/FileSystem.html):

{% highlight java %}
var filename = "myArtifactId.jar"; // or "flattened.pom"
var file = getClass().getClassLoader().getResource(filename);
var uri = URI.create("jar:" + file.toURI().toString());
FileSystem fs = FileSystems.newFileSystem(uri, Map.of());
{% endhighlight %}

Now I can get a list of files that exist in the jar:

{% highlight java %}
var path = fs.getPath("/");
Set<String> filenames = StreamSupport
    .stream(walk.spliterator(), false)
    .map(Path::toString)
    .collect(Collectors.toSet());
{% endhighlight %}

Or read the content of a file to check the content of the manifest:

{% highlight java %}
var path = fs.getPath("/META-INF/MANIFEST.MF");
var out = new ByteArrayOutputStream();
Files.copy(path, out);
String content = out.toString();
assertTrue(content.contains("Automatic-Module-Name: nl.jqno.equalsverifier"));
{% endhighlight %}

I can even check if files are compiled to the correct Java version:

{% highlight java %}
var path = fs.getPath("/com/example/MyClass.class");
var out = new ByteArrayOutputStream();
Files.copy(path, out);
byte[] content = out.toByteArray();
var actualVersion = content[7]; // the major version of the class file is at this location
assertEquals(52, actualVersion); // 52 = Java 8
{% endhighlight %}

([Here](https://en.wikipedia.org/wiki/Java_class_file#General_layout) is a description of Java's class file format, including a list of Java major class file versions.)

Note that for this post, I didn't bother handling exceptions or closing resources. Filling in those blanks is left as an exercise for you, dear reader 😉.

This was a pretty fun thing to play around with! If you want to see the full code, take a look at the [`equalsverifier-release-verify` submodule on GitHub](https://github.com/jqno/equalsverifier/tree/main/equalsverifier-release-verify)!
