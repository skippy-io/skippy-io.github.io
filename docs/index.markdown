---
layout: default
title: Home
list_title: News
permalink: /
---

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

## Getting Started

New to Skippy? The best way to get started are the introductory tutorials:
- [Getting Started with Skippy, Gradle & JUnit 5](https://www.skippy.io/tutorials/skippy-gradle-junit5)
- [Getting Started with Skippy, Maven & JUnit 5](https://www.skippy.io/tutorials/skippy-maven-junit5)

## Teaser

Let's take a whirlwind tour of Skippy, Gradle & JUnit 5. The concepts are similar for Maven & JUnit 4.

### Step 1: Install Skippy

```groovy
    plugins {
+       id 'io.skippy' version '0.0.14'
    }
    
    dependencies {
+       testImplementation 'io.skippy:skippy-junit5:0.0.14'
    }
```

### Step 2: Skippify Your Tests

```java
+    import io.skippy.junit5.Skippified;

+    @Skippified
     public class FooTest {     

         @Test
         public void testGetFoo() {
             assertEquals("foo", Foo.getFoo());
         }

     }
```

### Step 3: Perform a Test Impact Analysis

```
./gradlew skippyAnalyze
```

`skippyAnalyze` stores impact data for skippified tests and a bunch of other files in the .skippy folder.

```
ls -l .skippy

classes.md5
com.example.FooTest.cov
...
```

### Step 4: Skippy's Predictive Test Selection In Action

```
./gradlew test

FooTest > testFoo() SKIPPED
BarTest > testBar() SKIPPED
```

Skippy compares the current state of the project with the data in the .skippy folder to make skip-or-execute 
predictions for skippified tests. Skippy detects no changes and skips `FooTest` and `BarTest`.

Introduce a bug in class `Foo`:
```java
     class Foo {
    
         static String getFoo() {
-            return "foo";
+            return "null";
         }
         
     }

```

Re-run the tests:

```
./gradlew test

FooTest > testFoo() FAILED
BarTest > testBar() SKIPPED
```

Skippy detects the change and makes an execute prediction for `FooTest`. The regression is caught quickly - 
unrelated tests remain skipped.

## Contributing

Contributions are always welcome!
- Source Code: [https://github.com/skippy-io/skippy](https://github.com/skippy-io/skippy) 
- Issue Tracker: [https://github.com/skippy-io/skippy/issues](https://github.com/skippy-io/skippy/issues)