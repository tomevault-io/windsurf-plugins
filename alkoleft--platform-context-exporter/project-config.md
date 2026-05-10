---
trigger: always_on
description: Modern Java Development Guidelines (Java 8+)
---

# Modern Java Development Guidelines (Java 8+)

Modern Java development (Java 8+) emphasizes leveraging lambda expressions and functional interfaces over anonymous classes, and using the Stream API for declarative collection processing. The `Optional` API should be used for handling potentially absent values gracefully, and the `java.time` API for all date/time operations. Default methods allow non-breaking interface evolution. Local Variable Type Inference (`var`) can improve readability when used judiciously. Unmodifiable collection factory methods (`List.of()`, etc.) provide concise immutable collections. `CompletableFuture` facilitates composable asynchronous programming. The Java Platform Module System (JPMS, Java 9+) enables strong encapsulation. Performance implications of new features should be considered and profiled. Testing strategies need to adapt to these modern features, and text blocks (Java 15+) offer improved readability for multi-line strings.

## Implementing These Principles

These guidelines are built upon the following core principles:

1.  **Conciseness and Readability**: Leverage modern Java features (like lambdas, streams, `var`, text blocks) to write code that is more concise and easier to read and understand, reducing boilerplate and focusing on intent.
2.  **Immutability and Safety**: Embrace features like `Optional`, unmodifiable collection factories, and the `java.time` API to create more robust, null-safe, and thread-safe code by default, reducing common sources of bugs.
3.  **Expressive Power**: Utilize functional constructs like streams and `CompletableFuture` to express complex data manipulations and asynchronous workflows in a more declarative and composable manner.
4.  **Asynchronous and Modular Design**: Employ `CompletableFuture` for efficient asynchronous programming and the Java Platform Module System (JPMS) for building maintainable, strongly encapsulated applications with clear dependencies.
5.  **Performance Awareness**: While modern features offer syntactic improvements, remain mindful of their potential performance implications. Profile critical sections and make informed decisions, especially with streams and asynchronous operations.

## Table of contents

- Rule 1: Lambda Expressions and Functional Interfaces
- Rule 2: Stream API
- Rule 3: Optional API
- Rule 4: Date/Time API (java.time)
- Rule 5: Default Methods in Interfaces
- Rule 6: Local Variable Type Inference (var)
- Rule 7: Collection Factory Methods
- Rule 8: CompletableFuture for Asynchronous Programming
- Rule 9: Module System (Java 9+)
- Rule 10: Performance Considerations with Modern Features
- Rule 11: Testing Modern Java Code
- Rule 12: Use Text Blocks for Readable Multi-line Strings

## Rule 1: Lambda Expressions and Functional Interfaces

Title: Effectively Use Lambda Expressions and Functional Interfaces
Description:
- Prefer lambda expressions over anonymous inner classes for concise implementation of functional interfaces.
- Keep lambda expressions short, readable, and focused on a single piece of logic.
- Use method references (e.g., `System.out::println`, `String::isEmpty`) when they are clearer and more direct than an equivalent lambda.
- Leverage the rich set of built-in functional interfaces from the `java.util.function` package (e.g., `Predicate`, `Function`, `Consumer`, `Supplier`).
- Create custom functional interfaces only when a specific signature is needed that isn't covered by built-in ones.
- Always annotate custom functional interfaces with `@FunctionalInterface` to ensure they meet the criteria (a single abstract method) and to clearly communicate their purpose.

**Good example:**
```java
import java.util.List;
import java.util.stream.Collectors;
import java.util.Arrays;
import java.util.function.Predicate;

// Custom functional interface
@FunctionalInterface
interface DataProcessor<T, R> {
    R process(T data);
}

public class LambdaExample {
    public static void main(String args) {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David", "Eve");

        // Good: Using method reference
        System.out.println("Printing names using method reference:");
        names.forEach(System.out::println);

        // Good: Simple lambda for filtering
        List<String> longNames = names.stream()
            .filter(str -> str.length() > 4) // Lambda expression
            .collect(Collectors.toList());
        System.out.println("\nLong names (length > 4): " + longNames);

        // Good: Using a built-in functional interface (Predicate)
        Predicate<String> startsWithA = s -> s.startsWith("A");
        List<String> namesStartingWithA = names.stream()
            .filter(startsWithA)
            .collect(Collectors.toList());
        System.out.println("Names starting with 'A': " + namesStartingWithA);

        // Good: Using a custom functional interface
        DataProcessor<String, Integer> nameLengthProcessor = (String name) -> name.length();
        int lengthOfAlice = nameLengthProcessor.process("Alice");
        System.out.println("Length of 'Alice' using custom processor: " + lengthOfAlice);
    }
}
```

**Bad Example:**
```java
import java.util.List;
import java.util.Arrays;
import java.util.function.Consumer;

public class OldStyleAnonymousClass {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alkoleft/platform-context-exporter](https://github.com/alkoleft/platform-context-exporter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
