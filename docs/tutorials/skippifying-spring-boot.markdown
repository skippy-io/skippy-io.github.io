---
layout: page
title: "Skippifying Spring Boot"
permalink: /tutorials/skippifying-spring-boot
---

This tutorial goes over how to skippify [Spring Boot](https://github.com/spring-projects/spring-boot).
The scale and complexity of Spring Boot's build make it a great choice to put Skippy to the test.

All experiments for this article were performed with the following setup:
- MacBook Air M2, 2022
- 8 GB RAM
- Java 17 

All changes are made in the fork [skippy-io/spring-boot-skippified](https://github.com/skippy-io/spring-boot-skippified).

## Table Of Contents

- [Build Modification](#build-modification)
- [Baseline: Test Execution Without Skippy](#baseline-test-execution-without-skippy)
- [Test Impact Analysis](#test-impact-analysis)
- [Predictive Test Selection](#predictive-test-selection)
- [Summary](#summary)

## Build Modification

The fork contains two new commits:
- [f24d46](https://github.com/skippy-io/spring-boot-skippified/commit/f24d46088d0ce813e715ca657e9b08b345b73dcd) adds
additional log output that will be useful throughout this tutorial
- [a16530](https://github.com/skippy-io/spring-boot-skippified/commit/a16530d85f54fd34b66b5fef1dff70a97b3ddc07)
skippifies the tests

### Change Set

Let's walk through the change set of [a16530](https://github.com/skippy-io/spring-boot-skippified/commit/a16530d85f54fd34b66b5fef1dff70a97b3ddc07).
This commit skippifies the build with 5 lines of configuration changes. There is no need to modify any files in src/main or src/test.

#### build.gradle

```
    plugins {
        ...
+       id 'io.skippy' version '0.0.13'
    }
    
    dependencies {
        ...
+       testRuntimeOnly("io.skippy:skippy-junit5:0.0.13")
    }

    test {
        ...
+       jvmArgs += "-Djunit.jupiter.extensions.autodetection.enabled=true"
    }
```

The first two changes are self-explanatory: The PR applies the Skippy plugin and declares a dependency to `skippy-junit5`. 
The last change enables JUnit 5's [Automatic Extension Detection](https://junit.org/junit5/docs/current/user-guide/#extensions-registration-automatic)
(more on that in the next section).

#### src/test/resources/META-INF

Lastly, the PR adds META-INF/services/org.junit.jupiter.api.extension.Extension to the resources folder
in src/test:

```
+   io.skippy.junit5.SkipOrExecuteCondition
+   io.skippy.junit5.CoverageFileCallbacks
```

This file, together with the newly added JVM argument in build.gradle, is equivalent to adding `@Skippified` to every
test class in src/test. Thanks to Skippy's non-invasive design, 5 lines of configuration changes is all it takes
to skippify more than 4000 tests.

## Baseline: Test Execution Without Skippy

Clone the repo and checkout revision [f24d46](https://github.com/skippy-io/spring-boot-skippified/commit/f24d46088d0ce813e715ca657e9b08b345b73dcd) where the build contains additional logging but has not been skippified yet:
```
git clone git@github.com:skippy-io/spring-boot-skippified.git
cd spring-boot-skippified/spring-boot-project/spring-boot
git checkout f24d46
```

Run the `check` task:
```
../../gradlew check --no-build-cache --rerun-tasks --no-daemon
```

The purpose of the command-line arguments `--no-build-cache`, `--rerun-tasks` and `--no-daemon` is to have a consistent
baseline to measure the builds in isolation.

Over 10 runs, `check` averages 2 minutes and 55 seconds, with 2 minutes and 7 seconds dedicated to test execution.

## Test Impact Analysis

Checkout the skippified revision [a16530](https://github.com/skippy-io/spring-boot-skippified/commit/a16530d85f54fd34b66b5fef1dff70a97b3ddc07) of the build:

```
git checkout a16530
```

Run the `skippyAnalyze` task:
```
../../gradlew skippyAnalyze --no-build-cache --rerun-tasks --no-daemon
```

Over 10 runs, `skippyAnalyze` averages 3 minutes and 18 seconds, with 2 minutes and 27 seconds dedicated to test execution.
It analyzes over 4000 tests and more than 2500 classes with minimal overhead, adding only 23 seconds (or 13%) to the total
time compared to `./gradlew check`.

## Predictive Test Selection

Next, let's see how well Skippy's Predictive Test Selection performs.

Re-run the tests:
```
../../gradlew check --no-build-cache --rerun-tasks --no-daemon
```

Output:
```
---------------------------------------------------------------------
|  Results: SUCCESS (4060 tests, 0 passed, 0 failed, 4060 skipped)  |
---------------------------------------------------------------------

Task timings:
   ...
   2971ms  :spring-boot-project:spring-boot:test

BUILD SUCCESSFUL in 54s
42 actionable tasks: 42 executed
```

The `check` task shows a substantial 69% decrease in runtime, while the `test` task exhibits an even more remarkable
98% reduction, both measured against the baseline.

Skippy detects that nothing has changed since the Test Impact Analysis has been performed, and effectively makes skip
predictions for every test.

Let's introduce some bugs and see what happens.

### Bug 1

`org.springframework.boot.admin` is the first package in alphabetical order. The package contains one class - 
let's introduce a bug in the constructor:

```
    public class SpringApplicationAdminMXBeanRegistrar ... {    
        ...        
        public SpringApplicationAdminMXBeanRegistrar(String name) throws MalformedObjectNameException {
-           this.objectName = new ObjectName(name);
+           this.objectName = null;
        }        
        ...
    }
```

PR: [https://github.com/skippy-io/spring-boot-skippified/pull/3/files](https://github.com/skippy-io/spring-boot-skippified/pull/3/files)

Re-run the tests against the baseline build:
```
git checkout f24d46
../../gradlew check --no-build-cache --rerun-tasks --no-daemon
```

Output:
```
----------------------------------------------------------------------
|  Results: FAILURE (4387 tests, 4362 passed, 3 failed, 22 skipped)  |
----------------------------------------------------------------------

Found test failures in 1 test task:

:spring-boot-project:spring-boot:test
    org.springframework.boot.admin.SpringApplicationAdminMXBeanRegistrarTests > environmentIsExposed()
    org.springframework.boot.admin.SpringApplicationAdminMXBeanRegistrarTests > shutdownApp()
    org.springframework.boot.admin.SpringApplicationAdminMXBeanRegistrarTests > validateReadyFlag()

Task timings:
    ...
    127846ms  :spring-boot-project:spring-boot:test
```

Next, re-run the tests against the skippified build:
```
git checkout a16530
../../gradlew check --no-build-cache --rerun-tasks --no-daemon
```

Output:
```
---------------------------------------------------------------------
|  Results: FAILURE (4060 tests, 1 passed, 3 failed, 4056 skipped)  |
---------------------------------------------------------------------

Found test failures in 1 test task:

:spring-boot-project:spring-boot:test
    org.springframework.boot.admin.SpringApplicationAdminMXBeanRegistrarTests > environmentIsExposed()
    org.springframework.boot.admin.SpringApplicationAdminMXBeanRegistrarTests > shutdownApp()
    org.springframework.boot.admin.SpringApplicationAdminMXBeanRegistrarTests > validateReadyFlag()

Task timings:
    ...
    5846ms  :spring-boot-project:spring-boot:test
```

Comparison:
- ✅ Both the baseline and the skippified build prevent the regression
- ⬇️️ Skippy reduces the number of executed tests from 4365 to 4 (99% reduction)
- 🚀 Skippy reduces the test time from 2 minutes and 2 seconds to 6 seconds (95% reduction)

### Bug 2

The next bug is introduced in `org.springframework.boot.flyway.FlywayDatabaseInitializerDetector`: 

```
    class FlywayDatabaseInitializerDetector ... {    
        ... 
        @Override
        protected Set<Class<?>> getDatabaseInitializerBeanTypes() {
-            return Collections.singleton(Flyway.class);
+            return Collections.emptySet();
        }       
        ...    
    }
```

PR: [https://github.com/skippy-io/spring-boot-skippified/pull/4/files](https://github.com/skippy-io/spring-boot-skippified/pull/4/files)

Comparison:
- ❎️ Neither the baseline nor the skippified build detect the regression - Skippy is as good as the baseline
- ⬇️️ Skippy reduces the number of executed tests from 4365 to 0 (100% reduction)
- 🚀 Skippy reduces the test time from 2 minutes and 2 seconds to 3 seconds (98% reduction)

### Bug 3

The next bug is introduced in `org.springframework.boot.r2dbc.init.R2dbcScriptDatabaseInitializer`:

```
    public class R2dbcScriptDatabaseInitializer ... {
        ...  
        @Override
        protected void runScripts(Scripts scripts) {
+           if (true) {
+               return;
+           }
            ...
        }
        ...    
    }
```

PR: [https://github.com/skippy-io/spring-boot-skippified/pull/5/files](https://github.com/skippy-io/spring-boot-skippified/pull/5/files)

Comparison:
- ✅ Both the baseline and the skippified build prevent the regression
- ⬇️️ Skippy reduces the number of executed tests from 4365 to 14 (99% reduction)
- 🚀 Skippy reduces the test time from 2 minutes and 2 seconds to 6 seconds (95% reduction)

### Bug 4

The next bug is introduced in `org.springframework.boot.security.reactive.ApplicationContextServerWebExchangeMatcher`:

```
    public abstract class ApplicationContextServerWebExchangeMatcher<C> ... {  
        ...
        protected Supplier<C> getContext(ServerWebExchange exchange) {
+           if (true) {
+               return null;
+           }
            ...
        }
        ...
    }
```

PR: [https://github.com/skippy-io/spring-boot-skippified/pull/6/files](https://github.com/skippy-io/spring-boot-skippified/pull/6/files)

Comparison:
- ✅ Both the baseline and the skippified build prevent the regression
- ⬇️️ Skippy reduces the number of executed tests from 4365 to 5 (99% reduction)
- 🚀 Skippy reduces the test time from 2 minutes and 2 seconds to 5 seconds (96% reduction)

### Bug 5

The last bug is introduced in `org.springframework.boot.SpringApplication`:

```
    public class SpringApplication {
        ...
        public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
+           if (true) {
+               return null;
+           }
            return new SpringApplication(primarySources).run(args);
        }
        ...
    }
```

PR: [https://github.com/skippy-io/spring-boot-skippified/pull/7/files](https://github.com/skippy-io/spring-boot-skippified/pull/7/files)

Comparison:
- ✅ Both the baseline and the skippified build prevent the regression
- ⬇️️ Skippy reduces the number of executed tests from 4365 to 430 (90% reduction)
- 🚀 Skippy reduces the test time from 2 minutes and 2 seconds to 20 seconds (84% reduction)

## Summary

- Skippy's Test Impact Analysis is shown to be highly efficient for large projects. It analyzes over 4000 tests and more
  than 2500 classes with minimal overhead, adding only 23 seconds (or 13%) to the total time compared to `./gradlew check`.
- Skippy’s Predictive Test Selection demonstrates its effectiveness in identifying all regressions that are caught when
  running all tests. This feature reduces the number of tests that need to be executed by 90% to 100% and
  cuts down the time required for test execution by 84% to 98%.

| Bug                                                                    | Detected<br />Baseline | Detected<br/>Skippy | Tests run<br/>Baseline | Tests run<br/>Skippy | Test time<br/>Baseline | Test time<br/>Skippy |
|------------------------------------------------------------------------|------------------------|---------------------|------------------------|----------------------|------------------------|----------------------|
| [#1](https://github.com/skippy-io/spring-boot-skippified/pull/3/files) | ✅                     | ✅                  | 4365                   | 4 (-99%)             | 2m 2s                  | 6s (-95%)            |
| [#2](https://github.com/skippy-io/spring-boot-skippified/pull/4/files) | ❌                     | ❌                  | 4365                   | 0 (-100%)            | 2m 2s                  | 3s (-98%)            |
| [#3](https://github.com/skippy-io/spring-boot-skippified/pull/5/files) | ✅                     | ✅                  | 4365                   | 14 (-99%)            | 2m 2s                  | 6s (-95%)            |
| [#4](https://github.com/skippy-io/spring-boot-skippified/pull/6/files) | ✅                     | ✅                  | 4365                   | 5 (-99%)             | 2m 2s                  | 5s (-96%)            |
| [#5](https://github.com/skippy-io/spring-boot-skippified/pull/7/files) | ✅                     | ✅                  | 4365                   | 430 (-90%)           | 2m 2s                  | 20s (-84%)           |

