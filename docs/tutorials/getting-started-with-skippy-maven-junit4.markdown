---
layout: page
title: "Tutorial: Getting started with Skippy, Maven & JUnit 4"
permalink: /tutorials/skippy-maven-junit4
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

### The pom.xml File

`pom.xml` declares a dependency to `skippy-junit4`:

```xml
<dependency>
    <groupId>io.skippy</groupId>
    <artifactId>skippy-junit4</artifactId>
    <version>{% include_relative version.markdown %}</version>
    <scope>test</scope>
</dependency>
```

It also adds the Skippy plugin:
```xml
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
```

Since Skippy internally depends on [JaCoCo](https://www.jacoco.org/), the JaCoCo plugin has to be added as well:
```xml
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
```

{% include_relative getting-started-with-skippy-x-junit.markdown %}