---
layout: page
title: How Skippy Works
permalink: /tutorials/how-skippy-works
---

Documentation for Skippy version `0.0.11`.

Skippy consists of two pillars:
- Build Plugins for [Gradle](https://github.com/skippy-io/skippy/tree/main/skippy-gradle) and [Maven](https://github.com/skippy-io/skippy/tree/main/skippy-maven) that implement Skippy's Test Impact Analysis
- JUnit libraries for [JUnit 4](https://github.com/skippy-io/skippy/tree/main/skippy-junit4) & [JUnit 5](https://github.com/skippy-io/skippy/tree/main/skippy-junit5) that implement Skippy's Predictive Test Selection

In the next sections, we'll look at how Skippy, Gradle and JUnit 5 work together. Although we focus on these tools, the
concepts are similar for Maven and JUnit 4.

## The Skippy Gradle Plugin

Skippy's Gradle plugin adds the `skippyClean` and `skippyAnalyze` tasks to a project. We will discuss `skippyAnalyze`
in detail since it triggers Skippy's Test Impact Analysis. The plugin additionally applies the JaCoCo plugin, which 
Skippy leverages internally to collect coverage data:

```
public final class SkippyPlugin implements org.gradle.api.Plugin<Project> {

    @Override
    public void apply(Project project) {
        project.getPlugins().apply(JacocoPlugin.class);
        project.getTasks().register("skippyClean", SkippyCleanTask.class);
        project.getTasks().register("skippyAnalyze", SkippyAnalyzeTask.class);
        ...
    }

}
```

Code: [SkippyPlugin.java](https://github.com/skippy-io/skippy/blob/2c0b7b78adf18edcaa19a397ab74619d76ad1b7e/skippy-gradle/src/main/java/io/skippy/gradle/SkippyPlugin.java#L34)

### The skippyAnalyze Task  

`skippyAnalyze` executes in two steps:
1. Generation of `.cov` files
2. Generation of the `classes.md5` file

We will discuss each step in the following sections.

#### Step 1: Generation of `.cov` Files

The `skippyAnalyze` task depends on the `check` lifecycle task to execute all tests in your project:

```
class SkippyAnalyzeTask extends DefaultTask {

    public SkippyAnalyzeTask(...) {
        ...
        dependsOn(..., "check");
        ...
    }
    
}
```

Code: [AnalyzeTask.java](https://github.com/skippy-io/skippy/blob/2c0b7b78adf18edcaa19a397ab74619d76ad1b7e/skippy-gradle/src/main/java/io/skippy/gradle/SkippyAnalyzeTask.java#L49)

`skippyAnalysis` wouldn't be particularly useful if the only thing it did was to run tests. After all, its main purpose
is to perform a Test Impact Analysis. To accomplish this, it sets the `SKIPPY_ANALYZE_MARKER` environment variable for
all test tasks:

```
class SkippyAnalyzeTask extends DefaultTask {

    public SkippyAnalyzeTask(...) {
        ...
        getProject().getTasks().withType(Test.class,
            test -> test.environment(SkippyConstants.SKIPPY_ANALYZE_MARKER, true)
        );
        ...
    }
    
}
```

Code: [AnalyzeTask.java](https://github.com/skippy-io/skippy/blob/2c0b7b78adf18edcaa19a397ab74619d76ad1b7e/skippy-gradle/src/main/java/io/skippy/gradle/SkippyAnalyzeTask.java#L58)

This is a signal for Skippy's JUnit libraries to emit coverages files during the execution of the test suite:

```
import org.jacoco.agent.rt.IAgent;
import org.jacoco.agent.rt.RT;

public final class SkippyTestApi {

    public static void prepareCoverageDataCaptureFor(Class<?> testClass) {
        if (isSkippyCoverageBuild()) {
            ...
            IAgent agent = RT.getAgent();        
            agent.reset();
            ...
        }
    }

    public static void captureCoverageDataFor(Class<?> testClass) {
        if (isSkippyCoverageBuild()) {
            IAgent agent = RT.getAgent();
            byte[] executionData = agent.getExecutionData(true);
            ...
            // write coverage data to file
        }
    }

    private static boolean isSkippyCoverageBuild() {
        return Boolean.valueOf(System.getProperty(SKIPPY_ANALYZE_MARKER)) 
            || Boolean.valueOf(System.getenv().get(SKIPPY_ANALYZE_MARKER));
    }

}
```

Code: [SkippyTestApi.java](https://github.com/skippy-io/skippy/blob/2c0b7b78adf18edcaa19a397ab74619d76ad1b7e/skippy-junit-common/src/main/java/io/skippy/junit/SkippyTestApi.java#L92)

Skippy uses JUnit 5's extension mechanism to invoke 

- `SkippyTestApi#prepareCoverageDataCaptureFor` before and
- `SkippyTestApi#captureCoverageDataFor` after 

the execution of a test class:

```
public final class CoverageFileCallbacks implements TestInstancePreConstructCallback, TestInstancePreDestroyCallback {

    @Override
    public void preConstructTestInstance(TestInstanceFactoryContext factoryContext, ExtensionContext context) {
        context.getTestClass().ifPresent(SkippyTestApi::prepareCoverageDataCaptureFor);
    }

    @Override
    public void preDestroyTestInstance(ExtensionContext context) {
        context.getTestClass().ifPresent(SkippyTestApi::captureCoverageDataFor);
    }

}
```

Code: [CoverageFileCallbacks.java](https://github.com/skippy-io/skippy/blob/2c0b7b78adf18edcaa19a397ab74619d76ad1b7e/skippy-junit5/src/main/java/io/skippy/junit5/CoverageFileCallbacks.java#L33)

As a result, `skippyAnalyze` generates `.cov` files that contains coverage data for each skippified test (more on that
soon):
```
ls -l skippy

com.example.LeftPadderTest.cov
com.example.RightPadderTest.cov
...
```

Each `.cov` file contains the list of classes that are covered by the corresponding test. 

Example:
```
cat com.example.LeftPadderTest.cov
 
com.example.LeftPadder
com.example.LeftPadderTest
com.example.StringUtils
```

#### Step 2: Generation Of The `classes.md5` File

The `skippyAnalyze` task also creates a hash for each class file in one of the build's output folders. Those hashes
are stored in the `classes.md5` file in the `skippy` folder.

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

Code: [ClassesMd5Writer.java](https://github.com/skippy-io/skippy/blob/2c0b7b78adf18edcaa19a397ab74619d76ad1b7e/skippy-build-common/src/main/java/io/skippy/build/ClassesMd5Writer.java#L53) \| [DebugAgnosticHash.java](https://github.com/skippy-io/skippy/blob/2c0b7b78adf18edcaa19a397ab74619d76ad1b7e/skippy-common/src/main/java/io/skippy/core/DebugAgnosticHash.java#L46)

## Predictive Test Selection

Skippy utilizes the data that is generated by Skippy's Test Impact Analysis to predict which tests to run.

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

This meta annotation applies two extensions: `SkippyExecutionCondition` and `SkippyCoverageFileGenerator`.

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@ExtendWith(SkippyExecutionCondition.class)
@ExtendWith(SkippyCoverageFileGenerator.class)
public @interface Skippified {
}
```

Code: [Skippyfied.java](https://github.com/skippy-io/skippy/blob/2c0b7b78adf18edcaa19a397ab74619d76ad1b7e/skippy-junit5/src/main/java/io/skippy/junit5/Skippified.java#L53)


The `SkippyExecutionCondition` extension is used at runtime to make a skip-or-execute decision:

```
final class SkippyExecutionCondition implements ExecutionCondition {

public final class SkipOrExecuteCondition implements ExecutionCondition {

    private final SkippyTestApi skippyTestApi;

    ...

    @Override
    public ConditionEvaluationResult evaluateExecutionCondition(ExtensionContext context) {
        if (context.getTestInstance().isEmpty()) {
            return ConditionEvaluationResult.enabled("");
        }
        if (skippyTestApi.testNeedsToBeExecuted(context.getTestClass().get())) {
            return ConditionEvaluationResult.enabled("");
        }
        return ConditionEvaluationResult.disabled("");
    }

}
```

Code: [SkipOrExecuteCondition.java](https://github.com/skippy-io/skippy/blob/2c0b7b78adf18edcaa19a397ab74619d76ad1b7e/skippy-junit5/src/main/java/io/skippy/junit5/SkipOrExecuteCondition.java#L29)

`SkippyExecutionCondition` internally delegates the skip-or-execute decision to an instance of `SkippyTestApi`.
`SkippyTestApi` passes the call through to an instance of `SkippyAnalysis`, which is the programmatic representation
of Skippy's Test Impact Analysis:

```
class SkippyAnalysis {

    ...

    DecisionWithReason decide(FullyQualifiedClassName testFqn) {
        if (coverageData.noDataAvailableFor(testFqn)) {
            return DecisionWithReason.executeTest(NO_COVERAGE_DATA_FOR_TEST);
        }
        if (hashedClasses.noDataFor(testFqn)) {
            return DecisionWithReason.executeTest(NO_HASH_FOR_TEST);
        }
        if (hashedClasses.hasChanged(testFqn)) {
            return DecisionWithReason.executeTest(BYTECODE_CHANGE_IN_TEST);
        }
        return decideBasedOnCoveredClasses(testFqn);
    }

    private DecisionWithReason decideBasedOnCoveredClasses(FullyQualifiedClassName testFqn) {
       for (var coveredClassFqn : coverageData.getCoveredClasses(testFqn)) {
            if (hashedClasses.hasChanged(coveredClassFqn)) {
                return DecisionWithReason.executeTest(BYTECODE_CHANGE_IN_COVERED_CLASS);
            }
            if (hashedClasses.noDataFor(coveredClassFqn)) {
                return DecisionWithReason.executeTest(NO_HASH_FOR_COVERED_CLASS);
            }
        }
        return DecisionWithReason.skipTest(Reason.NO_CHANGE);
    }

}
```

Code: [SkippyAnalysis.java](https://github.com/skippy-io/skippy/blob/2c0b7b78adf18edcaa19a397ab74619d76ad1b7e/skippy-junit-common/src/main/java/io/skippy/junit/SkippyAnalysis.java#L93)

You might ask: Why the complex indirections, like the `SkippyExecutionCondition -> SkippyTestApi -> SkippyAnalysis` call chain? There are two key reasons:

- It allows to support various build tools (like Gradle & Maven) and unit testing frameworks (such as JUnit 4 & JUnit 5) while keeping the maintenance costs in check.
- It ensures information hiding between subprojects. For instance, `SkippyExecutionCondition` only knows about `SkippyTestApi`, not any other classes in `skippy-junit-common`.

And voila - that's how Skippy works.