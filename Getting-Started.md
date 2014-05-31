## Getting Binaries

You can find binaries and dependency information for Maven, Ivy, Gradle, SBT, and others at [http://search.maven.org](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.netflix.rxjava%22%20AND%20a%3A%22rxjava-core%22).

Example for Maven:

```xml
<dependency>
    <groupId>com.netflix.rxjava</groupId>
    <artifactId>rxjava-core</artifactId>
    <version>0.17.0</version>
</dependency>
```
and for Ivy:

```xml
<dependency org="com.netflix.rxjava" name="rxjava-core" rev="0.17.0" />
```

and for SBT:

```scala
libraryDependencies += "com.netflix.rxjava" % "rxjava-scala" % "0.17.0"
```

If you need to download the jars instead of using a build system, create a Maven `pom` file like this with the desired version:

```xml
<?xml version="1.0"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.netflix.rxjava.download</groupId>
	<artifactId>rxjava-download</artifactId>
	<version>1.0-SNAPSHOT</version>
	<name>Simple POM to download rxjava-core and dependencies</name>
	<url>http://github.com/Netflix/RxJava</url>
	<dependencies>
		<dependency>
			<groupId>com.netflix.rxjava</groupId>
			<artifactId>rxjava-core</artifactId>
			<version>0.17.0</version>
			<scope/>
		</dependency>
	</dependencies>
</project>
```

Then execute:

```
$ mvn -f download-rxjava-pom.xml dependency:copy-dependencies
```

That command downloads `rxjava-core-*.jar` and its dependencies into `./target/dependency/`.

You need Java 6 or later.

## Building

To check out and build the RxJava source, issue the following commands:

```
$ git clone git@github.com:Netflix/RxJava.git
$ cd RxJava/
$ ./gradlew build
```

To do a clean build, issue the following command:

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

On a clean build you will see the unit tests run. They will look something like this:

```
> Building > :rxjava-core:test > 91 tests completed
```

#### Troubleshooting

One developer reported getting the following error:

> Could not resolve all dependencies for configuration ':language-adaptors:rxjava-scala:provided'

He was able to resolve the problem by removing old versions of `scala-library` from `.gradle/caches` and `.m2/repository/org/scala-lang/` and then doing a clean build. <a href="https://gist.github.com/jaceklaskowski/9496058">(See this page for details.)</a>