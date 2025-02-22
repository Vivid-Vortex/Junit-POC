# Using JSON Files with JUnit Parameterized Tests

## Q: Is there any annotation similar to @Paramaterized from CSV file for taking input from JSON files in JUnit?

A: For reading test parameters from JSON files in JUnit, there isn't a direct built-in annotation like `@ParameterizedTest` with `@CsvFileSource`. However, there are two good approaches:

1. Using JUnit Jupiter with `@ParameterizedTest` and a custom `ArgumentsProvider`:
```java
public class JsonArgumentsProvider implements ArgumentsProvider {
    @Override
    public Stream<? extends Arguments> provideArguments(ExtensionContext context) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        TestData[] testData = mapper.readValue(new File("test-data.json"), TestData[].class);
        return Arrays.stream(testData).map(Arguments::of);
    }
}

@ParameterizedTest
@ArgumentsSource(JsonArgumentsProvider.class)
void testWithJsonData(TestData data) {
    // Your test code
}
```

2. Using `@JsonFileSource` from junit-json-params library:
```java
@ParameterizedTest
@JsonFileSource(resources = "/test-data.json")
void testWithJsonData(TestData data) {
    // Your test code
}
```

## Q: Can you elaborate both approaches with simple examples?

A: Here's a complete example for both approaches using a user registration testing scenario:

```java
// First approach: Custom ArgumentsProvider
import org.junit.jupiter.api.extension.ExtensionContext;
import org.junit.jupiter.params.provider.Arguments;
import org.junit.jupiter.params.provider.ArgumentsProvider;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ArgumentsSource;
import java.io.File;
import java.util.stream.Stream;

// Test data class
class UserTestData {
    private String username;
    private String email;
    private int age;
    private boolean expectedValid;

    // Getters and setters
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }
    public boolean isExpectedValid() { return expectedValid; }
    public void setExpectedValid(boolean expectedValid) { this.expectedValid = expectedValid; }
}

// Custom ArgumentsProvider
class UserJsonArgumentsProvider implements ArgumentsProvider {
    @Override
    public Stream<? extends Arguments> provideArguments(ExtensionContext context) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        UserTestData[] testData = mapper.readValue(
            new File("src/test/resources/user-test-data.json"), 
            UserTestData[].class
        );
        return Arrays.stream(testData).map(Arguments::of);
    }
}

// Test class using custom ArgumentsProvider
class UserRegistrationTest {
    @ParameterizedTest
    @ArgumentsSource(UserJsonArgumentsProvider.class)
    void testUserRegistration(UserTestData testData) {
        UserRegistrationService service = new UserRegistrationService();
        boolean isValid = service.validateUser(
            testData.getUsername(), 
            testData.getEmail(), 
            testData.getAge()
        );
        assertEquals(testData.isExpectedValid(), isValid);
    }
}
```

Example JSON file (user-test-data.json):
```json
[
    {
        "username": "john_doe",
        "email": "john@example.com",
        "age": 25,
        "expectedValid": true
    },
    {
        "username": "jane_smith",
        "email": "invalid-email",
        "age": 17,
        "expectedValid": false
    }
]
```

Second approach using junit-json-params library:

First, add the dependency to your pom.xml:
```xml
<dependency>
    <groupId>org.junit-pioneer</groupId>
    <artifactId>junit-pioneer</artifactId>
    <version>1.9.1</version>
    <scope>test</scope>
</dependency>
```

Then create your test:
```java
import org.junitpioneer.jupiter.json.JsonFileSource;
import org.junit.jupiter.params.ParameterizedTest;

class UserRegistrationJsonFileTest {
    @ParameterizedTest
    @JsonFileSource(resources = "/user-test-data.json")
    void testUserRegistration(UserTestData testData) {
        UserRegistrationService service = new UserRegistrationService();
        boolean isValid = service.validateUser(
            testData.getUsername(), 
            testData.getEmail(), 
            testData.getAge()
        );
        assertEquals(testData.isExpectedValid(), isValid);
    }
}
```

## Q: Can you make the UserJsonArgumentsProvider method generic for handling any number of any types of JSONs?

A: Here's a generic implementation that can handle any type of JSON data:

