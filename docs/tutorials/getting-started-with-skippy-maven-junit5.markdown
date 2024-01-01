---
layout: page
title: Tutorial
permalink: /tutorials/skippy-maven-junit5
---

Documentation for Skippy version `0.0.11`.

## Getting Started with Skippy, Maven & JUnit 5

A quick tour how to use Skippy with Grade and JUnit 5.

__What You Need__
- About 15 minutes
- A favorite text editor or IDE
- Java 17 or later
- Maven 3.9+

## Table Of Contents

- [Setting Up Your Environment](#setting-up-your-environment)
- [Exploring The Codebase](#exploring-the-codebase)
- [Run The Tests](#run-the-tests)
- [Skippy’s Test Impact Analysis](#skippys-test-impact-analysis)
- [Skippy's Predictive Test Selection In Action](#skippys-predictive-test-selection-in-action)

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
mvn verify
```````

A successful build will display:
```
[INFO] BUILD SUCCESS
```

Execute the `clean` task before proceeding with the rest of the tutorial:

```
mvn clean
```

## Exploring the Codebase

Let's take a quick look at the codebase.

### The pom.xml File

`pom.xml` declares a dependency to `skippy-junit5`:

```xml
<dependency>
    <groupId>io.skippy</groupId>
    <artifactId>skippy-junit5</artifactId>
    <version>0.0.11</version>
    <scope>test</scope>
</dependency>
```

It also adds the Skippy plugin:
```xml
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

Note: You can play around with those tasks. If you do so, execute `clean` and `skippy:clean` before proceeding with the
rest of the tutorial:
```
mvn clean skippy:clean
```
{% include_relative getting-started-with-skippy-x-junit5.markdown %}