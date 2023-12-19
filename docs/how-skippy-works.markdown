---
layout: page
title: How Skippy Works
permalink: /how-skippy-works/
---

Skippy improves test efficiency with a two-pronged approach:
- [a powerful build plugin](https://github.com/skippy-io/skippy/blob/99a4954c5565baa21d355e653c7a0d509ce32682/skippy-gradle/src/main/java/io/skippy/gradle/SkippyPlugin.java#L71) and
- [a smart JUnit extension](https://github.com/skippy-io/skippy/blob/99a4954c5565baa21d355e653c7a0d509ce32682/skippy-junit5/src/main/java/io/skippy/junit5/Skippy.java#L29).

This article provides a high-level overview how both components work together.

Documentation version: `0.0.6`

## The Build Plugin: Overview

Let's start by discussing what happens when you run `./gradlew skippyAnalyze`.

Code: [SkippyPlugin.java](https://github.com/skippy-io/skippy/blob/99a4954c5565baa21d355e653c7a0d509ce32682/skippy-gradle/src/main/java/io/skippy/gradle/SkippyPlugin.java#L71) / [AnalyzeTask.java](https://github.com/skippy-io/skippy/blob/99a4954c5565baa21d355e653c7a0d509ce32682/skippy-gradle/src/main/java/io/skippy/gradle/AnalyzeTask.java#L42)

### Step 1: Collect All Class Files

The plugin traverses the output directories of each source set and collects all class files it finds.

Code: [ClassFileCollector.java](https://github.com/skippy-io/skippy/blob/99a4954c5565baa21d355e653c7a0d509ce32682/skippy-gradle/src/main/java/io/skippy/gradle/collector/ClassFileCollector.java#L36)

### Step 2: Identify Skippified Tests

Using [ASM](https://asm.ow2.io/)'s bytecode analysis capabilities, it then selects the skippified tests among
the class files collected in Step 1. This is accomplished by checking for classes annotated with
`@ExtendsWith(Skippy.class)`.

Code: [SkippyJUnit5Detector](https://github.com/skippy-io/skippy/blob/99a4954c5565baa21d355e653c7a0d509ce32682/skippy-gradle/src/main/java/io/skippy/gradle/asm/SkippyJUnit5Detector.java#L33)

### Step 3: Create JaCoCo Coverage Report For Each Skippified Test

Using Gradle's [Tooling API](https://docs.gradle.org/current/userguide/third_party_integration.html#embedding),
the task then creates individual [JaCoCo](https://www.jacoco.org/) coverage reports for the skippified test identified
in Step 2. Think of it as the programmatic counterpart to the following shell script:

```
./gradlew test jacocoTestReport --tests "com.example.Test1"
./gradlew test jacocoTestReport --tests "com.example.Test2"
...
./gradlew test jacocoTestReport --tests "com.example.Test8"
./gradlew test jacocoTestReport --tests "com.example.Test9"
```

Code: [AnalyzeTask.java#createCoverageReportsForSkippifiedTests](https://github.com/skippy-io/skippy/blob/99a4954c5565baa21d355e653c7a0d509ce32682/skippy-gradle/src/main/java/io/skippy/gradle/AnalyzeTask.java#L68)

The individual coverage reports are stored in the `skippy` directory:
```
ls -l skippy

com.example.Test1.csv
com.example.Test2.csv
...
```
Skippy uses those reports to generate a Test Impact Analysis. Test Impact Analysis is a $50 term for a 50 cent
concept: It's a mapping between tests and the classes they cover:
```
{
    // Test1 covers itself and class Foo

    "com.example.Test1": [
        "com.example.Test1",
        "com.example.Foo"
    ],

    // Test2 covers itself and class Bar

    "com.example.Test2": [
        "com.example.Test2",
        "com.example.Bar"
    ]
}
```
Code: [TestImpactAnalysis.java](https://github.com/skippy-io/skippy/blob/99a4954c5565baa21d355e653c7a0d509ce32682/skippy-core/src/main/java/io/skippy/core/TestImpactAnalysis.java#L36)

### Step 4: Create A Hash For Each Class File

In addition to the Test Impact Analysis from Step 3, Skippy creates hashes for all class files collected in Step 1.
Those hashes are stored in a file called `analyzedFiles.txt`.

Example:
```
build/classes/java/main/com/example/Foo.class:9U3+WYit7uiiNqA9jplN2A==
build/classes/java/test/com/example/Test1.class:3KxzE+CKm6BJ3KetctvnNA==
build/classes/java/main/com/example/Bar.class:ZT0GoiWG8Az5TevH9/JwBg==
build/classes/java/test/com/example/Test2.class:naR4eGh3LU+eDNSQXvsIyw==
```

A single failing test case with fail the entire `skippyAnalyze` task. This way, Skippy knows that the
Test Impact Analysis and content of the `analyzedFiles.txt` corresponds to a successful execution of the test suite.

The class files are not hashed as-is. Instead, Skippy uses ASM's bytecode manipulation capabilities to hash a copy of
the original class file that is stripped of all debug information (like [LineNumberTable attributes](https://docs.oracle.com/javase/specs/jvms/se17/html/jvms-4.html#jvms-4.7.12)).

This allows Skippy to treat certain changes like
- change in formatting and indentation,
- updated JavaDocs and
- addition of newlines and linebreaks

as 'no-ops'.

Code: [AnalyzeTask.java#createAnalyzedFilesTxt](https://github.com/skippy-io/skippy/blob/99a4954c5565baa21d355e653c7a0d509ce32682/skippy-gradle/src/main/java/io/skippy/gradle/AnalyzeTask.java#L78)

## Conditional Test Execution: Overview

Now, let's take a look how this data is utilized when tests are executed.

### The Skippy JUnit5 Extension

The Skippy JUnit 5 extension turns a regular JUnit tests into a skippified test:

```java
@ExtendWith(Skippy.class)
public class FooTest {

    @Test
    void testFoo() {
        assertEquals("hello", Foo.doSomething());
    }

}
```
Code: [Skippy.java](https://github.com/skippy-io/skippy/blob/99a4954c5565baa21d355e653c7a0d509ce32682/skippy-junit5/src/main/java/io/skippy/junit5/Skippy.java#L29C22-L29C22)

At execution time, the extension applies the following algorithm:

```
  Step 1: Does the skippy folder contain a Test Impact Analysis and a file named analyzedFiles.txt?

            Yes: Continue with Step 2.
            No:  Execute FooTest.

  Step 2: Does analyzedFiles.txt contain a hash for FooTest?

            Yes: Continue with Step 3.
            No:  Execute FooTest.

  Step 3: Is the current hash of FooTest equal to the hash in analyzedFiles.txt?

            Yes: Continue with Step 4.
            No:  Execute FooTest.

  Step 4: For all classes that are covered by FooTest:

            Is the current hash of the covered class equal to the hash in analyzedFiles.txt?

                No:  Execute FooTest.

  Step 5: Skip FooTest
```

Code: [SkippyAnalysis.java#executionRequired](https://github.com/skippy-io/skippy/blob/99a4954c5565baa21d355e653c7a0d509ce32682/skippy-core/src/main/java/io/skippy/core/SkippyAnalysis.java#L77)

And voila - that's how Skippy works.