### src/main/java

The main source set contains three classes:

```
com
└─ example
   ├─ LeftPadder.java
   ├─ RightPadder.java
   └─ StringUtils.java
```

`StringUtils` is a utility class that provides methods for padding strings:
```java
class StringUtils {

    static String padLeft(String input, int size) {
        // method logic
    }

    static String padRight(String input, int size) {
        // method logic
    }
}
```

`LeftPadder` and `RightPadder` utilize `StringUtils` for their functionality:
```java
class LeftPadder {

    static String padLeft(String input, int size) {
        return StringUtils.padLeft(input, size);
    }

}
```
```java
class RightPadder {

    static String padRight(String input, int size) {
        return StringUtils.padRight(input, size);
    }

}
```

### src/test/java

The test source set contains three tests and one class that stores constants:
```
com
└─ example
   ├─ LeftPadderTest.java
   ├─ RightPadderTest.java
   ├─ StringUtilsTest.java
   └─ TestConstants.java
```

`LeftPadderTest` and `RightPadderTest` are unit tests for their respective classes:

```java
import io.skippy.junit5.Skippified;

@Skippified
public class LeftPadderTest {

    @Test
    void testPadLeft() {
        var input = TestConstants.HELLO;
        assertEquals(" hello", LeftPadder.padLeft(input, 6));
    }

}
```
Note: We will refer to tests that are annotated with `@Skippified` as skippified tests.

`StringUtilsTest` tests the `StringUtil` class and is a standard (e.g., non-skippified) JUnit test:
```java
public class StringUtilsTest {

    @Test
    void testPadLeft() {
        var input = TestConstants.HELLO;
        assertEquals(" hello", StringUtils.padLeft(input, 6));
    }

    @Test
    void testPadRight() {
        var input = TestConstants.HELLO;
        assertEquals("hello ", StringUtils.padRight(input, 6));
    }

}
```

`TestConstants` declares a string constant:
```java
class TestConstants {
    static final String HELLO = "hello";
}
```

## Run The Tests

Run the tests:

{% if page.permalink == "/tutorials/skippy-gradle-junit5" %}
```
./gradlew test --rerun-tasks
```
{% else %}
```
mvn test
```
{% endif %}

Output:

{% if page.permalink == "/tutorials/skippy-gradle-junit5" %}
```
LeftPadderTest > testPadLeft() PASSED
RightPadderTest > testPadRight() PASSED
StringUtilsTest > testPadLeft() PASSED
StringUtilsTest > testPadRight() PASSED
```
{% else %}
```
[INFO] Running com.example.LeftPadderTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] Running com.example.RightPadderTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] Running com.example.StringUtilsTest
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0
```
{% endif %}

Let's inspecSkippy's decision (`skippy/decision.log`):

```
com.example.LeftPadderTest:EXECUTE:NO_COVERAGE_DATA_FOR_TEST
com.example.RightPadderTest:EXECUTE:NO_COVERAGE_DATA_FOR_TEST
```

Skippy did not find data in the skippy folder to make a skip-or-execute decision for `LeftPadderTest` and
`RightPadderTest`. In this case, Skippy will always execute skippified tests. Also note that there is no log entry for
`StringUtilsTest`: It's a non-skippified test. We will omit the output for non-skippified tests during the remainder of
the tutorial.

## Perform A Skippy Analysis

Perform a Skippy analysis:

{% if page.permalink == "/tutorials/skippy-gradle-junit5" %}
```
./gradlew skippyAnalyze
```
{% else %}
```
mvn test -DskippyAnalyze=true
```
{% endif %}

The Skippy Analysis creates a bunch of files in the `skippy` folder:

```
classes.md5
com.example.LeftPadderTest.cov
com.example.RightPadderTest.cov
decisions.log 
```

__Note__: You can skip to the next section if you don't care about how Skippy works under the hood.

Let's take a look at `com.example.LeftPadderTest.cov`:
```
com.example.LeftPadder
com.example.LeftPadderTest
com.example.StringUtils
```

