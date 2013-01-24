## Getting Binaries

Binaries and dependency information for Maven, Ivy, Gradle and others can be found at [http://search.maven.org](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.netflix.rx%22%20AND%20a%3A%22rxjava-core%22).

Example for Maven:

```xml
<dependency>
    <groupId>com.netflix.rx</groupId>
    <artifactId>rxjava-core</artifactId>
    <version>0.5.0</version>
</dependency>
```
and for Ivy:

```xml
<dependency org="com.netflix.rx" name="rxjava-core" rev="0.5.0" />
```

If you need to download the jars instead of using a build system, create a Maven pom file like this with the desired version:

```xml
<?xml version="1.0"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.netflix.rx.download</groupId>
	<artifactId>rxjava-download</artifactId>
	<version>1.0-SNAPSHOT</version>
	<name>Simple POM to download rxjava-core and dependencies</name>
	<url>http://github.com/Netflix/RxJava</url>
	<dependencies>
		<dependency>
			<groupId>com.netflix.rx</groupId>
			<artifactId>rxjava-core</artifactId>
			<version>0.5.0</version>
			<scope/>
		</dependency>
	</dependencies>
</project>
```

Then execute:

```
mvn -f download-rxjava-pom.xml dependency:copy-dependencies
```

It will download rxjava-core-*.jar and its dependencies into ./target/dependency/.

You need Java 6 or later.

## Hello World!

The simplest use of RxJava is as follows:

```java
public class CommandHelloWorld extends HystrixCommand<String> {

    private final String name;

    public CommandHelloWorld(String name) {
        super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
        this.name = name;
    }

    @Override
    protected String run() {
        return "Hello " + name + "!";
    }
}
```
[View Source](../blob/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples/basic/CommandHelloWorld.java)

This command could be used like this:

```java
String s = new CommandHelloWorld("Bob").execute();
Future<String> s = new CommandHelloWorld("Bob").queue();
```

More examples and information can be found in the [[How To Use]] section.

Example source code can be found in the [hystrix-examples](../tree/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples) module.

## Building

To checkout the source and build:

```
$ git clone git@github.com:Netflix/RxJava.git
$ cd RxJava/
$ ./gradlew build
```

To do a clean build:

```
$ ./gradlew clean build
```

A build should look similar to this:

```
$ ./gradlew build
:rxjava-core:compileJava
:rxjava-core:processResources UP-TO-DATE
:rxjava-core:classes
:rxjava-core:jar
:rxjava-core:sourcesJar
:rxjava-core:signArchives SKIPPED
:rxjava-core:assemble
:rxjava-core:licenseMain UP-TO-DATE
:rxjava-core:licenseTest UP-TO-DATE
:rxjava-core:compileTestJava
:rxjava-core:processTestResources UP-TO-DATE
:rxjava-core:testClasses
:rxjava-core:test
:rxjava-core:check
:rxjava-core:build

BUILD SUCCESSFUL

Total time: 30.758 secs
```

On a clean build you will see the unit tests be run and look something like this:

```
> Building > :rxjava-core:test > 91 tests completed
```