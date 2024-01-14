---
layout: page
title: How Skippy Works -  A Deep Dive
permalink: /tutorials/how-skippy-works
---

Documentation for Skippy version `0.0.13`.

Skippy is built on top of three pillars:
- Build Plugins for [Gradle](https://github.com/skippy-io/skippy/tree/main/skippy-gradle) and [Maven](https://github.com/skippy-io/skippy/tree/main/skippy-maven) that implement Skippy's Test Impact Analysis
- JUnit libraries for [JUnit 4](https://github.com/skippy-io/skippy/tree/main/skippy-junit4) & [JUnit 5](https://github.com/skippy-io/skippy/tree/main/skippy-junit5) that implement Skippy's Predictive Test Selection
- [JaCoCo](https://github.com/jacoco/jacoco)'s dynamic bytecode analysis to capture per-test coverage data

In the next sections, we'll take a technical deep dive to learn how Skippy, JaCoCo, Gradle and JUnit 5 work together. The
concepts are similar for Maven and JUnit 4. The code snippets on this page have been simplified for the sake of clarity.
GitHub links to the actual implementations are provided.

## The Skippy Gradle Plugin

Skippy's Gradle plugin adds the `skippyClean` and `skippyAnalyze` tasks:
```
class SkippyPlugin implements Plugin<Project> {

    @Override
    public void apply(Project project) {
        project.getTasks().register("skippyClean", SkippyCleanTask.class);
        project.getTasks().register("skippyAnalyze", SkippyAnalyzeTask.class);
    }

}
```
GitHub: [SkippyPlugin.java](https://github.com/skippy-io/skippy/blob/3eb9b9d4e659f2d00b34ff7797db5ed1558c78ad/skippy-gradle/src/main/java/io/skippy/gradle/SkippyPlugin.java#L33)

`skippyAnalyze` triggers Skippy's Test Impact Analysis. `skippyClean` is the corresponding clean-up task.

### skippyAnalyze

`skippyAnalyze` executes in two steps:
1. Generation of .cov files
2. Generation of the classes.md5 file

We will discuss each step in the following sections.

#### Step 1: Generation of .cov Files

Let's take a look at the implementation of `skippyAnalyze`:

```
class SkippyAnalyzeTask extends DefaultTask {

    @Inject
    public SkippyAnalyzeTask() {
        dependsOn("check");
        getProject().getPlugins().apply(JacocoPlugin.class);
        getProject().getTasks().withType(Test.class,
            test -> test.environment(SkippyConstants.TEST_IMPACT_ANALYSIS_RUNNING, true)
        );
        doLast((task) -> skippyBuildApi.writeClassesMd5FileAndCompactCoverageFiles());
    }
    
}
```

GitHub: [SkippyAnalyzeTask.java](https://github.com/skippy-io/skippy/blob/3eb9b9d4e659f2d00b34ff7797db5ed1558c78ad/skippy-gradle/src/main/java/io/skippy/gradle/SkippyAnalyzeTask.java#L43)

The task declares a dependency to Gradle's `check` lifecycle task:

```
dependsOn("check")
```

This dependency triggers the test suites every time `skippyAnalyze` is invoked. `skippyAnalysis` wouldn't be
particularly useful if the only thing it did was to run tests. After all, its main purpose is to perform a Test Impact
Analysis. To accomplish this, it applies the `JacocoPlugin` and sets the `TEST_IMPACT_ANALYSIS_RUNNING` environment variable for
all test tasks:

```
getProject().getPlugins().apply(JacocoPlugin.class);
getProject().getTasks().withType(Test.class,
        test -> test.environment(SkippyConstants.TEST_IMPACT_ANALYSIS_RUNNING, true)
);
```

The environment variable is a signal for Skippy's JUnit libraries to emit coverages files during the execution of the test suite:

```
import org.jacoco.agent.rt.IAgent;
import org.jacoco.agent.rt.RT;

class SkippyTestApi {

    public static void prepareCoverageDataCaptureFor(Class<?> testClass) {
        if ( ! testImpactAnalysisIsRunning()) {
            return;
        }
        IAgent agent = RT.getAgent();
        agent.reset();
    }

    public static void captureCoverageDataFor(Class<?> testClass) {
        if ( ! testImpactAnalysisIsRunning()) {
            return;
        }
        IAgent agent = RT.getAgent();
        byte[] executionData = agent.getExecutionData(true);
        // write .cov file
    }

    private static boolean testImpactAnalysisIsRunning() {
        return Boolean.valueOf(System.getProperty(TEST_IMPACT_ANALYSIS_RUNNING)) ||
                Boolean.valueOf(System.getenv().get(TEST_IMPACT_ANALYSIS_RUNNING));
    }

}
```

GitHub: [SkippyTestApi.java](https://github.com/skippy-io/skippy/blob/3eb9b9d4e659f2d00b34ff7797db5ed1558c78ad/skippy-junit-common/src/main/java/io/skippy/junit/SkippyTestApi.java#L93)

Skippy uses JUnit 5's extension mechanism to invoke 

- `SkippyTestApi#prepareCoverageDataCaptureFor` before and
- `SkippyTestApi#captureCoverageDataFor` after 

the execution of a test class:

```
class CoverageFileCallbacks implements BeforeAllCallback, AfterAllCallback {

    @Override
    public void beforeAll(ExtensionContext context) {
        context.getTestClass().ifPresent(SkippyTestApi::prepareCoverageDataCaptureFor);
    }

    @Override
    public void afterAll(ExtensionContext context) {
        context.getTestClass().ifPresent(SkippyTestApi::captureCoverageDataFor);
    }
}
```

GitHub: [CoverageFileCallbacks.java](https://github.com/skippy-io/skippy/blob/3eb9b9d4e659f2d00b34ff7797db5ed1558c78ad/skippy-junit5/src/main/java/io/skippy/junit5/CoverageFileCallbacks.java#L27)

As a result, `skippyAnalyze` generates .cov files that contains coverage data for each skippified test (more on that
soon):
```
ls -l skippy

com.example.LeftPadderTest.cov
com.example.RightPadderTest.cov
...
```

Each .cov file contains the list of classes that are covered by the corresponding test. 

Example:
```
cat com.example.LeftPadderTest.cov
 
com.example.LeftPadder
com.example.LeftPadderTest
com.example.StringUtils
```

#### Step 2: Generation Of The classes.md5 File

`skippyAnalyze` also creates a hash for each class file in one of the build's output folders:

```
class SkippyAnalyzeTask extends DefaultTask {

    @Inject
    public SkippyAnalyzeTask() {
        ...
        doLast((task) -> skippyBuildApi.writeClassesMd5FileAndCompactCoverageFiles());
    }
    
}
```

GitHub: [SkippyAnalyzeTask.java](https://github.com/skippy-io/skippy/blob/3eb9b9d4e659f2d00b34ff7797db5ed1558c78ad/skippy-gradle/src/main/java/io/skippy/gradle/SkippyAnalyzeTask.java#L43) \| [ClassesMd5Writer.java](https://github.com/skippy-io/skippy/blob/3eb9b9d4e659f2d00b34ff7797db5ed1558c78ad/skippy-build-common/src/main/java/io/skippy/build/ClassesMd5Writer.java#L53) \| [DebugAgnosticHash.java](https://github.com/skippy-io/skippy/blob/3eb9b9d4e659f2d00b34ff7797db5ed1558c78ad/skippy-common/src/main/java/io/skippy/core/DebugAgnosticHash.java#L46)

Those hashes are stored in the classes.md5 file in the skippy folder.

Example:
```
cat skippy/classes.md5
 
build/classes/java/main:com/example/LeftPadder.class:9U3+WYit7uiiNqA9jplN2A==
build/classes/java/main:com/example/RightPadder.class:ZT0GoiWG8Az5TevH9/JwBg==
build/classes/java/main:com/example/StringUtils.class:4VP9fWGFUJHKIBG47OXZTQ==
build/classes/java/test:com/example/LeftPadderTest.class:sGLJTZJw4beE9m2Kg6chUg==
build/classes/java/test:com/example/RightPadderTest.class:E/ObvuQTODFFqU6gxjbxTQ==
build/classes/java/test:com/example/StringUtilsTest.class:p+N8biKVOm6BltcZkKcC/g==
build/classes/java/test:com/example/TestConstants.class:3qNbG+sSd1S1OGe0EZ9GPA==
```

The class files are not hashed as-is. Instead, Skippy uses ASM's bytecode manipulation capabilities to hash a copy of
the original class file that is stripped of all debug information (like 
[LineNumberTable attributes](https://docs.oracle.com/javase/specs/jvms/se17/html/jvms-4.html#jvms-4.7.12)).

This allows Skippy to treat certain changes like
- change in formatting and indentation,
- updated JavaDocs and
- addition of newlines and linebreaks

as 'no-ops'.

## Predictive Test Selection

Skippy utilizes the data that is generated by Skippy's Test Impact Analysis to predict which tests to run. Let's first
discuss the concept of a skippified test.

### @Skippified

Skippy's `@Skippified` annotation turns a regular JUnit tests into a skippified test:

```java
import io.skippy.junit5.Skippified;

@Skippified
public class FooTest {

    @Test
    void testFoo() {
        assertEquals("hello", Foo.doSomething());
    }

}
```

This meta annotation applies two extensions: `SkipOrExecuteCondition` and `CoverageFileCallbacks`.

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@ExtendWith(SkipOrExecuteCondition.class)
@ExtendWith(CoverageFileCallbacks.class)
public @interface Skippified {
}
```

GitHub: [Skippyfied.java](https://github.com/skippy-io/skippy/blob/3eb9b9d4e659f2d00b34ff7797db5ed1558c78ad/skippy-junit5/src/main/java/io/skippy/junit5/Skippified.java#L53)


The `SkipOrExecuteCondition` extension is used at test time to make a skip-or-execute predictions:

```
class SkipOrExecuteCondition implements ExecutionCondition {

    SkippyTestApi skippyTestApi;

    @Override
    public ConditionEvaluationResult evaluateExecutionCondition(ExtensionContext context) {
        if (skippyTestApi.testNeedsToBeExecuted(context.getTestClass().get())) {
            return ConditionEvaluationResult.enabled("");
        }
        return ConditionEvaluationResult.disabled("");
    }

}
```

GitHub: [SkipOrExecuteCondition.java](https://github.com/skippy-io/skippy/blob/3eb9b9d4e659f2d00b34ff7797db5ed1558c78ad/skippy-junit5/src/main/java/io/skippy/junit5/SkipOrExecuteCondition.java#L29)

`SkippyExecutionCondition` internally delegates the skip-or-execute prediction to an instance of `SkippyTestApi`.
`SkippyTestApi` passes the call through to an instance of `SkippyAnalysis`:

```
class SkippyAnalysis {

    HashedClasses hashedClasses;
    CoverageData coverageData;

    PredictionWithReason predict(FullyQualifiedClassName testFqn) {
        if (coverageData.noDataAvailableFor(testFqn)) {
            return PredictionWithReason.execute(NO_COVERAGE_DATA_FOR_TEST);
        }
        if (hashedClasses.noDataFor(testFqn)) {
            return PredictionWithReason.execute(NO_HASH_FOR_TEST);
        }
        if (hashedClasses.hasChanged(testFqn)) {
            return PredictionWithReason.execute(BYTECODE_CHANGE_IN_TEST);
        }
        return predictBasedOnCoveredClasses(testFqn);
    }

    PredictionWithReason predictBasedOnCoveredClasses(FullyQualifiedClassName testFqn) {
        for (var coveredClassFqn : coverageData.getCoveredClasses(testFqn)) {
            if (hashedClasses.hasChanged(coveredClassFqn)) {
                return PredictionWithReason.execute(BYTECODE_CHANGE_IN_COVERED_CLASS);
            }
            if (hashedClasses.noDataFor(coveredClassFqn)) {
                return PredictionWithReason.execute(NO_HASH_FOR_COVERED_CLASS);
            }
        }
        return PredictionWithReason.skip(Reason.NO_CHANGE);
    }

}
```

GitHub: [SkippyAnalysis.java](https://github.com/skippy-io/skippy/blob/3eb9b9d4e659f2d00b34ff7797db5ed1558c78ad/skippy-junit-common/src/main/java/io/skippy/junit/SkippyAnalysis.java#L89)

The fields `hashedClasses` and `coverageData` are the programmatic representation of Skippy's Test Impact Analysis. 
`SkippyAnalysis#predict` is the implementation of Skippy's Predictive Test Selection.

You might ask: Why the complex indirections, like the `SkippyExecutionCondition -> SkippyTestApi -> SkippyAnalysis` call chain? There are two key reasons:

- It allows to support various build tools (like Gradle & Maven) and unit testing frameworks (such as JUnit 4 & JUnit 5) while keeping the maintenance costs in check.
- It ensures information hiding between subprojects. For instance, `SkippyExecutionCondition` only knows about `SkippyTestApi`, not any other classes in `skippy-junit-common`.

And voila - that's how Skippy works.

{% include_relative comments.markdown %}