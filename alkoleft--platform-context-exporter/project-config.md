---
trigger: always_on
description: Java Functional Programming rules
---

# Java Functional Programming rules

Java functional programming revolves around immutable objects and state transformations, ensuring functions are pure (no side effects, depend only on inputs). It leverages functional interfaces, concise lambda expressions, and the Stream API for collection processing. Core paradigms include function composition, `Optional` for null safety, and higher-order functions. Modern Java features like Records enhance immutable data transfer, while pattern matching (for `instanceof` and `switch`) and switch expressions improve conditional logic. Sealed classes and interfaces enable controlled, exhaustive hierarchies, and upcoming Stream Gatherers will offer advanced custom stream operations.

## Implementing These Principles

These guidelines are built upon the following core principles:

1.  **Immutability**: Prioritize immutable data structures (e.g., Records, `List.of()`) and state transformations that produce new instances rather than modifying existing ones. This reduces side effects and simplifies reasoning about state.
2.  **Purity and Side-Effect Management**: Strive to write pure functions—functions whose output depends only on their input and which have no observable side effects. Isolate and control side effects when they are necessary.
3.  **Expressiveness and Conciseness**: Leverage lambda expressions, method references, and the Stream API to write code that is declarative, concise, and clearly expresses the intent of data transformations and operations.
4.  **Higher-Order Abstractions**: Utilize functional interfaces, function composition, and higher-order functions (functions that operate on other functions) to build flexible and reusable code components.
5.  **Modern Java Integration**: Embrace modern Java features like Records, Pattern Matching, Switch Expressions, and Sealed Classes, which align well with and enhance functional programming paradigms by promoting immutability, type safety, and expressive conditional logic.

## Table of contents

- Rule 1: Immutable Objects
- Rule 2: State Immutability
- Rule 3: Pure Functions
- Rule 4: Functional Interfaces
- Rule 5: Lambda Expressions
- Rule 6: Streams
- Rule 7: Functional Programming Paradigms
- Rule 8: Leverage Records for Immutable Data Transfer
- Rule 9: Employ Pattern Matching for `instanceof` and `switch`
- Rule 10: Use Switch Expressions for Concise Multi-way Conditionals
- Rule 11: Leverage Sealed Classes and Interfaces for Controlled Hierarchies
- Rule 12: Explore Stream Gatherers for Custom Stream Operations

## Rule 1: Immutable Objects

Title: Ensure Objects are Immutable
Description:
- Use `final` classes and fields.
- Initialize all fields in the constructor.
- Do not provide setter methods.
- Return defensive copies of mutable fields (e.g., collections, dates) when exposing them via getters.

**Good example:**

```java
import java.util.List;
import java.util.ArrayList;

public final class Person {
    private final String name;
    private final int age;
    private final List<String> hobbies; // Make it List, not ArrayList

    public Person(String name, int age, List<String> hobbies) {
        this.name = name;
        this.age = age;
        // Ensure the incoming list is defensively copied to an immutable list
        this.hobbies = List.copyOf(hobbies); 
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    // Return an immutable view or a defensive copy
    public List<String> getHobbies() {
        return this.hobbies; // List.copyOf already returns an unmodifiable list
    }
}
```

**Bad Example:**

```java
// Bad example to be added
// e.g., a mutable class with setters, or returning internal mutable collections directly.
```

## Rule 2: State Immutability

Title: Prefer Immutable State Transformations
Description:
- Instead of modifying existing objects, return new objects representing the new state.
- Utilize collectors that produce immutable collections (e.g., `Collectors.toUnmodifiableList()`).
- Leverage immutable collection types provided by libraries or Java itself.

**Good example:**

```java
import java.util.List;
import java.util.stream.Collectors;

public class PriceCalculator {
    public static List<Double> applyDiscount(List<Double> prices, double discount) {
        return prices.stream()
            .map(price -> price * (1 - discount))
            .collect(Collectors.toUnmodifiableList()); // Ensures the returned list is immutable
    }
}
```

**Bad Example:**

```java
// Bad example to be added
// e.g., a method that modifies the input list directly.
```

## Rule 3: Pure Functions

Title: Write Pure Functions
Description:
- Functions should depend only on their input parameters and not on any external or hidden state.
- They should not cause any side effects (e.g., modifying external variables, I/O operations).
- Given the same input, a pure function must always return the same output.
- Avoid modifying external state or relying on it.

**Good example:**

```java
import java.util.List;
import java.util.stream.Collectors;

public class MathOperations {
    // Pure function: depends only on input, no side effects
    public static int add(int a, int b) {
        return a + b;
    }


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alkoleft/platform-context-exporter](https://github.com/alkoleft/platform-context-exporter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