```java
public class GenericJsonArgumentsProvider<T> implements ArgumentsProvider {
    private final String jsonFilePath;
    private final Class<T> targetClass;

    public GenericJsonArgumentsProvider(String jsonFilePath, Class<T> targetClass) {
        this.jsonFilePath = jsonFilePath;
        this.targetClass = targetClass;
    }

    @Override
    public Stream<? extends Arguments> provideArguments(ExtensionContext context) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        // Create array type of the target class
        T[] testData = mapper.readValue(
            new File(jsonFilePath),
            mapper.getTypeFactory().constructArrayType(targetClass)
        );
        return Arrays.stream(testData).map(Arguments::of);
    }
}

// Custom annotation to make usage easier
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@ArgumentsSource(GenericJsonArgumentsProvider.class)
public @interface JsonFileArgumentsSource {
    String value();  // for file path
    Class<?> type(); // for target class
}
```

Usage examples:
```java
// Example 1: Using provider directly
@ParameterizedTest
@ArgumentsSource(new GenericJsonArgumentsProvider<>(
    "src/test/resources/user-test-data.json",
    UserTestData.class
))
void testWithUserData(UserTestData data) {
    // Test code
}

// Example 2: Using custom annotation
@ParameterizedTest
@JsonFileArgumentsSource(
    value = "src/test/resources/product-data.json",
    type = ProductData.class
)
void testWithProductData(ProductData data) {
    // Test code
}
```

## Q: Can you provide a complete example of using this generic provider?

A: Here's a complete example using the generic JSON provider to test a shopping cart service:

```java
// Business classes
class Product {
    private String id;
    private String name;
    private double price;
    private boolean inStock;
    // Getters and Setters
}

class Order {
    private String orderId;
    private List<String> productIds;
    private String customerEmail;
    private double totalAmount;
    // Getters and Setters
}

// Test data classes
class ProductTestData {
    private Product input;
    private boolean expectedValidation;
    private String expectedErrorMessage;
    // Getters and Setters
}

class OrderTestData {
    private Order input;
    private boolean expectedSuccess;
    private double expectedTotal;
    // Getters and Setters
}

// Service class
class ShoppingCartService {
    public boolean validateProduct(Product product) {
        return product != null &&
               product.getId() != null &&
               product.getName() != null &&
               product.getPrice() > 0;
    }

    public double calculateOrderTotal(Order order) {
        if (order == null || order.getProductIds() == null) {
            throw new IllegalArgumentException("Invalid order");
        }
        return order.getProductIds().size() * 10.0; // Simplified calculation
    }
}

// Test class
@ExtendWith(MockitoExtension.class)
class ShoppingCartServiceTest {
    private ShoppingCartService shoppingCartService;

    @BeforeEach
    void setUp() {
        shoppingCartService = new ShoppingCartService();
    }

    @ParameterizedTest
    @JsonFileArgumentsSource(
        value = "src/test/resources/product-validation-test-data.json",
        type = ProductTestData.class
    )
    void testProductValidation(ProductTestData testData) {
        boolean result = shoppingCartService.validateProduct(testData.getInput());
        assertEquals(testData.isExpectedValidation(), result);
    }

    @ParameterizedTest
    @JsonFileArgumentsSource(
        value = "src/test/resources/order-calculation-test-data.json",
        type = OrderTestData.class
    )
    void testOrderCalculation(OrderTestData testData) {
        if (testData.isExpectedSuccess()) {
            double total = shoppingCartService.calculateOrderTotal(testData.getInput());
            assertEquals(testData.getExpectedTotal(), total, 0.01);
        } else {
            assertThrows(IllegalArgumentException.class, () -> 
                shoppingCartService.calculateOrderTotal(testData.getInput())
            );
        }
    }
}
```

Example JSON test data files:

product-validation-test-data.json:
```json
[
    {
        "input": {
            "id": "P1",
            "name": "Valid Product",
            "price": 29.99,
            "inStock": true
        },
        "expectedValidation": true,
        "expectedErrorMessage": null
    },
    {
        "input": {
            "id": null,
            "name": "Invalid Product",
            "price": 0,
            "inStock": false
        },
        "expectedValidation": false,
        "expectedErrorMessage": "Invalid product data"
    }
]
```

order-calculation-test-data.json:
```json
[
    {
        "input": {
            "orderId": "O1",
            "productIds": ["P1", "P2", "P3"],
            "customerEmail": "test@example.com",
            "totalAmount": 0
        },
        "expectedSuccess": true,
        "expectedTotal": 30.0
    },
    {
        "input": null,
        "expectedSuccess": false,
        "expectedTotal": 0
    }
]
```

This example demonstrates:
1. Multiple test data types
2. Complex object structures
3. Different testing scenarios
4. Both positive and negative test cases
5. Exception testing
6. Clean separation of test data from test logic
