---
layout: page
title: How Skippy Works
permalink: /how-skippy-works/
---
Documentation for Skippy version `0.0.7`.

Skippy improves test efficiency with a two-pronged approach:
- [A Powerful Build Plugin](https://github.com/skippy-io/skippy/blob/65a168e892cf40a81bf1536bd841b9e0173d08cd/skippy-gradle/src/main/java/io/skippy/gradle/SkippyPlugin.java#L38) and
- [A Smart JUnit Extension](https://github.com/skippy-io/skippy/blob/65a168e892cf40a81bf1536bd841b9e0173d08cd/skippy-junit5/src/main/java/io/skippy/junit5/Skippified.java#L53).

In the following sections, we'll explore how these components interact and function together.

## The Skippy Gradle Plugin

The Skippy Gradle Plugin integrates the `skippyAnalyze` task into your project, which operates in two primary steps:

### Step 1: Generation of `.cov` Files

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

Code: [AnalyzeTask.java](https://github.com/skippy-io/skippy/blob/65a168e892cf40a81bf1536bd841b9e0173d08cd/skippy-gradle/src/main/java/io/skippy/gradle/SkippyAnalyzeTask.java#L57)

`skippyAnalysis` wouldn't be particularly useful if the only thing it does is to run tests: It's main purpose is to 
capture coverage data for each test. To do so, it applies the JaCoCo plugin and sets the `skippyEmitCovFiles` property 
for all test tasks:
```
class SkippyAnalyzeTask extends DefaultTask {

    public SkippyAnalyzeTask(...) {
        ...
        getProject().getPlugins().apply(JacocoPlugin.class);
        getProject().getExtensions().getByType(JacocoPluginExtension.class).setToolVersion(SkippyProperties.getJacocoVersion());
        getProject().getTasks().withType(Test.class, test -> test.environment("skippyEmitCovFiles", true));
        ...
    }
    
}
```

This allows Skippy's JUnit libraries to emit coverages files during the execution of the test suite:

```
import org.jacoco.agent.rt.IAgent;
import org.jacoco.agent.rt.RT;

class SkippyCoverageFileGenerator implements TestInstancePreConstructCallback, TestInstancePreDestroyCallback {

    ...

    @Override
    public void preDestroyTestInstance(ExtensionContext context) {
        if ( ! Boolean.valueOf(System.getenv().get("skippyEmitCovFiles"))) {
            return;
        }
        IAgent agent = RT.getAgent();
        emitCovFile(context, agent);
    }

            
    private static void emitCovFile(ExtensionContext context, IAgent agent) {
        // read execution data from JaCoCo agent and write it to .cov file
        ...                
    }
    
}
```

Code: [SkippyCoverageFileGenerator.java](https://github.com/skippy-io/skippy/blob/65a168e892cf40a81bf1536bd841b9e0173d08cd/skippy-junit5/src/main/java/io/skippy/junit5/SkippyCoverageFileGenerator.java#L44)

The individual coverage files are stored in the `skippy` directory:
```
ls -l skippy

com.example.LeftPadderTest.cov
com.example.RightPadderTest.cov
...
```

Each `.cov` file contains the list of classes that are covered by the corresponding test. 

Example of a `.cov` file:
```
com.example.LeftPadder
com.example.LeftPadderTest
com.example.StringUtils
```

### Step 2: Create A Hash For Each Class File

Skippy also creates a hash for each class file in the build's output folders. These hashes are stored in the 
`classes.md5` file within the skippy folder.

Example `classes.md5` file:
```
build/classes/java/main:com/example/LeftPadder.class:9U3+WYit7uiiNqA9jplN2A==
build/classes/java/main:com/example/RightPadder.class:ZT0GoiWG8Az5TevH9/JwBg==
build/classes/java/main:com/example/StringUtils.class:4VP9fWGFUJHKIBG47OXZTQ==
build/classes/java/test:com/example/LeftPadderTest.class:sGLJTZJw4beE9m2Kg6chUg==
build/classes/java/test:com/example/RightPadderTest.class:E/ObvuQTODFFqU6gxjbxTQ==
build/classes/java/test:com/example/StringUtilsTest.class:p+N8biKVOm6BltcZkKcC/g==
build/classes/java/test:com/example/TestConstants.class:3qNbG+sSd1S1OGe0EZ9GPA==
```

The class files are not hashed as-is. Instead, Skippy uses ASM's bytecode manipulation capabilities to hash a copy of
the original class file that is stripped of all debug information (like [LineNumberTable attributes](https://docs.oracle.com/javase/specs/jvms/se17/html/jvms-4.html#jvms-4.7.12)).

This allows Skippy to treat certain changes like
- change in formatting and indentation,
- updated JavaDocs and
- addition of newlines and linebreaks

as 'no-ops'.

Code: [ClassesMd5Writer.java](https://github.com/skippy-io/skippy/blob/65a168e892cf40a81bf1536bd841b9e0173d08cd/skippy-gradle/src/main/java/io/skippy/gradle/io/ClassesMd5Writer.java#L54)

## Conditional Test Execution: Overview

Skippy utilizes the data from the steps above to make intelligent decisions during test execution.

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

Code: [Skippyfied.java](https://github.com/skippy-io/skippy/blob/65a168e892cf40a81bf1536bd841b9e0173d08cd/skippy-junit5/src/main/java/io/skippy/junit5/Skippified.java#L53)


The `SkippyExecutionCondition` extension is used at runtime to make a skip-or-execute decision:

```
final class SkippyExecutionCondition implements ExecutionCondition {

    private final SkippyAnalysis skippyAnalysis;

    ...

    @Override
    public ConditionEvaluationResult evaluateExecutionCondition(ExtensionContext context) {
        ...
        if (skippyAnalysis.testNeedsToBeExecuted(context.getTestClass().get())) {
            return ConditionEvaluationResult.enabled("");
        }
        return ConditionEvaluationResult.disabled("");
    }

}
```

Code: [SkippyExecutionCondition.java](https://github.com/skippy-io/skippy/blob/65a168e892cf40a81bf1536bd841b9e0173d08cd/skippy-junit5/src/main/java/io/skippy/junit5/SkippyExecutionCondition.java#L46)

`SkippyExecutionCondition` internally delegates the skip-or-execute decision to an instance of `SkippyAnalysis`:

```
public class SkippyAnalysis {

    private final HashedClasses hashedClasses;
    private final CoverageData coverageData;
    
    ...

}
```

Code: [SkippyAnalysis.java](https://github.com/skippy-io/skippy/blob/65a168e892cf40a81bf1536bd841b9e0173d08cd/skippy-core/src/main/java/io/skippy/core/SkippyAnalysis.java#L35)
/ [HashedClasses.java](https://github.com/skippy-io/skippy/blob/65a168e892cf40a81bf1536bd841b9e0173d08cd/skippy-core/src/main/java/io/skippy/core/HashedClasses.java#L35)
/ [CoverageData.java](https://github.com/skippy-io/skippy/blob/65a168e892cf40a81bf1536bd841b9e0173d08cd/skippy-core/src/main/java/io/skippy/core/CoverageData.java#L35)

`SkippyAnalysis` is an in-memory representation of the content of the skippy folder:

- `hashedClasses` is the programmatic representation of the `classes.md5` file
- `coverageData` is the programmatic representation of the `.cov` files

`SkippyAnalysis` uses this data to make a skip-or-execute decision:

```
public class SkippyAnalysis {
   
    ...
    
    public boolean testNeedsToBeExecuted(Class<?> test) {
        return decide(new FullyQualifiedClassName(test.getName())).decision == Decision.EXECUTE_TEST;
    }

    DecisionWithReason decide(FullyQualifiedClassName testFqn) {
        if (coverageData.noDataAvailableFor(testFqn)) {
            return DecisionWithReason.executeTest(NO_COVERAGE_DATA_FOR_TEST);
        }
        if (hashedClasses.noDataFor(testFqn)) {
            return DecisionWithReason.executeTest(NO_HASH_FOR_TEST);
        }
        if (hashedClasses.getChangedClasses().contains(testFqn)) {
            return DecisionWithReason.executeTest(BYTECODE_CHANGE_IN_TEST);
        }
        return decideBasedOnCoveredClasses(testFqn);
    }

    private DecisionWithReason decideBasedOnCoveredClasses(FullyQualifiedClassName testFqn) {
        ...
    }    

}
```

And voila - that's how Skippy works.