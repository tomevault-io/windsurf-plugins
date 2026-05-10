---
trigger: always_on
description: Java Object-Oriented Design Guidelines
---

# Java Object-Oriented Design Guidelines

This document provides comprehensive guidelines for robust Java object-oriented design and refactoring. It emphasizes core principles like SOLID, DRY, and YAGNI, best practices for class and interface design including favoring composition over inheritance and designing for immutability. The rules also cover mastering encapsulation, inheritance, and polymorphism, and finally, identifying and refactoring common object-oriented design code smells such as God Classes, Feature Envy, and Data Clumps to promote maintainable, flexible, and understandable code.

## Implementing These Principles

These guidelines are built upon the following core principles:

1.  **Adherence to Fundamental Design Principles**: Embrace foundational principles like SOLID, DRY, and YAGNI. These principles are key to building systems that are robust, maintainable, flexible, and easy to understand.
2.  **Effective Class and Interface Design**: Employ best practices for designing classes and interfaces. This includes favoring composition over inheritance to achieve flexibility, programming to an interface rather than an implementation to promote loose coupling, keeping classes small and focused on a single responsibility, and designing for immutability where appropriate to enhance simplicity and thread-safety.
3.  **Mastery of Core OOP Concepts**: Thoroughly understand and correctly apply the pillars of object-oriented programming:
    *   **Encapsulation**: Protect internal state and expose behavior through well-defined interfaces.
    *   **Inheritance**: Model true "is-a" relationships, ensuring subclasses are substitutable for their base types (Liskov Substitution Principle).
    *   **Polymorphism**: Allow objects of different types to respond to the same message in their own way, simplifying client code.
4.  **Proactive Code Smell Management**: Develop the ability to identify common object-oriented design "code smells" (e.g., God Class, Feature Envy, Data Clumps, Refused Bequest). Recognizing and refactoring these smells is crucial for improving the long-term health, maintainability, and clarity of the codebase.

## Table of contents

- Rule 1: Adhere to Core Design Principles (SOLID, DRY, YAGNI)
- Rule 2: Follow Best Practices for Class and Interface Design
- Rule 3: Master Encapsulation, Inheritance, and Polymorphism
- Rule 4: Identify and Refactor Object-Oriented Design Code Smells
- Rule 5: Creating and Destroying Objects 
- Rule 6: Classes and Interfaces Best Practices 
- Rule 7: Enums and Annotations 
- Rule 8: Method Design
- Rule 9: Exception Handling

## Rule 1: Adhere to Core Design Principles (SOLID, DRY, YAGNI)

Title: Apply Fundamental Software Design Principles
Description: Core principles like SOLID, DRY, and YAGNI are foundational to good object-oriented design, leading to more robust, maintainable, and understandable systems.

### Sub-Rule 1.1: Single Responsibility Principle (SRP)
Title: A class should have one, and only one, reason to change.
Description: This means a class should only have one job or primary responsibility. If a class handles multiple responsibilities, changes to one responsibility might inadvertently affect others.

**Good example:**
```java
// Good: Separate responsibilities
class UserData {
    private String name;
    private String email;
    // constructor, getters
    public UserData(String name, String email) { this.name = name; this.email = email; }
    public String getName() { return name; }
    public String getEmail() { return email; }
}

class UserPersistence {
    public void saveUser(UserData user) {
        System.out.println("Saving user " + user.getName() + " to database.");
        // Database saving logic
    }
}

class UserEmailer {
    public void sendWelcomeEmail(UserData user) {
        System.out.println("Sending welcome email to " + user.getEmail());
        // Email sending logic
    }
}
```

**Bad Example:**
```java
// Bad: User class with multiple responsibilities
class User {
    private String name;
    private String email;

    public User(String name, String email) { this.name = name; this.email = email; }

    public String getName() { return name; }
    public String getEmail() { return email; }

    public void saveToDatabase() {
        System.out.println("Saving user " + name + " to database.");
        // Database logic mixed in
    }

    public void sendWelcomeEmail() {
        System.out.println("Sending welcome email to " + email);
        // Email logic mixed in
    }
    // If email sending changes, or DB logic changes, this class needs to change.
}
```

### Sub-Rule 1.2: Open/Closed Principle (OCP)
Title: Software entities should be open for extension but closed for modification.
Description: You should be able to add new functionality without changing existing, tested code. This is often achieved using interfaces, abstract classes, and polymorphism.

**Good example:**
```java
interface Shape {
    double calculateArea();
}

class Rectangle implements Shape {
    private double width, height;
    public Rectangle(double w, double h) { width=w; height=h; }
    @Override public double calculateArea() { return width * height; }
}

class Circle implements Shape {
    private double radius;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alkoleft/platform-context-exporter](https://github.com/alkoleft/platform-context-exporter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
