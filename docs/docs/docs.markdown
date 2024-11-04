---
layout: page
title: Documentation
permalink: /docs/
---

Reference documentation for Skippy version `{% include_relative version.markdown %}`.

New to Skippy? The best way to get started are the introductory tutorials:
- [Getting started with Skippy, Gradle & JUnit 5](https://www.skippy.io/tutorials/skippy-gradle-junit5)
- [Getting started with Skippy, Maven & JUnit 5](https://www.skippy.io/tutorials/skippy-maven-junit5)

## Table of Contents

* [Overview](#overview)
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
  * [Opt-Out Mechanism](#opt-out-mechanism)
* [Coverage for skipped tests](#coverage-for-skipped-tests)
  * [Gradle](#gradle-1)
  * [Maven](#maven-1)
* [Customize how Skippy reads and writes data](#customize-how-skippy-reads-and-writes-data)
  * [Gradle](#gradle-2)
  * [Maven](#maven-2)
* [Skippy in your CI pipeline](#skippy-in-your-ci-pipeline)
* [Roadmap](#roadmap)


## Overview

### What is it?

Skippy is a Test Impact Analysis & Predictive Test Selection framework for the JVM. It cuts down on unnecessary testing
and flakiness without compromising the integrity of your builds. You can run it from the command line, your favorite IDE
and continuous integration server. Skippy supports Gradle, Maven, JUnit 4 and JUnit 5.

Skippy is specifically designed to prevent regressions in your codebase.
It supports all types of tests where the tests and the code under test run in the same JVM.
It is best suited for deterministic tests, even those prone to occasional flakiness.
It provides the most value for test suites that are either slow or flaky (regardless of whether the test suite contains unit, integration, or functional tests).

### What is it not?

Skippy is not designed for tests that assert the overall health of a system. Don't use it for tests you want to fail
in response to misbehaving services, infrastructure issues, etc.

### Highlights

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

| Skippy   | JUnit 4       | JUnit 5      |
|----------|---------------|--------------|
| ≥ 0.0.18 | 4.10 or later | 5.0 or later |
| ≥ 0.0.8  | 4.10 or later | 5.9 or later |
| ≤ 0.0.7  | ❌            | 5.9 or later |

### Gradle / Maven

| Skippy   | Gradle                         | Maven          |
|----------|--------------------------------|----------------|
| ≥ 0.0.9  | 7.3 or later<br/> 8.0 or later | 3.2.3 or later |
| ≤ 0.0.8  | 7.3 or later<br/> 8.0 or later | ❌             |

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

```java
import io.skippy.junit5.PredictWithSkippy;

@PredictWithSkippy
public class FooTest {

    @Test
    void testFoo() {
        assertEquals("hello", Foo.hello());
    }
}
```

The annotation based approach allows you to enable predictive test selection for a sub-set of your tests.

#### Automatic Enablement

Automatic enablement is based on JUnit 5's [Automatic Extension Registration](https://junit.org/junit5/docs/current/user-guide/#extensions-registration-automatic).
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

```groovy
test {
  jvmArgs += "-Djunit.jupiter.extensions.autodetection.enabled=true"
}
```

Maven example:

```xml
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

### Opt-Out Mechanism

With [Automatic Enablement](#automatic-enablement), Skippy is enabled for all tests without source code changes. 
However, if there are specific tests you don’t want Skippy to optimize, use the `@AlwaysRun` annotation:

```java
import io.skippy.core.AlwaysRun;

@AlwaysRun
public class FooTest {

    @Test
    void testThatYouNeverWantToSkip() {
      // this test will always run
    }

}
```

Any test marked with `@AlwaysRun`, or inheriting from a superclass or interface annotated with it, will always run and bypass Skippy's skip-or-execute logic.

This is also useful for `@Nested` tests (even if you use [Annotation Based Enablement](#annotation-based-enablement)):

```java
@PredictWithSkippy
public class NestedTestsTest {

    @Nested
    class FooTest {
        
        @Test
        void testSomething() {
            // this test can be skipped by Skippy
        }
        
    }

    @AlwaysRun
    @Nested
    class BarTest {
        
        @Test
        void testSomething() {
            // this test will always run
        }
        
    }

}
```


#### Custom PredictionModifier

Skippy’s [PredictionModifier](https://github.com/skippy-io/skippy/blob/13403b0e700aba29887ca5759a880f90e1215f1b/skippy-core/src/main/java/io/skippy/core/PredictionModifier.java#L42)
extension controls this behavior, with the [DefaultPredictionModifier](https://github.com/skippy-io/skippy/blob/13403b0e700aba29887ca5759a880f90e1215f1b/skippy-core/src/main/java/io/skippy/core/DefaultPredictionModifier.java#L27)
automatically excluding `@AlwaysRun` tests. 

If you need custom logic, register a custom `PredictionModifier` in your build configuration. This gives you the flexibility to opt-out of Skippy based on  

- class name,
- package name or 
- any other information you can derive from a `Class<?>` object.

#### Gradle

Add the following to your `build.gradle` file to register a custom `PredictionModifier` in Gradle: 

```groovy
skippy {
  predictionModifier = 'com.example.MyPredictionModifier'
}

dependencies {
  testImplementation 'com.example:my-prediction-modifier:1.0.0'
}
```

#### Maven

Add the following to your `pom.xml` file to register a custom `PredictionModifier` in Maven:

```xml
  ...
  <dependencies>
    <!-- provide your extension to your tests -->
    <dependency>
      <groupId>com.example</groupId>
      <artifactId>my-prediction-modifier</artifactId>
      <version>1.0.0</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  ...
  <plugin>
    <groupId>io.skippy</groupId>
    <artifactId>skippy-maven</artifactId>
    <dependencies>
    <configuration>
      <!-- register your extension -->
      <predictionModifier>com.example.MyPredictionModifier</predictionModifier>
    </configuration>
  </plugin>
  ...
```

## Coverage for skipped tests

Skippy supports the creation of code coverage reports that include coverage for skipped tests. This feature can be
enabled using the setting `coverageForSkippedTests` which defaults to `false`.

When enabled, Skippy stores an execution data files for each test that is executed:
```
./gradlew test

LeftPadderTest > testPadLeft() PASSED
RightPadderTest > testPadRight() PASSED
```

The execution data files are stored in the .skippy folder:
```
ls -l .skippy                   

A4C4B20204923316E28BF16B20811F3C.exec
D16D8B65CE4D3DDCC1217644BA0C1DFF.exec
```
The `executionId` property in the test-impact-analysis.json file connects the dots between the execution of a test and
the corresponding execution data file:
```json
{
  "classes": {
    "0": {
      "name": "com.example.LeftPadderTest",
      ...
      "hash": "D6954047"
    },
    "1": {
      "name": "com.example.RightPadderTest",
      ...
    }
  },
  "tests": [
    {
      "class": 0,
      ...
      "executionId": "A4C4B20204923316E28BF16B20811F3C"
    },
    {
      "class": 1,
      ...
      "executionId": "D16D8B65CE4D3DDCC1217644BA0C1DFF"
    }
  ]
}
```

At the end a build, Skippy will generate a file named skippy.exec within the build directory. It is typically found 
under build/ for Gradle projects or target/ for Maven projects:

```
ls -l build

skippy.exec
```

skippy.exec aggregates the execution data for all tests that have been skipped in the current build. It can be merged
with JaCoCo’s standard execution data file to generate a combined report with coverage for executed and skipped tests:

- [jacoco:merge](https://www.eclemma.org/jacoco/trunk/doc/merge-mojo.html)
- [JacocoReport](https://docs.gradle.org/current/dsl/org.gradle.testing.jacoco.tasks.JacocoReport.html)

This feature is particularly useful for corporate CI pipelines with strict code coverage requirements.

### Gradle

To enable this feature in Gradle, add the following to your `build.gradle` file:

```groovy
skippy {
  coverageForSkippedTests = true
}
```

### Maven

To enable this feature in Maven, add the following to your `pom.xml` file:

```xml
  <plugin>
    <groupId>io.skippy</groupId>
    <artifactId>skippy-maven</artifactId>
    ...
    <configuration>
      <coverageForSkippedTests>true</coverageForSkippedTests>
    </configuration>
    ...
  </plugin>
```

## Customize how Skippy reads and writes data

Skippy's [SkippyRepositoryExtension](https://github.com/skippy-io/skippy/blob/55c194327f425705e8f8e5b46ef9dab78f60d337/skippy-core/src/main/java/io/skippy/core/SkippyRepositoryExtension.java#L40) 
allows projects to customize the way Skippy reads and writes data.
Skippy's default behavior:

- It stores and retrieves all data in / from the .skippy folder.
- It only retains the latest Test Impact Analysis.
- It only retains the JaCoCo execution data files that are referenced by the latest Test Impact Analysis.

The default settings are designed for small projects that do not require code coverage reports, and thus, do not store
JaCoCo execution data files. Those projects will typically disable the `coverageForSkippedTests` setting. While these
defaults support projects of any size and allow for the storage of JaCoCo data files for experimental purposes, they are
not recommended for large projects or those needing to store such files long-term. Doing so could significantly increase
the size of a project's history in a version control system.

Large projects aiming to store and permanently retain Test Impact Analysis instances and JaCoCo execution data files can
register a custom `SkippyRepositoryExtension`. This extension enables the storage of these artifacts outside the
project’s repository using systems such as databases, network file systems, or blob storage solutions like AWS S3.

Example for a custom `SkippyRepositoryExtension`: [FileSystemBackedRepositoryExtension](https://github.com/skippy-io/skippy/blob/55c194327f425705e8f8e5b46ef9dab78f60d337/skippy-extensions/skippy-repository-filesystem/src/main/java/io/skippy/extension/FileSystemBackedRepositoryExtension.java#L26)

### Gradle

To enable this feature in Gradle, add the following to your `build.gradle` file:

```groovy
buildscript {    
    dependencies {
        // provide your extension to Skippy's Gradle plugin
        classpath 'com.example:my-repository-extension:1.0.0'
    }
}

skippy {
    // register your extension
    repository = 'com.example.MyRepositoryExtension'
}

dependencies {
    // provide your extension to your tests
    testImplementation 'com.example:my-repository-extension:1.0.0'
}
```

Your extension must be provided as dependency to the build script and your tests.

### Maven

To enable this feature in Maven, add the following to your `pom.xml` file:

```xml
  ...
  <dependencies>
    <!-- provide your extension to your tests -->
    <dependency>
      <groupId>com.example</groupId>
      <artifactId>my-repository-extension</artifactId>
      <version>1.0.0</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  ...
  <plugin>
    <groupId>io.skippy</groupId>
    <artifactId>skippy-maven</artifactId>
    <dependencies>
      <!-- provide your extension to Skippy's Maven plugin -->
      <dependency>
        <groupId>com.example</groupId>
        <artifactId>my-repository-extension</artifactId>
        <version>1.0.0</version>
      </dependency>
    </dependencies>
    <configuration>
      <!-- register your extension -->
      <repository>com.example.MyRepositoryExtension</repository>
    </configuration>
  </plugin>
  ...
```

Your extension must be provided as dependency to Skippy's Maven plugin and your tests.

## Skippy in your CI pipeline

Skippy supports execution in CI environments out of the box. It has actually been designed for this purpose. 
The only thing you need to do is to add the .skippy folder to version control. This will automatically enable
Skippy's Predictive Test Selection when your pipeline runs.

## Roadmap

- Support for reasoning based on changes in the classpath (e.g., updated 3rd party dependencies)
- Support for reasoning based on changes in resources (e.g., a change in `application.properties`)

{% include_relative comments.markdown %}
