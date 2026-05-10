---
trigger: always_on
description: Java rules to apply data oriented programming style
---

# Java rules to apply data oriented programming style

Java Data-Oriented Programming emphasizes separating code (behavior) from data structures, which should ideally be immutable (e.g., using records). Data manipulation should occur via pure functions that transform data into new instances. It's often beneficial to keep data structures flat and denormalized (using IDs for references) where appropriate, and to start with generic data representations (like `Map<String, Object>`) converting to specific types only when necessary. Data integrity is ensured through pure validation functions. Flexible, generic data access layers facilitate working with various data types and storage mechanisms. All data transformations should be explicit, traceable, and composed of clear, pure functional steps.

## Implementing These Principles

These guidelines are built upon the following core principles:

1.  **Separation of Concerns (Data vs. Code)**: Strictly decouple data structures (which should be simple carriers like records or POJOs) from the code (behavior) that operates on them. Behavior should reside in separate utility classes or services.
2.  **Immutability**: Design data structures to be immutable. Use records or final fields, and ensure that any transformations on data produce new instances rather than modifying existing ones.
3.  **Pure Data Transformations**: Manipulate data using pure functions that depend only on their inputs and produce no side effects. This makes transformations predictable, testable, and easier to reason about.
4.  **Simplicity and Flexibility of Data Structures**: Prefer flat, denormalized data structures where appropriate, using IDs for references rather than deep nesting. Start with generic representations (like `Map<String, Object>`) if the schema is dynamic, converting to specific types only when necessary for processing.
5.  **Explicit and Traceable Operations**: Ensure all data validation and transformation steps are explicit, composed of clear functional steps, and easily traceable. Avoid hidden or implicit logic within data objects.

## Table of contents

- Rule 1: Separate Code from Data
- Rule 2: Data Should Be Immutable
- Rule 3: Use Pure Functions to Manipulate Data
- Rule 4: Keep Data Flat and Denormalized
- Rule 5: Keep Data Generic Until Specific
- Rule 6: Data Integrity through Validation Functions
- Rule 7: Flexible and Generic Data Access
- Rule 8: Explicit and Traceable Data Transformation

## Rule 1: Separate Code from Data

Title: Decouple Behavior (Code) from Data Structures
Description:
- Use records or simple POJOs primarily for holding data.
- Place behavior (methods that operate on data) in separate utility classes or services.
- Avoid mixing state (fields) and complex behavior (methods with logic) within the same class intended as a data carrier.
- Prefer static methods in utility classes for operations on data objects.
- Design data structures to be self-contained and focused solely on representing state.

**Good example:**

```java
// Data structure (record)
record UserData(String name, int age) {}

// Behavior in a separate utility class
class UserActions {
    public static void validateAge(UserData user) {
        if (user.age() < 0) {
            throw new IllegalArgumentException("Age cannot be negative: " + user.age());
        }
        System.out.println("Age for " + user.name() + " is valid.");
    }

    public static UserData activateUser(UserData user) {
        // Example transformation logic
        System.out.println("Activating user: " + user.name());
        return new UserData(user.name().toUpperCase(), user.age()); // Returns new data
    }
}

class SeparateCodeDataExample {
    public static void main(String args) {
        UserData user1 = new UserData("Alice", 30);
        UserActions.validateAge(user1);
        UserData activatedUser1 = UserActions.activateUser(user1);
        System.out.println("Activated user: " + activatedUser1);

        try {
            UserData user2 = new UserData("Bob", -5);
            UserActions.validateAge(user2); // This will throw
        } catch (IllegalArgumentException e) {
            System.err.println(e.getMessage());
        }
    }
}
```

**Bad Example:**

```java
// Mixing code and data in one class
class User {
    private String name;
    private int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // Behavior mixed with data
    public void validateAge() {
        if (age < 0) {
            throw new IllegalArgumentException("Age cannot be negative: " + age);
        }
        System.out.println("Age for " + name + " is valid.");
    }

    public String getName() { return name; }
    public int getAge() { return age; }

    public static void main(String args) {
        User user1 = new User("Alice", 30);
        user1.validateAge();
        // Problem: User object itself has methods, not just data.
        // If User was a record, it couldn't have such instance methods beyond generated ones.
    }
}
```

## Rule 2: Data Should Be Immutable

Title: Ensure Data Immutability
Description:
- Use records (which are inherently immutable) whenever possible for data carriers.
- Declare all fields as `final`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alkoleft/platform-context-exporter](https://github.com/alkoleft/platform-context-exporter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