The `.cov` file contains a list of classes that are covered by `com.example.LeftPadderTest`. This data was captured
from [JaCoCo](https://www.jacoco.org/) during the execution of the test suite. You might wonder: Shouldn't there be
coverage for `com.example.TestConstants`? Yes. But: JaCoCo's analysis is based on the execution of instrumented
bytecode. Since the Java compiler inlines the value of `TestConstants.HELLO` into`LeftPadderTest`'s class file, JaCoCo
has no way to detect this.

Don't worry - Skippy has you covered! Skippy combines JaCoCo's dynamic bytecode analysis with a custom, static bytecode
analysis to detect relevant changes. To do this, it needs additional information that is stored in  `classes.md5`:

```
build/classes/java/main:com/example/LeftPadder.class:9U3+WYit7uiiNqA9jplN2A==
build/classes/java/main:com/example/RightPadder.class:ZT0GoiWG8Az5TevH9/JwBg==
build/classes/java/main:com/example/StringUtils.class:4VP9fWGFUJHKIBG47OXZTQ==
build/classes/java/test:com/example/LeftPadderTest.class:sGLJTZJw4beE9m2Kg6chUg==
build/classes/java/test:com/example/RightPadderTest.class:E/ObvuQTODFFqU6gxjbxTQ==
build/classes/java/test:com/example/StringUtilsTest.class:p+N8biKVOm6BltcZkKcC/g==
build/classes/java/test:com/example/TestConstants.class:3qNbG+sSd1S1OGe0EZ9GPA==
```
The file contains hashes for all classes in the project.

Now, let's see what Skippy can do with this data.

## Re-Run The Tests

Re-run the tests:

{% if page.permalink == "/tutorials/skippy-gradle-junit5" %}
```
./gradlew test --rerun-tasks
```
{% else %}
```
mvn test
```
{% endif %}

Output:

{% if page.permalink == "/tutorials/skippy-gradle-junit5" %}
```
LeftPadderTest > testPadLeft() SKIPPED
RightPadderTest > testPadRight() SKIPPED
```
{% else %}
```
[INFO] Running com.example.LeftPadderTest
[WARNING] Tests run: 1, Failures: 0, Errors: 0, Skipped: 1

[INFO] Running com.example.RightPadderTest
[WARNING] Tests run: 1, Failures: 0, Errors: 0, Skipped: 1
```
{% endif %}

Content of `skippy/decision.log`:

```
com.example.LeftPadderTest:SKIP:NO_CHANGE
com.example.RightPadderTest:SKIP:NO_CHANGE
```

Skippy compares the current state of the project with the analysis in the `skippy` folder. It detects that both
`LeftPadderTest` and `RightPadderTest` can be skipped:

- There was no change in either `LeftPadderTest` or `RightPadderTest`.
- There was no change in any of the covered classes.

Hence, the `SKIP` decision in `skippy/decision.log`.

## Testing After Modifications

When changes are made, Skippy reassesses which tests to run based on it's bytecode based change detection. Reasoning
based on the bytecode is powerful: It allows Skippy to distinguish relevant changes (e.g., new or updated instructions)
from irrelevant ones (e.g., a change in a
[LineNumberTable attribute](https://docs.oracle.com/javase/specs/jvms/se17/html/jvms-4.html#jvms-4.7.12) due to the
addition of a line break somewhere in the source file).

Let's perform some experiments.

### Experiment 1

Add a comment to `StringUtils`:

```java
/**
 * New class comment.
 */
class StringUtils {
        
    ...
}
```

Re-run the tests:

{% if page.permalink == "/tutorials/skippy-gradle-junit5" %}
```
./gradlew test --rerun-tasks
```
{% else %}
```
mvn test
```
{% endif %}

Skippy detects that the newly added comment can not break any of the existing tests. `LeftPadderTest` and
`RightPadderTest` will be skipped:

{% if page.permalink == "/tutorials/skippy-gradle-junit5" %}
```
LeftPadderTest > testPadLeft() SKIPPED
RightPadderTest > testPadRight() SKIPPED
```
{% else %}
```
[INFO] Running com.example.LeftPadderTest
[WARNING] Tests run: 1, Failures: 0, Errors: 0, Skipped: 1

[INFO] Running com.example.RightPadderTest
[WARNING] Tests run: 1, Failures: 0, Errors: 0, Skipped: 1
```
{% endif %}

Content of `skippy/decision.log`:

```
com.example.LeftPadderTest:SKIP:NO_CHANGE
com.example.RightPadderTest:SKIP:NO_CHANGE
```

### Experiment 2

Undo the changes from the previous experiment:
```
git stash
```

Comment out the first three lines of `StringUtils#padLeft`:

```java
class StringUtils {
    
    static String padLeft(String input, int size) {
//        if (input.length() < size) {
//            return padLeft(" " + input, size);
//        }
        return input;
    }

    static String padRight(String input, int size) {
        if (input.length() < size) {
            return padRight(input + " ", size);
        }
        return input;
    }
    
}
```

Re-run the tests:

{% if page.permalink == "/tutorials/skippy-gradle-junit5" %}
```
./gradlew test --rerun-tasks
```
{% else %}
```
mvn test
```
{% endif %}

Skippy detects the change and runs the skippified tests again:

{% if page.permalink == "/tutorials/skippy-gradle-junit5" %}
```
LeftPadderTest > testPadLeft() FAILED
    org.opentest4j.AssertionFailedError: expected: < hello> but was: <hello>

RightPadderTest > testPadRight() PASSED
```
{% else %}
```
[INFO] Running com.example.LeftPadderTest
[ERROR] com.example.LeftPadderTest.testPadLeft <<< FAILURE!
org.opentest4j.AssertionFailedError: expected: < hello> but was: <hello>       

[INFO] Running com.example.RightPadderTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
```
{% endif %}

Content of `skippy/decision.log`:

```
com.example.LeftPadderTest:EXECUTE:BYTECODE_CHANGE_IN_COVERED_CLASS
com.example.RightPadderTest:EXECUTE:BYTECODE_CHANGE_IN_COVERED_CLASS
```

At this point in time, Skippy executes a test if the covered class contains a significant bytecode change
(e.g., new or updated instructions). The test itself may or may not depend on this change. In the above example,
`RightPadderTest` could be skipped as well.

While we plan to implement more granular change detection in the future, we currently apply the 80/20 rule: Our focus
is robustness and simplicity while providing a significant reduction in useless testing for applications that contain
large quantities of source files and tests.

### Experiment 3

Undo the changes from the previous experiment:
```
git stash
```

Now, let's see what happens if you change the expected value in `LeftPadderTest` from
`" hello"` to `" HELLO"`:
```java
@Skippified
public class LeftPadderTest {

    @Test
    void testPadLeft() {
        var input = TestConstants.HELLO;
        // assertEquals(" hello", LeftPadder.padLeft(input, 6));
        assertEquals(" HELLO", LeftPadder.padLeft(input, 6));
    }

}
```

Re-run the tests:

{% if page.permalink == "/tutorials/skippy-gradle-junit5" %}
```
./gradlew test --rerun-tasks
```
{% else %}
```
mvn test
```
{% endif %}

Skippy detects the change and runs `LeftPadderTest` again:

{% if page.permalink == "/tutorials/skippy-gradle-junit5" %}
```
LeftPadderTest > testPadLeft() FAILED
    org.opentest4j.AssertionFailedError: expected: < HELLO> but was: < hello>
    
RightPadderTest > testPadRight() SKIPPED
```
{% else %}
```
[INFO] Running com.example.LeftPadderTest
[ERROR] com.example.LeftPadderTest.testPadLeft <<< FAILURE!
org.opentest4j.AssertionFailedError: expected: < HELLO> but was: < hello>

[INFO] Running com.example.RightPadderTest
[WARNING] Tests run: 1, Failures: 0, Errors: 0, Skipped: 1
```
{% endif %}

Content of `skippy/decision.log`:

```
com.example.LeftPadderTest:EXECUTE:BYTECODE_CHANGE_IN_TEST
com.example.RightPadderTest:SKIP:NO_CHANGE
```

### Experiment 4

Undo the changes from the previous experiment:
```
git stash
```

Lastly, let's see what happens if you change the value of the constant in `TestConstants`:

```java
class TestConstants {

    // static final String HELLO = "hello";
    static final String HELLO = "bonjour";

}
```

Re-run the tests:

{% if page.permalink == "/tutorials/skippy-gradle-junit5" %}
```
./gradlew test --rerun-tasks
```
{% else %}
```
mvn test
```
{% endif %}

Skippy detects the change and runs both skippified tests:

{% if page.permalink == "/tutorials/skippy-gradle-junit5" %}
```
LeftPadderTest > testPadLeft() FAILED
    org.opentest4j.AssertionFailedError: expected: < hello> but was: <bonjour>
       
RightPadderTest > testPadRight() FAILED
    org.opentest4j.AssertionFailedError: expected: <hello > but was: <bonjour>
```
{% else %}
```
[INFO] Running com.example.LeftPadderTest
[ERROR] com.example.LeftPadderTest.testPadLeft <<< FAILURE!
org.opentest4j.AssertionFailedError: expected: < hello> but was: <bonjour>

[INFO] Running com.example.RightPadderTest
[ERROR] com.example.RightPadderTest.testPadRight <<< FAILURE!
org.opentest4j.AssertionFailedError: expected: <hello > but was: <bonjour>
```
{% endif %}

Content of `skippy/decision.log`:

```
com.example.LeftPadderTest:EXECUTE:BYTECODE_CHANGE_IN_TEST
com.example.RightPadderTest:EXECUTE:BYTECODE_CHANGE_IN_TEST
```

Congratulations! You've successfully integrated Skippy into your project, ensuring that only necessary tests are run,
saving you time and resources.
