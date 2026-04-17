When writing tests, [AssertJ](https://assertj.github.io/doc/) assertions are used by default. [Here](https://assertj.github.io/doc/#assertj-core-quick-start) is a quick start guide for AssertJ.

## Keep tests clean
* Use Arrange->Act->Assert
* Minimum number of assertions per test (one is sufficient in most unit tests)
* Non-clean tests -> hard to maintain existing or add new tests -> fear of making code changes -> code rots
* **F.I.R.S.T.**:
  * **Fast**, otherwise you won't want to run them frequently
  * **Independent**, otherwise you get cascade of failures which makes diagnosis difficult
  * **Repeatable** in any environment, even without network. Otherwise, you'll always have excuse for failure
  * **Self-Validating** means elimination of need for manual evaluation
  * **Timely** means that tests should be written *just before* writing production code. Otherwise, you could design code to be non-testable

## Test Driven Development Laws
1. You may not write production code until you have written failing unit test.
2. You may not write more of a unit test than is sufficient to fail, and not compiling is failing.
3. You may not write more production code than is sufficient to pass the currently failing test.

## Unit testing
Unit tests are used for lowest-level testing. They generally focus on methods and small scope and minimum number of dependencies is what makes them fast. When possible, around 60% of test cases should be unit tests.

### Best practices:
* Aim for each unit test method to perform exactly one assertion because it's easier to determine what went wrong and you are sure that all assertions are performed, which is not the case when exception happens
* Use the strongest possible assertions to ensure that production code works correctly
* Put assertion parameters in the proper order: first expected, then actual. Example: ```assertEquals(expected, actual)```. This ensures that JUnit's messages are accurate.
* Name unit tests using a convention that includes the method and condition being tested
* Ensure that test code is separated from production code
* Do not print anything out in unit tests except for debugging purposes
* Try to write tests that force exceptions, and then add behaviour to your handler to satisfy your tests. This will cause you to build the transaction scope of the try block first and will help you maintain the transaction nature of that scope.
* Do not initialize in a unit test class constructor; use an `@Before` method instead:
```java
// Don't do this!
public final class TestFoo {
    private FooDependency fooDependency = new FooDependency();

  // rest of test class removed for brevity
}
```

```java
// Do this instead
public final class TestFoo {
    private FooDependency fooDependency;

    @Before
    public void setUp() {
        fooDependency = new FooDependency();
    }
}
```
* Do not catch checked exceptions and then manually fail a test. A good practice is to declare that a method can throw an exception:
```java
@Test
public void foo_seven() throws Exception {
    assertEquals(3, new Foo().foo(7));
}
```
* In the previous case, a general exception is matched because we don't want our tests to be brittle

* Do not write your own catch blocks that exist only to pass a test. Use `@Test(expected = ExpectedException.class)` instead:
```java
@Test(expected = IOException.class)
public void foo_nine() throws Exception {
  new Foo().foo(9);
}
```
* Do not use `Thread.sleep` in unit tests, use the `Awaitility` library for time-based and/or asynchronous test scenarios

### Modern Unit Testing Practices

* Use JUnit 5 (Jupiter)
  ```java
  @Test
  void testSomething() {
      // ...
  }
  ```

* Use parameterized tests
  ```java
  @ParameterizedTest
  @ValueSource(strings = {"Hello", "World"})
  void testWithParameters(String input) {
      // ...
  }
  ```

* Use dynamic tests
  ```java
  @TestFactory
  Stream<DynamicTest> dynamicTests() {
      return Stream.of("Hello", "World")
          .map(text -> dynamicTest("Testing " + text, () -> {
              // test logic
          }));
  }
  ```

* Use test fixtures with `@BeforeEach` and `@AfterEach`
  ```java
  @BeforeEach
  void setUp() {
      // setup code
  }
  ```

* Use nested tests for better organization
  ```java
  @Nested
  class WhenSomething {
      @Test
      void shouldDoSomething() {
          // ...
      }
  }
  ```

* Use AssertJ for fluent assertions
  ```java
  assertThat(result)
      .isNotNull()
      .hasSize(2)
      .containsExactly("Hello", "World");
  ```

* Use Behavior Driven Development (BDD) style of writing tests
  ```java
  import static org.mockito.BDDMockito.*;

  @Mock
  Seller seller;

  @InjectMocks
  Shop shop;

  void shouldBuyBread() throws Exception {
    // given
    given(seller.askForBread()).willReturn(new Bread());

    // when
    Goods goods = shop.buyBread();

    // then
    assertThat(goods, containBread());
  }
  ```

## Integration testing
Integration tests are used to verify interactions between multiple classes. They are slower than unit tests and they make around 30% of total number of tests. Validating responses from APIs or Web applications is the most common example of integration testing. Next sub-chapters will describe use of REST-assured for that purpose.

### About REST-assured
A great tool for validating responses received from REST service is [REST-assured](http://rest-assured.io/). Take a look at some some simple functionality-describing examples
[here](https://github.com/rest-assured/rest-assured/wiki/Usage#examples). REST-assured has validation support for [Json Schema](https://github.com/rest-assured/rest-assured/wiki/Usage#json-schema-validation),
[XSD Schema](https://github.com/rest-assured/rest-assured/wiki/Usage#example-2---xml), DTD and many more.

### Modern Integration Testing Practices

* Use TestContainers for testing with real dependencies
  ```java
  @Testcontainers
  class IntegrationTest {
      @Container
      static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:latest");
  }
  ```
  Resources:
  - https://www.docker.com/blog/testcontainers-best-practices/
  - https://testcontainers.com/guides/testcontainers-container-lifecycle/
  - https://maciejwalkowiak.com/blog/testcontainers-spring-boot-setup/


* Use Spring Boot Test for testing Spring applications
  ```java
  @SpringBootTest
  @AutoConfigureMockMvc
  class ControllerTest {
      @Autowired
      private MockMvc mockMvc;
  }
  ```

## *Learning tests*
*Learning tests* are used to get familiar with third-party libraries. It is a quick way for a developer to check his understanding of API usage and can be very useful to detect future changes. When those tests start to fail after some time, it means that something has changed in the API, so the developer has to update his code to conform with the changes, or stick with an older version of the API, if possible.

## Mocking

### Mockito
[Mockito](http://site.mockito.org/) is a mocking framework for unit tests in Java. A short intro, some test examples and reference to documentation can be found [here](http://site.mockito.org/), and motivation behind it and it's features [here](https://github.com/mockito/mockito/wiki/Features-And-Motivations). Mockito is included in Spring Boot as a dependency by default. In case you don't use Spring Boot, here is an example for adding it to project build using Gradle:
```gradle
repositories { jcenter() }
dependencies { testCompile "org.mockito:mockito-core:2.+" }
```
Now interaction can be verified:
```java
import static org.mockito.Mockito.*;

// mock creation
List mockedList = mock(List.class);

// using a mock object - it does not throw any "unexpected interaction" exception
mockedList.add("one");
mockedList.clear();

// selective, explicit, highly readable verification
verify(mockedList).add("one");
verify(mockedList).clear();
```
And method calls can be stubbed this way:
```java
// you can mock concrete classes, not only interfaces
LinkedList mockedList = mock(LinkedList.class);

// stubbing appears before the actual execution
when(mockedList.get(0)).thenReturn("first");

// the following prints "first"
System.out.println(mockedList.get(0));

// the following prints "null" because get(999) was not stubbed
System.out.println(mockedList.get(999));
```

Use Mockito annotations:
```java
@ExtendWith(MockitoExtension.class)
class TestClass {
    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService;
}
```

Use argument matchers:
```java
when(userRepository.findById(any())).thenReturn(user);
```

Use argument captors:
```java
@Captor
private ArgumentCaptor<User> userCaptor;

@Test
void shouldSaveUserWithCorrectData() {
    // Given
    User userToSave = new User("John", "john@example.com");

    // When
    userService.saveUser(userToSave);

    // Then
    verify(userRepository).save(userCaptor.capture());
    User savedUser = userCaptor.getValue();

    assertThat(savedUser.getName()).isEqualTo("John");
    assertThat(savedUser.getEmail()).isEqualTo("john@example.com");
}
```

Argument captors are particularly useful when:
* You need to verify complex objects passed to mocked methods
* You want to verify multiple calls to the same method with different arguments
* You need to verify the state of objects that were modified by the method under test

Example with multiple captures:
```java
@Test
void shouldSaveMultipleUsers() {
    // Given
    User user1 = new User("John", "john@example.com");
    User user2 = new User("Jane", "jane@example.com");

    // When
    userService.saveUser(user1);
    userService.saveUser(user2);

    // Then
    verify(userRepository, times(2)).save(userCaptor.capture());
    List<User> savedUsers = userCaptor.getAllValues();

    assertThat(savedUsers).hasSize(2);
    assertThat(savedUsers.get(0).getName()).isEqualTo("John");
    assertThat(savedUsers.get(1).getName()).isEqualTo("Jane");
}
```

### Wiremock
[WireMock](https://wiremock.org/) is a simulator for HTTP-based APIs. Some might consider it a service virtualization
tool or a mock server. Wiremock is widely used in unit and integration testing. It can be run from [JUnit 4.x Rule](http://wiremock.org/docs/junit-rule/),
[Java](http://wiremock.org/docs/java-usage/) and as a
[Standalone Process](http://wiremock.org/docs/running-standalone/).
It supports [Stubbing](http://wiremock.org/docs/stubbing/),
[Veryfing](http://wiremock.org/docs/verifying/),
[Request Matching](http://wiremock.org/docs/request-matching/),
[Response Templating](http://wiremock.org/docs/response-templating/) etc.
Visit the [WireMock docs](http://wiremock.org/docs/) for more information about WireMock.

### Localstack
[Localstack](https://localstack.cloud/) provides an easy-to-use test/mocking framework for developing cloud applications.

#### Key Features
- Supports numerous AWS services locally including S3, SQS, SNS, Lambda, DynamoDB, etc.
- Compatible with AWS SDKs
- Can be integrated with TestContainers for automated testing

#### Using Localstack with TestContainers

```java
@Testcontainers
class AwsServiceTest {
    @Container
    static LocalStackContainer localstack = new LocalStackContainer(DockerImageName.parse("localstack/localstack:latest"))
            .withServices(LocalStackContainer.Service.S3, LocalStackContainer.Service.SQS);

    private AmazonS3 s3;
    private AmazonSQS sqs;

    @BeforeEach
    void setUp() {
        s3 = AmazonS3ClientBuilder
                .standard()
                .withEndpointConfiguration(localstack.getEndpointConfiguration(LocalStackContainer.Service.S3))
                .withCredentials(localstack.getDefaultCredentialsProvider())
                .build();

        sqs = AmazonSQSClientBuilder
                .standard()
                .withEndpointConfiguration(localstack.getEndpointConfiguration(LocalStackContainer.Service.SQS))
                .withCredentials(localstack.getDefaultCredentialsProvider())
                .build();
    }

    @Test
    void testS3Operations() {
        String bucketName = "test-bucket";
        s3.createBucket(bucketName);

        assertThat(s3.listBuckets())
            .extracting(Bucket::getName)
            .contains(bucketName);
    }
}
```

#### Best Practices for Localstack Testing

- **Integration with Spring Boot**
   ```java
   @SpringBootTest
   @Testcontainers
   class SpringAwsIntegrationTest {
       @Container
       static LocalStackContainer localstack = new LocalStackContainer(DockerImageName.parse("localstack/localstack:latest"))
               .withServices(LocalStackContainer.Service.S3);

       @DynamicPropertySource
       static void overrideProperties(DynamicPropertyRegistry registry) {
           registry.add("aws.endpoint", () -> localstack.getEndpoint().toString());
           registry.add("aws.region", localstack::getRegion);
       }
   }
   ```

- **Error Simulation**
   - Test error scenarios by manipulating network conditions
   - Use Localstack's ability to inject faults and latency

Resources:
- [Localstack Documentation](https://docs.localstack.cloud/overview/)
- [TestContainers Localstack Module](https://www.testcontainers.org/modules/localstack/)
- [AWS SDK with Localstack](https://docs.localstack.cloud/user-guide/aws/feature-coverage/)
