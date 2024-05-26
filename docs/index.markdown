---
layout: default
title: Home
list_title: News
permalink: /
---

## What is it?

Skippy is a Test Impact Analysis & Predictive Test Selection framework for Java and the JVM. It cuts down on unnecessary testing
and flakiness without compromising the integrity of your builds. You can run it from the command line, your favorite IDE
and continuous integration server. Skippy supports Gradle, Maven, JUnit 4 and JUnit 5.

Skippy is specifically designed to prevent regressions in your codebase. 
It supports all types of tests where the tests and the code under test run in the same JVM.
It is best suited for deterministic tests, even those prone to occasional flakiness.
It provides the most value for test suites that are either slow or flaky (regardless of whether the test suite contains unit, integration, or functional tests).

<center>
<iframe width="560" height="315" src="https://www.youtube.com/embed/VZ_MmQI0mOA?si=HPYFrLmZ_pciM6jn" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
</center>

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
+        id 'io.skippy' version '0.0.20'
     }
    
     dependencies {
+        testImplementation 'io.skippy:skippy-junit5:0.0.20'
     }
```

### Step 2: Enable Predictive Test Selection

Annotate the test you want to optimize with `@PredictWithSkippy`:

```java
+    import io.skippy.junit5.PredictWithSkippy;

+    @PredictWithSkippy
     public class FooTest {     

         @Test
         public void testFoo() {
             assertEquals("foo", Foo.getFoo());
         }

     }
```

### Step 3: Skippy In Action

Run the tests:
```
./gradlew test

FooTest > testFoo() PASSED
BarTest > testBar() PASSED
```

Skippy performs a Test Impact Analysis every time you run a test. The result is stored in the .skippy folder:

```
ls -l .skippy                       

test-impact-analysis.json
```

This data allows Skippy to make intelligent skip-or-execute predictions. Let's see what happens when you run the tests again:

```
./gradlew test --rerun

FooTest > testFoo() SKIPPED
BarTest > testBar() SKIPPED 
```

Skippy detects that nothing has changed and skips both tests.

Next, introduce a bug in class `Foo`:
```java
     class Foo {
    
         static String getFoo() {
-            return "foo";
+            return null;
         }
         
     }
```

Re-run the tests:

```
./gradlew test

FooTest > testFoo() FAILED
    org.opentest4j.AssertionFailedError: expected: <foo> but was: <null>
BarTest > testBar() SKIPPED 
```

Skippy detects the change and executes `FooTest`. The regression is caught quickly - `BarTest` remains skipped.

Fix the bug and re-run the tests:

```
./gradlew test

FooTest > testFoo() PASSED
BarTest > testBar() SKIPPED 
```

Skippy executes `FooTest` and updates the data in the .skippy folder.
Both tests will be skipped when you run them again:

```
./gradlew test --rerun

FooTest > testFoo() SKIPPED
BarTest > testBar() SKIPPED 
```

## Contributing

Contributions are always welcome!
- Source Code: [https://github.com/skippy-io/skippy](https://github.com/skippy-io/skippy) 
- Issue Tracker: [https://github.com/skippy-io/skippy/issues](https://github.com/skippy-io/skippy/issues)