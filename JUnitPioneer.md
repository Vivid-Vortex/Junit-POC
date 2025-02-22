**JUnit Pioneer** is an open-source library that extends the capabilities of JUnit 5, the popular testing framework for Java. It provides additional annotations, utilities, and extensions to make writing tests more convenient and expressive. JUnit Pioneer is designed to complement JUnit 5 by addressing common testing scenarios that are not natively supported or require extra boilerplate code.

Here are some key features and functionalities of JUnit Pioneer:

### 1. **Additional Annotations**
   - **@CartesianTest**: Allows you to create parameterized tests with a Cartesian product of arguments, making it easier to test all combinations of inputs.
   - **@ClearSystemProperty / @SetSystemProperty**: Simplifies testing code that relies on system properties by allowing you to set or clear system properties for the duration of a test.
   - **@ClearEnvironmentVariable / @SetEnvironmentVariable**: Similar to system properties, but for environment variables.
   - **@DefaultLocale / @DefaultTimeZone**: Sets the default locale or time zone for a test, ensuring consistent behavior across different environments.
   - **@DisableIfTestFails**: Automatically disables a test if it fails, which can be useful for flaky tests.

### 2. **Utilities**
   - **Temporary Directory Support**: Provides utilities for creating and managing temporary directories during tests, which is useful for file-based testing.
   - **Exception Testing**: Simplifies testing for exceptions with utilities like `assertThrows` and `assertDoesNotThrow`.

### 3. **Extensions**
   - **Retry Extension**: Allows you to retry failed tests a specified number of times, which can be helpful for dealing with intermittent failures.
   - **Benchmark Extension**: Measures the execution time of tests, helping you identify performance bottlenecks.

### 4. **Integration with JUnit 5**
   - JUnit Pioneer is fully compatible with JUnit 5 and integrates seamlessly with its core features, such as parameterized tests, dynamic tests, and extensions.

### 5. **Community-Driven**
   - JUnit Pioneer is developed and maintained by the community, with contributions from developers who want to improve the testing experience in Java. It is hosted on GitHub and welcomes contributions.

### Example Usage
Here’s an example of using the `@CartesianTest` annotation to test all combinations of inputs:

```java
import org.junitpioneer.jupiter.CartesianTest;
import org.junitpioneer.jupiter.CartesianTest.Values;

import static org.junit.jupiter.api.Assertions.assertEquals;

class MathTests {

    @CartesianTest
    void testAddition(
        @Values(ints = {1, 2, 3}) int a,
        @Values(ints = {4, 5, 6}) int b
    ) {
        assertEquals(a + b, Math.addExact(a, b));
    }
}
```

### Why Use JUnit Pioneer?
- **Reduces Boilerplate**: It eliminates the need for repetitive code in common testing scenarios.
- **Enhances Readability**: The additional annotations and utilities make tests more expressive and easier to understand.
- **Extends JUnit 5**: It fills gaps in JUnit 5’s functionality, making it a powerful companion for writing tests.

### Getting Started
To use JUnit Pioneer, add it as a dependency in your project. For Maven:

```xml
<dependency>
    <groupId>org.junit-pioneer</groupId>
    <artifactId>junit-pioneer</artifactId>
    <version>1.9.1</version> <!-- Check for the latest version -->
    <scope>test</scope>
</dependency>
```

For Gradle:

```groovy
testImplementation 'org.junit-pioneer:junit-pioneer:1.9.1' // Check for the latest version
```

JUnit Pioneer is a valuable tool for Java developers looking to enhance their testing capabilities with JUnit 5. Its focus on simplicity and practicality makes it a great addition to any testing toolkit.
