---
layout: page
title: "Tutorial: Getting started with Skippy, Gradle & JUnit 4"
permalink: /tutorials/skippy-gradle-junit4
---

{% include_relative overview-and-toc.markdown %}

## Setting Up Your Environment

Begin by cloning the [skippy-tutorials](https://github.com/skippy-io/skippy-tutorials) repository:
```
git clone git@github.com:skippy-io/skippy-tutorials.git
```

Then, move into the tutorial directory:
```
cd skippy-tutorials/getting-started-with-skippy-and-junit4/
```

## Exploring the Codebase

Let's take a quick look at the codebase.

### The build.gradle File

`build.gradle` applies the `io.skippy` plugin and adds a dependency to `skippy-junit4`:

```groovy
plugins {
    id 'io.skippy' version '{% include_relative version.markdown %}'
}

dependencies {
    testImplementation 'io.skippy:skippy-junit4:{% include_relative version.markdown %}'
}
```

{% include_relative getting-started-with-skippy-x-junit.markdown %}