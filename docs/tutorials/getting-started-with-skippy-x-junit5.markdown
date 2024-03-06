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

`LeftPadderTest` and `RightPadderTest` are unit tests for their respective classes.
Both tests are annotated with `@PredictWithSkippy` to enables Skippy's predictive test selection:

```java
import io.skippy.junit5.PredictWithSkippy;

@PredictWithSkippy
public class LeftPadderTest {

    @Test
    void testPadLeft() {
        var input = TestConstants.HELLO;
        assertEquals(" hello", LeftPadder.padLeft(input, 6));
    }

}
```

`StringUtilsTest` tests the `StringUtil` class and is a standard JUnit test that does not utilize
Skippy's predictive test selection:
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
./gradlew test --rerun
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

As you would expect, all tests are executed. But there is more than meets the eye: Skippy performs a 
Test Impact Analysis and stores the result as bunch of files in the .skippy folder:

```
ls -l .skippy

predictions.log
test-impact-analysis.json
```

test-impact-analysis.json contains  
- a mapping between tests and the classes they cover and 
- a snapshot of all class files in the project.

predictions.log gives you insights into Skippy's skip-or-execute decisions:

```
cat .skippy/predictions.log

com.example.LeftPadderTest,EXECUTE,NO_DATA_FOUND_FOR_TEST
com.example.RightPadderTest,EXECUTE,NO_DATA_FOUND_FOR_TEST
```

Skippy executed both tests because it did not find prior impact data. Also note that there is no log entry for 
`StringUtilsTest`: It's a normal test that is not "managed" by Skippy. We will omit the output for 
test that don't use Skippy's predictive test selection during 
the remainder of the tutorial.

## Commit the Skippy Folder

Add the .skippy folder to Git: 

```
git add .skippy && git commit -m 'add skippy folder'
```

This allows us to revert back to this state of the repository throughout the rest tutorial. 

## Re-Run The Tests

Re-run the tests:

{% if page.permalink == "/tutorials/skippy-gradle-junit5" %}
```
./gradlew test --rerun
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

Skippy detects that nothing has changed and skips both tests:

```
cat .skippy/predictions.log

com.example.LeftPadderTest,SKIP,NO_CHANGE
com.example.RightPadderTest,SKIP,NO_CHANGE
```

### Testing After Modifications

When changes are made, Skippy reassesses which tests to run based on it's bytecode based change detection. Reasoning
based on the bytecode is powerful: It allows Skippy to distinguish relevant changes (e.g., new or updated instructions)
from irrelevant ones (e.g., a change in a
[LineNumberTable attribute](https://docs.oracle.com/javase/specs/jvms/se17/html/jvms-4.html#jvms-4.7.12) due to the
addition of a line break somewhere in the source file).

Let's perform some experiments.

#### Experiment 1

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
./gradlew test --rerun
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

#### Experiment 2

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
./gradlew test --rerun
```
{% else %}
```
mvn test
```
{% endif %}

Skippy detects the change and runs the tests again:

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

Content of .skippy/predictions.log:

```
com.example.LeftPadderTest,EXECUTE,BYTECODE_CHANGE_IN_COVERED_CLASS,com.example.StringUtils
com.example.RightPadderTest,EXECUTE,BYTECODE_CHANGE_IN_COVERED_CLASS,com.example.StringUtils
```

At this point in time, Skippy executes a test if the covered class contains a significant bytecode change
(e.g., new or updated instructions). The test itself may or may not depend on this change. In the above example,
`RightPadderTest` could be skipped as well.

While I plan to implement more granular change detection in the future, I currently apply the 80/20 rule: My focus
is robustness and simplicity while providing a significant reduction in useless testing for applications that contain
large quantities of source files and tests.

#### Experiment 3

Undo the changes from the previous experiment:
```
git stash
```

Now, let's see what happens if you change the expected value in `LeftPadderTest` from
`" hello"` to `" HELLO"`:
```java
@PredictWithSkippy
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
./gradlew test --rerun
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

Content of .skippy/predictions.log:

```
com.example.LeftPadderTest,EXECUTE,BYTECODE_CHANGE_IN_TEST
com.example.RightPadderTest,SKIP,NO_CHANGE
```

#### Experiment 4

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
./gradlew test --rerun
```
{% else %}
```
mvn test
```
{% endif %}

Skippy detects the change and runs both tests:

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

Content of .skippy/predictions.log:

```
com.example.LeftPadderTest,EXECUTE,BYTECODE_CHANGE_IN_TEST
com.example.RightPadderTest,EXECUTE,BYTECODE_CHANGE_IN_TEST
```

Congratulations - You've completed the tutorial! By now, you should have a solid idea how Skippy works.

{% include_relative comments.markdown %}