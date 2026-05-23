---
trigger: always_on
description: Spring Data JDBC provides a simpler alternative to JPA, offering direct SQL control while maintaining Spring's repository abstractions. When combined with Java records, it creates clean, immutable data models perfect for modern Java applications.
---

# Spring Data JDBC with Records

Spring Data JDBC provides a simpler alternative to JPA, offering direct SQL control while maintaining Spring's repository abstractions. When combined with Java records, it creates clean, immutable data models perfect for modern Java applications.

## Implementing These Principles

These guidelines are built upon the following core principles:

- **Immutability**: Use records for immutable entities that are thread-safe and predictable
- **Simplicity**: Leverage Spring Data JDBC's straightforward approach over complex ORM mapping
- **Constructor Injection**: Always use constructor-based dependency injection for better testability
- **Transaction Boundaries**: Keep transactions at the service layer, not repository layer
- **SQL Control**: Use custom queries when needed for optimal performance

## Table of contents

- Rule 1: Use Records for Entity Classes
- Rule 2: Implement Repository Pattern Correctly
- Rule 3: Handle Updates with Immutable Records
- Rule 4: Design Aggregate Relationships Properly
- Rule 5: Use Custom Queries for Complex Operations
- Rule 6: Implement Proper Transaction Management
- Rule 7: Embrace Single Query Loading to Eliminate N+1 Problems

## Rule 1: Use Records for Entity Classes

Title: Prefer Records Over Classes for Entity Definitions
Description: Records provide immutability, automatic equals/hashCode, and clean constructor-based mapping that works perfectly with Spring Data JDBC. They eliminate boilerplate code and ensure thread safety. Use @PersistenceCreator when you have multiple constructors to specify which one Spring Data JDBC should use. Use @Column to explicitly map record fields to database columns, especially when field names differ from column names.

**Good example:**

```java
public record Customer(
    @Id 
    @Column("customer_id") 
    Long id,
    
    @Column("first_name") 
    String firstName,
    
    @Column("last_name") 
    String lastName,
    
    @Column("email_address") 
    String email,
    
    @Column("created_at") 
    LocalDateTime createdAt
) {
    // Constructor for Spring Data JDBC (explicit annotation when multiple constructors exist)
    @PersistenceCreator
    public Customer(Long id, String firstName, String lastName, String email, LocalDateTime createdAt) {
        this.id = id;
        this.firstName = firstName;
        this.lastName = lastName;
        this.email = email;
        this.createdAt = createdAt;
    }
    
    // Factory method for new entities
    public static Customer of(String firstName, String lastName, String email) {
        return new Customer(null, firstName, lastName, email, LocalDateTime.now());
    }
}
```

**Bad Example:**

```java
// Missing @PersistenceCreator annotation with multiple constructors
public record Customer(
    @Id Long id,
    String firstName,
    String lastName,
    String email,
    LocalDateTime createdAt
) {
    // Multiple constructors without @PersistenceCreator - Spring Data JDBC won't know which to use
    public Customer(Long id, String firstName, String lastName, String email, LocalDateTime createdAt) {
        this.id = id;
        this.firstName = firstName;
        this.lastName = lastName;
        this.email = email;
        this.createdAt = createdAt;
    }
    
    public Customer(String firstName, String lastName, String email) {
        this(null, firstName, lastName, email, LocalDateTime.now());
    }
}

// Or using mutable entity class with boilerplate
public class Customer {
    @Id
    private Long id;
    private String firstName;
    private String lastName;
    private String email;
    private LocalDateTime createdAt;
    
    // Constructors, getters, setters, equals, hashCode...
    // 50+ lines of boilerplate code
}
```

## Rule 2: Implement Repository Pattern Correctly

Title: Extend Appropriate Repository Interfaces
Description: Use CrudRepository or PagingAndSortingRepository as base interfaces. Leverage method query derivation for simple queries and @Query for complex ones. Always annotate with @Repository.

**Good example:**

```java
@Repository
public interface CustomerRepository extends CrudRepository<Customer, Long> {
    
    // Method query derivation
    List<Customer> findByLastName(String lastName);
    Optional<Customer> findByEmail(String email);
    
    // Custom query for complex operations
    @Query("SELECT * FROM customer WHERE email LIKE :pattern")
    List<Customer> findByEmailPattern(@Param("pattern") String pattern);
}
```

**Bad Example:**

```java
// Missing @Repository annotation and poor method naming
public interface CustomerRepository extends CrudRepository<Customer, Long> {
    
    // Unclear method names that don't follow Spring Data conventions
    List<Customer> getCustomersWithLastName(String lastName);
    
    // Raw SQL without parameters
    @Query("SELECT * FROM customer WHERE email LIKE '%@gmail.com%'")
    List<Customer> findGmailUsers();
}
```

## Rule 3: Handle Updates with Immutable Records

Title: Create New Record Instances for Updates

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jabrena/cursor-rules-spring-boot](https://github.com/jabrena/cursor-rules-spring-boot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
