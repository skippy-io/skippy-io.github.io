---
layout: page
title: Documentation
permalink: /docs/
---

Skippy is designed for the modern pace of software development, where Continuous Integration (CI) should be a boon, not
a burden. Unlike traditional CI tools that run all tests regardless of necessity, Skippy's intelligent test impact
analysis cuts down on unnecessary testing and flakiness, slashing build times to improve developer productivity and
happiness.

Harnessing both dynamic and static bytecode analysis, Skippy lets you iterate faster without compromising the integrity
of your builds. It’s about working smarter, not harder—let Skippy streamline your builds, so you can get back to what CI
was meant to be: a fast track to production.

## Highlights

- Support for Gradle & Maven
- Support for JUnit 4 & JUnit 5
- Lightweight: Use it from the command line, your favorite IDE and CI server
- Non-invasive: Use it for a single test, your entire suite and anything in-between
- Open Source under Apache 2 License

## Table of Contents


* [Compatibility](#compatibility)
* [Skippy's Test Impact Analysis](#skippys-test-impact-analysis)
  * [Gradle](#gradle)
  * [Maven](#maven)
* [Skippy's Predictive Test Selection](#skippys-predictive-test-selection)
  * [JUnit 4](#junit-4)
  * [JUnit 5](#junit-5)
* [Roadmap](#roadmap)

## Compatibility

Compatibility matrix for Skippys’s integration with [Java](https://openjdk.org/), [JUnit](https://junit.org/), 
[Gradle](https://gradle.org/), [Maven](https://maven.apache.org/)  and [JaCoCo](https://www.jacoco.org/). Versions not 
mentioned here might work, but there is no guarantee.

### Java

| Skippy       | Java         |
|--------------|--------------|
| all versions | 17 or later  |

### JUnit 4 / JUnit 5

| Skippy  | JUnit 4      | JUnit 5      |
|---------|--------------|--------------|
| ≥ 0.0.8 | 4.9 or later | 5.9 or later |
| ≤ 0.0.7 | ❌           | 5.9 or later |

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
    id("io.skippy") version "0.0.11"
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
        classpath 'io.skippy:skippy-gradle:0.0.11'
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
        classpath 'io.skippy:skippy-gradle:0.0.12-SNAPSHOT'
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

`skippyAnalyze` triggers a Skippy's Test Impact Analysis that captures
- a hash for each class file in the project and
- coverage data for each skippified test:

```
./gradlew skippyAnalyze
```

The data is stored as a bunch of files in the skippy directory:
```
ls -l skippy

classes.md5
com.example.LeftPadderTest.cov
com.example.RightPadderTest.cov
decisions.log
```

The generated files are consumed by Skippy's [testing libraries](#junit) to make skip-or-execute decisions.

#### skippyClean Task

`skippyClean` empties the skippy directory:

```
./gradlew skippyClean
```

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
        <version>0.0.11</version>
        <executions>
          <execution>
            <goals>
              <goal>analyze</goal>
              <goal>clean</goal>
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
        <version>0.0.12-SNAPSHOT</version>
        <executions>
          <execution>
            <goals>
              <goal>analyze</goal>
              <goal>clean</goal>
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

The plugin adds the `skippy:analyze` and `skippy:clean` goals to your project.

#### skippy:analyze

`skippy:analyze` triggers Skippy's Test Impact Analysis that captures
- a hash for each class file in the project and
- coverage data for each skippified test.

Note that you don't invoke `skippy:analyze` directly. The goal is bound to the `test` lifecycle phase. You trigger
a Skippy analysis by invoking the `test` phase with the parameter `-DskippyAnalyze=true`:
```
mvn test -DskippyAnalyze=true
```

The goal generates a bunch of files in the skippy directory:
```
ls -l skippy

classes.md5
com.example.LeftPadderTest.cov
com.example.RightPadderTest.cov
decisions.log
```

The generated files are consumed by Skippy's [testing libraries](#junit) to make skip-or-execute decisions at test time.

#### skippy:clean

`skippy:clean` empties the skippy directory:

```
mvn skippy:clean
```

## Skippy's Predictive Test Selection

Skippy's JUnit libraries implement Skippy's Predictive Test Selection algorithm. The algorithm makes skip-or-execute decisions
based on
- the local state of the project and
- the data in the `skippy` folder that was generated by Skippy's Test Impact Analysis.

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
    testImplementation 'io.skippy:skippy-junit4:0.0.11'
}
```

Maven:

```xml
<dependency>
    <groupId>io.skippy</groupId>
    <artifactId>skippy-junit4</artifactId>
    <version>0.0.11</version>
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
    testImplementation 'io.skippy:skippy-junit4:0.0.12-SNAPSHOT'
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
        <version>0.0.12-SNAPSHOT</version>
        <scope>test</scope>
    </dependency>
  </dependencies>
  
</project>
```

#### Skippify Your Tests

Add the Skippy class rule to your test:
```groovy
import io.skippy.junit4.Skippy;

public class FooTest {

    @ClassRule
    public static TestRule skippyRule = Skippy.skippify();

    @Test
    public void testFoo() {
        ...
    }

}
```

#### Run Your Tests

The Skippy class rule contains Skippy's Predictive Test Selection algorithm. The algorithm makes skip-or-execute decisions based on
- the state of the project and 
- the data in the `skippy` folder that was generated by Skippy's Test Impact Analysis.

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
    testImplementation 'io.skippy:skippy-junit5:0.0.11'
}
```

Maven:

```xml
<dependency>
    <groupId>io.skippy</groupId>
    <artifactId>skippy-junit5</artifactId>
    <version>0.0.11</version>
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
    testImplementation 'io.skippy:skippy-junit5:0.0.12-SNAPSHOT'
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
        <version>0.0.12-SNAPSHOT</version>
        <scope>test</scope>
    </dependency>
  </dependencies>
  
</project>
```

#### Skippify Your Tests

Annotate the tests you want to skippify with `@Skippified`:

```groovy
import io.skippy.junit5.Skippified;

@Skippified
public class FooTest {

    @Test
    void testFoo() {
        assertEquals("hello", Foo.hello());
    }
}
```

#### Run Your Tests

The Skippy extension contains Skippy's Predictive Test Selection algorithm. The algorithm makes skip-or-execute decisions based on
- the local state of the project and
- the data in the `skippy` folder that was generated by Skippy's Test Impact Analysis.

## Roadmap

- Support for incremental analysis updates
- Support to generate accurate coverage reports despite skipped tests (to support CI pipelines with strict coverage requirements)
- Support for reasoning based on changes in the classpath
- Support for reasoning based on changes in resources (e.g., a change in `application.properties`)
