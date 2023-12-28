---
layout: page
title: Tutorial
permalink: /tutorials/skippy-gradle-junit5
---

Documentation for Skippy version `0.0.9`.

## Getting Started with Skippy, Gradle & JUnit 5

A quick tour how to use Skippy with Grade and JUnit 5.

__What You Need__
- About 15 minutes
- A favorite text editor or IDE
- Java 17 or later
- Gradle 7.3+


## Setting Up Your Environment

Begin by cloning the skippy-docs repository:
```
git clone git@github.com:skippy-io/skippy-tutorials.git
```

Then, move into the tutorial directory:
```
cd skippy-tutorials/getting-started-with-skippy-and-junit5/
```

Ensure that the project builds successfully:
```````
./gradlew check
```````

A successful build will display:
```
BUILD SUCCESSFUL
```

Execute the `clean` task before proceeding with the rest of the tutorial:

```
./gradlew clean
```

## Exploring the Codebase

Let's take a quick look at the codebase.

### The build.gradle File

`build.gradle` applies the `io.skippy` plugin and adds a dependency to `skippy-junit5`:

```groovy
plugins {
    id 'io.skippy' version '0.0.9'
}

repositories {
    mavenCentral()
}

dependencies {
    testImplementation 'io.skippy:skippy-junit5:0.0.9'
}
```

The plugin adds a couple of tasks that we will use throughout the tutorial:
```
./gradlew tasks
```
Output:

```
...

Skippy tasks
------------
skippyAnalyze
skippyClean

...
```

Note: You can play around with those tasks. If you do so, execute `clean` and `skippyClean` before proceeding with the
rest of the tutorial:
```
./gradlew clean skippyClean
```
{% include_relative getting-started-with-skippy-x-junit5.markdown %}