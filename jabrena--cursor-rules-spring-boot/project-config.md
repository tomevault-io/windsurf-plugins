---
trigger: always_on
description: generateSystemHealthReport();
---

# Spring Boot Core

Spring Boot Core guidelines focus on proper usage of main annotations, bean management, and configuration best practices to build maintainable and efficient Spring Boot applications.

## Implementing These Principles

These guidelines are built upon the following core principles:

- Principle 1: Use appropriate Spring annotations to clearly express component responsibilities
- Principle 2: Leverage Spring's dependency injection and IoC container effectively
- Principle 3: Follow configuration best practices for maintainable and testable applications
- Principle 4: Apply proper bean lifecycle management and scoping

## Table of contents

- Rule 0: Spring Boot Main Application Class
- Rule 1: Main Spring Boot Annotations Usage
- Rule 2: Bean Definition and Management
- Rule 3: Configuration Classes and Properties
- Rule 4: Component Scanning and Package Organization
- Rule 5: Conditional Configuration and Profiles
- Rule 6: Constructor Dependency Injection Best Practices
- Rule 7: Bean Minimization and Composition
- Rule 8: Scheduled Tasks and Background Processing

## Rule 0: Spring Boot Main Application Class

Title: Create a Proper Spring Boot Main Application Class
Description: Every Spring Boot application should have a main application class annotated with @SpringBootApplication. This class serves as the entry point and configuration root, combining @Configuration, @EnableAutoConfiguration, and @ComponentScan annotations.

**Good example:**

```java
@SpringBootApplication
public class MainApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class, args);
    }
}

// For more complex scenarios with custom configuration
@SpringBootApplication(
    scanBasePackages = {
        "com.company.app.controller",
        "com.company.app.service", 
        "com.company.app.repository",
        "com.company.app.config"
    },
    exclude = {
        DataSourceAutoConfiguration.class,
        SecurityAutoConfiguration.class
    }
)
```

**Bad Example:**

```java
// Missing @SpringBootApplication annotation
public class MainApplication {
    public static void main(String[] args) {
        // Manual Spring context setup instead of SpringApplication.run()
        ApplicationContext context = new AnnotationConfigApplicationContext();
        // Manual configuration - loses Spring Boot benefits
    }
}

// Using individual annotations instead of @SpringBootApplication
@Configuration
@EnableAutoConfiguration  
@ComponentScan
public class MainApplication { // Verbose and error-prone
    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class, args);
    }
}

// Poor naming and structure
@SpringBootApplication
public class App { // Non-descriptive name
    
    @Autowired
    private UserService userService; // Business logic in main class
    
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
        
        // Business logic in main method - should be in separate components
        System.out.println("Processing users...");
    }
}
```

## Rule 1: Main Spring Boot Annotations Usage

Title: Use Appropriate Spring Boot Annotations for Component Definition
Description: Use the correct Spring Boot annotations to define components, controllers, services, and repositories. Each annotation has specific semantics and should be used according to the layer's responsibility.

**Good example:**

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        return ResponseEntity.ok(userService.findById(id));
    }
}

@Service
@Transactional
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    public User findById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
    }
}

@Repository
public interface UserRepository extends CrudRepository<User, Long> {
    
    @Query("SELECT * FROM users WHERE email = :email")
    Optional<User> findByEmail(@Param("email") String email);
    
    @Modifying
    @Query("UPDATE users SET last_login = :lastLogin WHERE id = :id")
    void updateLastLogin(@Param("id") Long id, @Param("lastLogin") LocalDateTime lastLogin);
}

@Table("users")
public class User {
    
    @Id
    private Long id;
    
    @Column("email")
    private String email;
    
    @Column("first_name")
    private String firstName;
    
    @Column("last_name") 
    private String lastName;
    
    @Column("last_login")
    private LocalDateTime lastLogin;
    
    // Constructors, getters, and setters
}
```

**Bad Example:**

```java
@Component // Should be @RestController
public class UserController {
    
    @Inject // Use @Autowired for Spring Boot
    private UserService userService;
}

@Component // Should be @Service
public class UserService {
    // Missing @Transactional for data operations
}

@Component // Should be @Repository
public class UserRepository {
    // Manual JDBC instead of using Spring Data JDBC
}
```

## Rule 2: Bean Definition and Management


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jabrena/cursor-rules-spring-boot](https://github.com/jabrena/cursor-rules-spring-boot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
