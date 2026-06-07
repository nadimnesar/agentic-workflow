# Testing Patterns Reference

Quick reference for common testing patterns with JUnit 5, Mockito, AssertJ, and Spring Boot.
Use alongside `testing` and `test-driven-development` skills.

## Test Structure (Arrange-Act-Assert)

```java
@Test
@DisplayName("creates a user with default status")
void createUser_shouldReturnCreatedUser() {
    // Arrange
    CreateUserRequest request = new CreateUserRequest("john@example.com", "John");

    // Act
    UserDTO result = userService.createUser(request);

    // Assert
    assertThat(result.email()).isEqualTo("john@example.com");
    assertThat(result.status()).isEqualTo(UserStatus.ACTIVE);
}
```

## Test Naming Convention

```java
// MethodName_shouldExpectedBehavior_whenCondition
// Or: MethodName_expectedBehavior_condition
class UserServiceTest {
    @Test
    void createUser_shouldThrowException_whenEmailAlreadyExists() {}
    @Test
    void createUser_shouldTrimWhitespace_whenNameHasLeadingSpaces() {}
}
```

## JUnit 5 Patterns

```java
@ParameterizedTest
@ValueSource(strings = {"", " ", "a@b"})
@DisplayName("rejects invalid email formats")
void createUser_shouldRejectInvalidEmail(String invalidEmail) {
    assertThrows(ValidationException.class,
        () -> userService.createUser(new CreateUserRequest(invalidEmail, "Name")));
}
```

## Mockito Patterns

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    @Mock
    private UserRepository userRepository;
    @InjectMocks
    private UserService userService;

    @Test
    void createUser_shouldSaveToDatabase() {
        CreateUserRequest request = new CreateUserRequest("test@example.com", "Test");
        when(userRepository.save(any(User.class))).thenReturn(new User(1L, ...));

        UserDTO result = userService.createUser(request);

        verify(userRepository).save(argThat(u ->
            u.getEmail().equals("test@example.com")
        ));
    }
}
```

## AssertJ Patterns

```java
// Basic
assertThat(result.getEmail()).isEqualTo("user@example.com");
assertThat(result.getRoles()).containsExactly("USER");

// Collections
assertThat(userList)
    .hasSize(3)
    .extracting(User::getEmail)
    .contains("alice@example.com", "bob@example.com");

// Exceptions
assertThatThrownBy(() -> service.createUser(null))
    .isInstanceOf(ConstraintViolationException.class)
    .hasMessageContaining("email");

// Soft assertions (multiple assertions continue on failure)
SoftAssertions.assertSoftly(softly -> {
    softly.assertThat(result.getEmail()).isEqualTo("test@example.com");
    softly.assertThat(result.getName()).isNotBlank();
});
```

## Spring Boot Test Slices

```java
// Controller slice
@WebMvcTest(UserController.class)
class UserControllerTest {
    @Autowired
    private MockMvc mockMvc;
    @MockitoBean
    private UserService userService;

    @Test
    void searchUsers_shouldReturn200() throws Exception {
        mockMvc.perform(get("/api/v1/users/search")
                .param("name", "john")
                .param("page", "0")
                .param("size", "20"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.content").isArray());
    }
}

// Repository slice
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class UserRepositoryTest {
    @Autowired
    private UserRepository userRepository;
    @Autowired
    private TestRestTemplate restTemplate;  // Testcontainers managed DB
}
```

## Testcontainers

```java
@Testcontainers
@SpringBootTest
class UserIntegrationTest {
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16");

    @Container
    static GenericContainer<?> redis = new GenericContainer<>("redis:7")
        .withExposedPorts(6379);

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.data.redis.host", redis::getHost);
    }

    @Test
    void searchUsers_shouldReturnResultsFromDatabase() {
        // test with real DB
    }
}
```

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|---|---|---|
| Testing implementation details | Breaks on refactor | Test inputs/outputs only |
| Shared mutable state via static fields | Tests pollute each other | Use @BeforeEach, not @BeforeAll |
| @SpringBootTest for everything | Slow test suite (minutes) | Use test slices (@WebMvcTest, @DataJpaTest) |
| Mocking everything including DB | False confidence | Testcontainers for real DB queries |
| Skipping tests to pass CI | Hides real bugs | Fix or delete the test |
| Thread.sleep() in async tests | Flaky, slow | Use Awaitility or CountDownLatch |
