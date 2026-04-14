---
trigger: always_on
description: miBudget is a personal budgeting web application built with Java Spring Boot that helps users manage their finances by:
---

# GitHub Copilot Instructions for miBudget

## Project Overview

miBudget is a personal budgeting web application built with Java Spring Boot that helps users manage their finances by:
- Connecting to multiple banks and investment accounts via the Plaid API
- Tracking and categorizing transactions automatically
- Creating intelligent budgets that learn from user behavior
- Generating monthly financial reports
- Providing a minimalist interface for budget management

## Technology Stack

- **Backend**: Java 18, Spring Boot 2.7.1
- **Web Framework**: Spring MVC with JSP views
- **Security**: Spring Security
- **Database**: MySQL with JPA/Hibernate
- **ORM**: Hibernate 6.1.1.Final
- **Build Tool**: Maven
- **External APIs**: Plaid API for bank integration
- **Logging**: Log4j2
- **Testing**: JUnit Jupiter

## Project Structure

```
src/
├── main/
│   ├── java/com/miBudget/
│   │   ├── MiBudgetApplication.java  # Spring Boot main application
│   │   ├── controllers/              # REST controllers
│   │   ├── core/                     # Core components (Constants, CorsFilter, SecurityConfig)
│   │   ├── daos/                     # Data Access Objects (JPA repositories)
│   │   ├── entities/                 # JPA entities (User, Budget, Transaction, etc.)
│   │   ├── enums/                    # Enumerations
│   │   ├── processors/               # Business logic processors
│   │   ├── services/                 # Service layer
│   │   ├── servlets/                 # Servlet components
│   │   └── utilities/                # Utility classes
│   ├── resources/
│   │   ├── application.properties    # Spring Boot configuration
│   │   ├── hibernate.cfg.xml         # Hibernate configuration
│   │   └── log4j2.properties         # Logging configuration
│   └── webapp/                       # JSP views and static resources
└── test/java/                        # Test files
```

## Coding Standards

### General Guidelines

1. **Java Version**: Use Java 18 features where appropriate
2. **Code Style**: Follow standard Java conventions
3. **Naming**: Use descriptive, camelCase for variables and methods, PascalCase for classes
4. **Comments**: Add JavaDoc for public methods and classes, especially when the purpose isn't immediately obvious
5. **Error Handling**: Use proper exception handling; log errors using Log4j2

### Entity Classes

- Use JPA annotations: `@Entity`, `@Table`, `@Id`, `@GeneratedValue`
- Include `@Transient` for fields that shouldn't be persisted
- Use `@SequenceGenerator` for ID generation with database sequences
- Provide constructors for different use cases (login, registration, etc.)
- Include getter/setter methods (Lombok `@Data` is available but not always used)

Example pattern:
```java
@Entity
@Table(name="table_name")
public class EntityName {
    @Id
    @SequenceGenerator(name="entityname_sequence", sequenceName="entityname_sequence", allocationSize = 1)
    @GeneratedValue(strategy=GenerationType.SEQUENCE, generator="entityname_sequence")
    private Long id;
    
    // fields, constructors, getters/setters
}
```

Note: Sequence names follow the pattern `{tablename}_sequence` (e.g., `users_sequence`, `budgets_sequence`)

### Controllers

- Use `@RestController` for REST endpoints
- Use `@CrossOrigin(origins = "*")` for CORS support
- Inject DAOs via constructor with `@Autowired`
- Use `@RequestMapping` with explicit path and method
- Use Log4j2 for logging (get logger via `LogManager.getLogger(ClassName.class)`)
- Log method entry with `Constants.start` pattern

Example pattern:
```java
@RestController
@CrossOrigin(origins = "*")
public class ControllerName {
    private static final Logger LOGGER = LogManager.getLogger(ControllerName.class);
    
    private final SomeDAO someDAO;
    
    @Autowired
    public ControllerName(SomeDAO someDAO) {
        this.someDAO = someDAO;
    }
    
    @RequestMapping(path="/endpoint", method=RequestMethod.POST)
    public void methodName(HttpServletRequest request, HttpServletResponse response) {
        LOGGER.info(Constants.start);
        LOGGER.info("ControllerName:methodName");
        // implementation
    }
}
```

### DAOs (Data Access Objects)

- Extend `JpaRepository<Entity, Long>` for standard CRUD operations
- Add custom query methods as needed
- Use method naming conventions for query derivation (e.g., `findByCellphone`)

### Services

- Implement service interfaces
- Use `@Service` annotation
- Handle business logic and orchestrate DAO calls

## Database

- Database: MySQL
- Connection configured in `application.properties` and `hibernate.cfg.xml`
- Tables: users, budgets, categories, accounts, transactions, items, rules
- Use Hibernate for ORM
- Sequences for ID generation (e.g., `users_sequence`, `budgets_sequence`)

## Building and Testing

### Build Commands

```bash
# Clean and compile
mvn clean compile

# Run tests
mvn test

# Create WAR file
mvn package

# Create executable JAR with dependencies
mvn clean compile assembly:single
```

### Test Guidelines

- Place tests in `src/test/java/com/miBudget/`
- Use JUnit Jupiter (JUnit 5)
- Test files should end with `Test.java`
- Test utility classes and processors

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aaronhunter1088) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
