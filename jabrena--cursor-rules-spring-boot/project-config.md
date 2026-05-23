---
trigger: always_on
description: Spring Boot slice testing allows you to test specific layers or "slices" of your application in isolation, providing faster and more focused tests than full integration tests. This approach helps maintain test clarity, reduces test execution time, and improves maintainability.
---

# Spring Boot Slice Testing

Spring Boot slice testing allows you to test specific layers or "slices" of your application in isolation, providing faster and more focused tests than full integration tests. This approach helps maintain test clarity, reduces test execution time, and improves maintainability.

## Implementing These Principles

These guidelines are built upon the following core principles:

- **Layer Isolation**: Test each application layer independently without loading the entire Spring context
- **Focused Testing**: Use appropriate slice annotations to load only the components needed for specific functionality
- **Mock Dependencies**: Mock external dependencies and other layers to achieve true unit testing at the slice level
- **Fast Execution**: Minimize Spring context loading to achieve rapid test feedback cycles

## Table of contents

- Rule 1: Use @WebMvcTest for Web Layer Testing
- Rule 2: Use @JdbcTest for Repository Layer Testing
- Rule 3: Use @JsonTest for JSON Serialization Testing
- Rule 4: Use @MockBean for Mocking Dependencies
- Rule 5: Configure Test Profiles Appropriately
- Rule 6: Use @TestConfiguration for Custom Test Setup

## Rule 1: Use @WebMvcTest for Web Layer Testing

Title: Test Controllers in Isolation with @WebMvcTest
Description: Use @WebMvcTest to test only the web layer (controllers) without loading the full application context. This annotation configures Spring MVC infrastructure and auto-configures MockMvc for testing HTTP requests and responses.

**Good example:**

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Test
    void shouldReturnUserWhenValidId() throws Exception {
        // Given
        User user = new User(1L, "John Doe", "john@example.com");
        when(userService.findById(1L)).thenReturn(user);
        
        // When & Then
        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("John Doe"))
            .andExpect(jsonPath("$.email").value("john@example.com"));
    }
}
```

**Bad Example:**

```java
@SpringBootTest
@AutoConfigureTestDatabase
class UserControllerTest {
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Test
    void shouldReturnUser() {
        // This loads the entire application context unnecessarily
        // and requires database setup for a simple controller test
        ResponseEntity<User> response = restTemplate.getForEntity("/api/users/1", User.class);
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
    }
}
```

## Rule 2: Use @JdbcTest for Repository Layer Testing

Title: Test JDBC Repositories with @JdbcTest
Description: Use @JdbcTest to test Spring Data JDBC repositories in isolation. This annotation configures an in-memory database, auto-configures JdbcTemplate and NamedParameterJdbcTemplate, and loads Spring Data JDBC repositories without loading the full application context.

**Good example:**

```java
@JdbcTest
class UserRepositoryTest {

    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void shouldFindUserByEmail() {
        // Given
        User user = new User(null, "John Doe", "john@example.com");
        User saved = userRepository.save(user);
        
        // When
        Optional<User> found = userRepository.findByEmail("john@example.com");
        
        // Then
        assertThat(found).isPresent();
        assertThat(found.get().getName()).isEqualTo("John Doe");
        assertThat(found.get().getEmail()).isEqualTo("john@example.com");
    }
    
    @Test
    void shouldReturnEmptyWhenUserNotFound() {
        // When
        Optional<User> found = userRepository.findByEmail("nonexistent@example.com");
        
        // Then
        assertThat(found).isEmpty();
    }
    
    @Test
    void shouldUseJdbcTemplateForCustomQueries() {
        // Given
        jdbcTemplate.update(
            "INSERT INTO users (name, email) VALUES (?, ?)", 
            "Jane Smith", "jane@example.com"
        );
        
        // When
        Long count = jdbcTemplate.queryForObject(
            "SELECT COUNT(*) FROM users WHERE email LIKE '%@example.com'", 
            Long.class
        );
        
        // Then
        assertThat(count).isEqualTo(1L);
    }
}
```

**Bad Example:**

```java
@SpringBootTest
class UserRepositoryTest {
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void shouldFindUserByEmail() {
        // This loads the entire application context and all beans
        // unnecessarily for a simple repository test
        User user = new User(null, "John Doe", "john@example.com");
        userRepository.save(user);
        
        Optional<User> found = userRepository.findByEmail("john@example.com");
        assertThat(found).isPresent();
    }
}
```

## Rule 3: Use @JsonTest for JSON Serialization Testing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jabrena/cursor-rules-spring-boot](https://github.com/jabrena/cursor-rules-spring-boot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
