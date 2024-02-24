---
layout: page
title: Documentation
permalink: /docs/
---

Reference documentation for Skippy version `{% include_relative version.markdown %}`.

New to Skippy? The best way to get started are the introductory tutorials:
- [Getting Started with Skippy, Gradle & JUnit 5](https://www.skippy.io/tutorials/skippy-gradle-junit5)
- [Getting Started with Skippy, Maven & JUnit 5](https://www.skippy.io/tutorials/skippy-maven-junit5)

## Table of Contents

* [What is it?](#what-is-it)
* [What is it not?](#what-is-it-not)
* [Highlights](#highlights)
* [Compatibility](#compatibility)
* [Skippy's Test Impact Analysis](#skippys-test-impact-analysis)
  * [Gradle](#gradle)
  * [Maven](#maven)
* [Skippy's Predictive Test Selection](#skippys-predictive-test-selection)
  * [JUnit 4](#junit-4)
  * [JUnit 5](#junit-5)
* [Use Skippy In Your CI Pipeline](#use-skippy-in-your-ci-pipeline)
* [Roadmap](#roadmap)


## What is it?

Skippy is a Test Impact Analysis & Predictive Test Selection framework for the JVM. It cuts down on unnecessary testing
and flakiness without compromising the integrity of your builds. You can run it from the command line, your favorite IDE
and continuous integration server. Skippy supports Gradle, Maven, JUnit 4 and JUnit 5.

Skippy is specifically designed to prevent regressions in your codebase.
It supports all types of tests where the tests and the code under test run in the same JVM.
It is best suited for deterministic tests, even those prone to occasional flakiness.
It provides the most value for test suites that are either slow or flaky (regardless of whether the test suite contains unit, integration, or functional tests).

## What is it not?

Skippy is not designed for tests that assert the overall health of a system. Don't use it for tests you want to fail
in response to misbehaving services, infrastructure issues, etc.

## Highlights

- Support for Gradle & Maven
- Support for JUnit 4 & JUnit 5
- Lightweight: Use it from the command line, your favorite IDE and CI server
- Non-invasive: Use it for a single test, your entire suite and anything in-between
- Free of lock-in: You can go back to a "run everything" approach at any time
- Open Source under Apache 2 License

## Compatibility

Compatibility matrix for Skippys’s integration with [Java](https://openjdk.org/), [JUnit](https://junit.org/), 
[Gradle](https://gradle.org/), [Maven](https://maven.apache.org/)  and [JaCoCo](https://www.jacoco.org/). Versions not 
mentioned here might work, but there is no guarantee.

### Java

| Skippy       | Java         |
|--------------|--------------|
| all versions | 17 or later  |

### JUnit 4 / JUnit 5

| Skippy  | JUnit 4       | JUnit 5      |
|---------|---------------|--------------|
| ≥ 0.0.13 | 4.10 or later | 5.0 or later |
| ≥ 0.0.8 | 4.10 or later | 5.9 or later |
| ≤ 0.0.7 | ❌            | 5.9 or later |

### Gradle / Maven

| Skippy  | Gradle                         | Maven        |
|---------|--------------------------------|--------------|
| ≥ 0.0.9 | 7.3 or later<br/> 8.0 or later | 3.9 or later |
| ≤ 0.0.8 | 7.3 or later<br/> 8.0 or later | ❌           |

### JaCoCo

| Skippy   | JaCoCo           |
|----------|------------------|
| ≥ 0.0.9  | 0.8.7 or greater |
| ≤ 0.0.8  | 0.8.9            |

## Skippy's Test Impact Analysis

The following section describe Skippy's plugins for [Gradle](https://gradle.org/) and 
[Maven](https://maven.apache.org/). The build plugins implement Skippy's Test Impact Analysis.

### Gradle

The [skippy-gradle](https://github.com/skippy-io/skippy/tree/main/skippy-gradle) sub-project contains the Skippy
plugin for Gradle.

#### Install

Release versions are available from Gradle's Plugin Portal using the
[plugins DSL](https://docs.gradle.org/current/userguide/plugins.html#sec:plugins_block):

```groovy
plugins {
    id("io.skippy") version '{% include_relative version.markdown %}'
}
```

Release versions are also available from Maven Central using the
[legacy plugin application](https://docs.gradle.org/current/userguide/plugins.html#sec:old_plugin_application):
```groovy
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'io.skippy:skippy-gradle:{% include_relative version.markdown %}'
    }
}

apply plugin: io.skippy.gradle.SkippyPlugin
```

Snapshots are available from `s01.oss.sonatype.org` using the
[legacy plugin application](https://docs.gradle.org/current/userguide/plugins.html#sec:old_plugin_application):

```groovy
buildscript {
    repositories {
        mavenCentral()
        maven { url = 'https://s01.oss.sonatype.org/content/repositories/snapshots/' }
    }
    dependencies {
        classpath 'io.skippy:skippy-gradle:{% include_relative snapshotversion.markdown %}'
    }
}

apply plugin: io.skippy.gradle.SkippyPlugin
```

#### Tasks

The plugin adds the `skippyClean` and `skippyAnalyze` tasks to your project:
```
./gradlew tasks --group=skippy
```

Output:
```
Skippy tasks
------------
skippyClean
skippyAnalyze
```

#### skippyAnalyze Task

`skippyAnalyze` performs a Test Impact Analysis that stores a bunch of files in the .skippy folder.
The generated files are consumed by Skippy's [testing libraries](#skippys-predictive-test-selection) 
to make skip-or-execute predictions. `skippyAnalyze` is not meant to be invoked directly: It runs 
automatically whenever a Test task is executed.

#### skippyClean Task

`skippyClean` empties the skippy directory:

```
./gradlew skippyClean
```

It is the Gradle counterpart to `rm -rf .skippy`.

### Maven

The [skippy-maven](https://github.com/skippy-io/skippy/tree/main/skippy-maven) sub-project contains the Skippy
plugin for Maven.

#### Install

Release versions are available from Maven Central:

```xml
<project>
  ...
  <build>
    <plugins>
      <plugin>
        <groupId>io.skippy</groupId>
        <artifactId>skippy-maven</artifactId>
        <version>{% include_relative version.markdown %}</version>
        <executions>
          <execution>
            <goals>
              <goal>buildFinished</goal>
              <goal>buildStarted</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
  ...
</project>
```

Snapshots are available from `s01.oss.sonatype.org`:

```xml
<project>
  ...
  <pluginRepositories>
    <pluginRepository>
      <id>oss.sonatype</id>
      <url>https://s01.oss.sonatype.org/content/repositories/snapshots/</url>
    </pluginRepository>
  </pluginRepositories>
  ...
  <build>
    <plugins>
      <plugin>
        <groupId>io.skippy</groupId>
        <artifactId>skippy-maven</artifactId>
        <version>{% include_relative snapshotversion.markdown %}</version>
        <executions>
          <execution>
            <goals>
              <goal>buildFinished</goal>
              <goal>buildStarted</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
  ...
</project>
```

Since Skippy internally depends on JaCoCo, the JaCoCo plugin has to be added as well:

```xml
<project>
  ...
  <build>
    <plugins>
      ...
      <plugin>
        <groupId>org.jacoco</groupId>
        <artifactId>jacoco-maven-plugin</artifactId>
        <version>0.8.11</version>
        <executions>
          <execution>
            <goals>
              <goal>prepare-agent</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
  ...
</project>
```

#### Goals

The plugin adds the 
- `skippy:buildStarted`,
- `skippy:buildFinished` and 
- `skippy:clean`
goals to your project.

  
#### skippy:buildStarted & skippy:buildFinished

`skippy:buildStarted` and `skippy:buildFinished` perform a Test Impact Analysis that stores a bunch of files 
in the .skippy folder. The generated files are consumed by Skippy's [testing libraries](#skippys-predictive-test-selection)
to make skip-or-execute predictions. Both goals are not meant to be invoked directly: They run automatically 
when you execute your tests.

#### skippy:clean

`skippy:clean` empties the skippy directory:

```
mvn skippy:clean
```

It is the Maven counterpart to `rm -rf .skippy`.

## Skippy's Predictive Test Selection

Skippy's JUnit libraries implement Skippy's Predictive Test Selection algorithm. The algorithm makes skip-or-execute
predictions based on
- the local state of the project and
- the impact data in the .skippy folder that was generated by Skippy's Test Impact Analysis.

### JUnit 4

The [skippy-junit4](https://github.com/skippy-io/skippy/tree/main/skippy-junit4) sub-project contains the Skippy
library for JUnit 4.

#### Install

Releases are available in Maven Central.

Gradle:

```groovy
repositories {
    mavenCentral()
}

dependencies {
    testImplementation 'io.skippy:skippy-junit4:{% include_relative version.markdown %}'
}
```

Maven:

```xml
<dependency>
    <groupId>io.skippy</groupId>
    <artifactId>skippy-junit4</artifactId>
    <version>{% include_relative version.markdown %}</version>
    <scope>test</scope>
</dependency>
```

Snapshots are available in `s01.oss.sonatype.org`.

Gradle:

```groovy
repositories {
    maven { url = 'https://s01.oss.sonatype.org/content/repositories/snapshots/' }
}

dependencies {
    testImplementation 'io.skippy:skippy-junit4:{% include_relative snapshotversion.markdown %}'
}
```

Maven:

```xml
<project>
  
  <repositories>
    <repository>
      <id>s01-oss-sonatype-org</id>
      <url>https://s01.oss.sonatype.org/content/repositories/snapshots/</url>
    </repository>
  </repositories>
  
  <dependencies>
    <dependency>
        <groupId>io.skippy</groupId>
        <artifactId>skippy-junit4</artifactId>
        <version>{% include_relative snapshotversion.markdown %}</version>
        <scope>test</scope>
    </dependency>
  </dependencies>
  
</project>
```

#### Enable Predictive Test Selection

Add the Skippy class rule to your test:
```groovy
import io.skippy.junit4.Skippy;

public class FooTest {

    @ClassRule
    public static TestRule skippyRule = Skippy.predictWithSkippy();

    @Test
    public void testFoo() {
        ...
    }

}
```

#### Run Your Tests

The Skippy class rule contains Skippy's Predictive Test Selection algorithm. The algorithm makes skip-or-execute
predictions based on
- the state of the project and 
- the impact data in the .skippy folder that was generated by Skippy's Test Impact Analysis.

### JUnit 5

The [skippy-junit5](https://github.com/skippy-io/skippy/tree/main/skippy-junit5) sub-project contains the Skippy
library for JUnit 5.

#### Install

Releases are available in Maven Central.

Gradle:

```groovy
repositories {
    mavenCentral()
}

dependencies {
    testImplementation 'io.skippy:skippy-junit5:{% include_relative version.markdown %}'
}
```

Maven:

```xml
<dependency>
    <groupId>io.skippy</groupId>
    <artifactId>skippy-junit5</artifactId>
    <version>{% include_relative version.markdown %}</version>
    <scope>test</scope>
</dependency>
```

Snapshots are available in `s01.oss.sonatype.org`.

Gradle:

```groovy
repositories {
    maven { url = 'https://s01.oss.sonatype.org/content/repositories/snapshots/' }
}

dependencies {
    testImplementation 'io.skippy:skippy-junit5:{% include_relative snapshotversion.markdown %}'
}
```

Maven:

```xml
<project>
  
  <repositories>
    <repository>
      <id>s01-oss-sonatype-org</id>
      <url>https://s01.oss.sonatype.org/content/repositories/snapshots/</url>
    </repository>
  </repositories>
  
  <dependencies>
    <dependency>
        <groupId>io.skippy</groupId>
        <artifactId>skippy-junit5</artifactId>
        <version>{% include_relative snapshotversion.markdown %}</version>
        <scope>test</scope>
    </dependency>
  </dependencies>
  
</project>
```

#### Enable Predictive Test Selection

You have two options to enable predictive test selection for you JUnit 5 tests:
- Annotation Based Enablement
- Automatic Enablement

#### Annotation Based Enablement

Annotate your tests with `@PredictWithSkippy` to enable predictive test selection:

```groovy
import io.skippy.junit5.PredictWithSkippy;

@PredictWithSkippy
public class FooTest {

    @Test
    void testFoo() {
        assertEquals("hello", Foo.hello());
    }
}
```

The annotation based approach allows you to pick-and-chose the tests you want to enable predictive test selection for.

#### Automatic Enablement

Automatic enablement of predictive test selection is based on JUnit 5's [Automatic Extension Registration](https://junit.org/junit5/docs/current/user-guide/#extensions-registration-automatic).
Add a file named `org.junit.jupiter.api.extension.Extension` in
`src/test/resources/META-INF/services` (adjust according to the location of your test resources). The file must have
the following content:
```
io.skippy.junit5.SkipOrExecuteCondition
io.skippy.junit5.CoverageFileCallbacks
```

Start your tests with the JVM argument `-Djunit.jupiter.extensions.autodetection.enabled=true` to automatically
enable predictive test selection for all tests.

Gradle example:

```
test {
  jvmArgs += "-Djunit.jupiter.extensions.autodetection.enabled=true"
}
```

Maven example:

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration>
        <systemPropertyVariables>
            <junit.jupiter.extensions.autodetection.enabled>true</junit.jupiter.extensions.autodetection.enabled>
        </systemPropertyVariables>
    </configuration>
</plugin>
```

#### Run Your Tests

The Skippy extension contains Skippy's Predictive Test Selection algorithm. The algorithm makes skip-or-execute
predictions based on
- the local state of the project and
- the impact data in the .skippy folder that was generated by Skippy's Test Impact Analysis.

## Use Skippy In Your CI Pipeline

It is safe to add the .skippy folder to version control. This will automatically enable Skippy's Predictive Test
Selection when your pipeline runs.

## Roadmap

- Support to generate accurate coverage reports despite skipped tests (to support CI pipelines with strict coverage requirements)
- Support for reasoning based on changes in the classpath
- Support for reasoning based on changes in resources (e.g., a change in `application.properties`)

{% include_relative comments.markdown %}
