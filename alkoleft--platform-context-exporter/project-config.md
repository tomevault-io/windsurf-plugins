---
trigger: always_on
description: Java Secure coding guidelines
---

# Java Secure coding guidelines

This document provides essential Java secure coding guidelines, focusing on five key areas: validating all untrusted inputs to prevent attacks like injection and path traversal; protecting against injection attacks (e.g., SQL injection) by using parameterized queries or prepared statements; minimizing the attack surface by adhering to the principle of least privilege and reducing exposure; employing strong, current cryptographic algorithms for hashing, encryption, and digital signatures while avoiding deprecated ones; and handling exceptions securely by avoiding the exposure of sensitive information in error messages to users and logging detailed, non-sensitive diagnostic information for developers.

## Implementing These Principles

These guidelines are built upon the following core principles:

1.  **Comprehensive Input Validation**: Treat all external input as untrusted. Rigorously validate and sanitize data for type, length, format, and range before processing to prevent common vulnerabilities like injection attacks, path traversal, and buffer overflows.
2.  **Defense Against Injection**: Actively protect against all forms of injection attacks (e.g., SQL, OS Command, LDAP, XPath). Primarily achieve this by using safe APIs like parameterized queries (e.g., `PreparedStatement` in JDBC) or dedicated libraries that correctly handle data escaping, and by never directly concatenating untrusted input into executable commands or queries.
3.  **Attack Surface Minimization**: Adhere to the principle of least privilege for users, processes, and code. Reduce the exposure of system components by running with minimal necessary permissions, exposing only essential functionalities and network ports, and regularly reviewing and removing unused features, libraries, and accounts.
4.  **Strong Cryptographic Practices**: Employ current, robust, and industry-standard cryptographic algorithms and libraries for all sensitive operations, including hashing (especially for passwords), encryption, and digital signatures. Avoid deprecated or weak algorithms. Ensure cryptographic keys are generated securely, stored safely, and managed properly throughout their lifecycle.
5.  **Secure Exception Handling**: Manage exceptions in a way that does not expose sensitive information to users or attackers. Log detailed, non-sensitive diagnostic information for developers to aid in debugging, but provide generic, non-revealing error messages to clients. Avoid direct exposure of stack traces or internal system details in error outputs.

## Table of contents

- Rule 1: Input Validation
- Rule 2: Protect Against Injection Attacks
- Rule 3: Minimize Attack Surface
- Rule 4: Use Strong Cryptography
- Rule 5: Handle Exceptions Securely

## Rule 1: Input Validation

Title: Validate All Untrusted Inputs
Description: Always validate and sanitize data received from untrusted sources (users, network, files, etc.) before processing. This helps prevent various attacks like injection, path traversal, and buffer overflows. Validation should check for type, length, format, and range.

**Good example:**

```java
import java.util.Objects;
public void processUserData(String username, String ageString) {
    if (Objects.isNull(username) || !username.matches("^[a-zA-Z0-9_]{3,16}$")) {
        throw new IllegalArgumentException("Invalid username format.");
    }

    int age;
    try {
        age = Integer.parseInt(ageString);
        if (age < 0 || age > 120) {
            throw new IllegalArgumentException("Age out of valid range.");
        }
    } catch (NumberFormatException e) {
        throw new IllegalArgumentException("Invalid age format.");
    }

    // Proceed with validated username and age
    System.out.println("Processing user: " + username + ", age: " + age);
}
```

**Bad Example:**

```java
public void processUserData(String username, String ageString) {
    // Directly using input without validation
    int age = Integer.parseInt(ageString); // Potential NumberFormatException if ageString is not a number
                                         // No checks for malicious username strings

    System.out.println("Processing user: " + username + ", age: " + age);
    // This could lead to issues if username is e.g. a script or ageString is not an integer
}
```

## Rule 2: Protect Against Injection Attacks

Title: Use Parameterized Queries or Prepared Statements for Database Access
Description: To prevent SQL Injection, always use parameterized queries (PreparedStatements in JDBC) or an ORM that handles this automatically. Never concatenate user input directly into SQL queries. Similar principles apply to other types of injection (OS command, LDAP, XPath, etc.).

**Good example:**

```java
// Using PreparedStatement to prevent SQL Injection
String customerId = request.getParameter("customerId");
String query = "SELECT * FROM orders WHERE customer_id = ?";
try (Connection con = DriverManager.getConnection(DB_URL, USER, PASS);
     PreparedStatement pstmt = con.prepareStatement(query)) {

    pstmt.setString(1, customerId);
    ResultSet rs = pstmt.executeQuery();
    // Process results
} catch (SQLException e) {
    // Handle exception
}
```

**Bad Example:**

```java
// Vulnerable to SQL Injection

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alkoleft/platform-context-exporter](https://github.com/alkoleft/platform-context-exporter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
