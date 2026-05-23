---
trigger: always_on
description: This comprehensive guide provides essential principles for designing robust, maintainable, and secure REST APIs using Spring Boot. These rules ensure your APIs follow industry best practices, maintain consistency, and provide excellent developer experience for API consumers.
---

# Java REST API Design Principles

This comprehensive guide provides essential principles for designing robust, maintainable, and secure REST APIs using Spring Boot. These rules ensure your APIs follow industry best practices, maintain consistency, and provide excellent developer experience for API consumers.

## Implementing These Principles

These guidelines are built upon the following core principles:

- **Semantic Consistency**: Use HTTP methods, status codes, and URI patterns according to their intended semantics
- **Clear Communication**: Provide unambiguous API contracts through proper DTOs, error handling, and documentation
- **Security by Design**: Implement authentication, authorization, and input validation from the start
- **Evolutionary Design**: Version APIs and structure them to support future changes without breaking existing clients

## Table of contents

- Rule 1: Use HTTP Methods Correctly
- Rule 2: Design Clear and Consistent Resource URIs
- Rule 3: Use HTTP Status Codes Appropriately
- Rule 4: Implement Effective Request and Response Payloads (DTOs)
- Rule 5: Version Your APIs
- Rule 6: Handle Errors Gracefully
- Rule 7: Secure Your APIs
- Rule 8: Document Your APIs
- Rule 9: Use Controller Advice for Global Exception Handling
- Rule 10: Implement Problem Details for Error Responses

## Rule 1: Use HTTP Methods Correctly

Title: Employ HTTP Methods Semantically
Description: Use HTTP methods according to their defined semantics to ensure predictability and compliance with web standards. `GET` for retrieval, `POST` for creation, `PUT` for update/replace, `PATCH` for partial update, and `DELETE` for removal.

**Good example:**

```java
// Using Spring MVC annotations for illustration
@RestController
@RequestMapping("/users")
public class UserController {

    @GetMapping("/{id}") // GET for retrieving a user
    public ResponseEntity<UserDTO> getUser(@PathVariable String id) {
        // ... logic to fetch user ...
        return ResponseEntity.ok(new UserDTO());
    }

    @PostMapping // POST for creating a new user
    public ResponseEntity<UserDTO> createUser(@RequestBody UserCreateDTO userCreateDTO) {
        // ... logic to create user ...
        UserDTO newUser = new UserDTO(); // Assume it gets an ID after creation
        return ResponseEntity.created(URI.create("/users/" + newUser.getId())).body(newUser);
    }

    @PutMapping("/{id}") // PUT for replacing/updating a user
    public ResponseEntity<UserDTO> updateUser(@PathVariable String id, @RequestBody UserUpdateDTO userUpdateDTO) {
        // ... logic to update user ...
        return ResponseEntity.ok(new UserDTO());
    }

    @DeleteMapping("/{id}") // DELETE for removing a user
    public ResponseEntity<Void> deleteUser(@PathVariable String id) {
        // ... logic to delete user ...
        return ResponseEntity.noContent().build();
    }
    
    @PatchMapping("/{id}") // PATCH for partial updates
    public ResponseEntity<UserDTO> partiallyUpdateUser(@PathVariable String id, @RequestBody Map<String, Object> updates) {
        // ... logic to partially update user ...
        return ResponseEntity.ok(new UserDTO());
    }
}
// Dummy DTO classes
class UserDTO { private String id; public String getId() { return id; } /* ... other fields, getters, setters ... */ }
class UserCreateDTO { /* ... fields ... */ }
class UserUpdateDTO { /* ... fields ... */ }
```

**Bad Example:**

```java
@RestController
@RequestMapping("/api")
public class BadUserController {

    // Bad: Using GET to perform a state change (e.g., delete)
    @GetMapping("/deleteUser")
    public ResponseEntity<String> deleteUserViaGet(@RequestParam String id) {
        System.out.println("Deleting user: " + id + " (Bad: GET used for delete)");
        // ... delete logic ...
        return ResponseEntity.ok("User deleted (but GET was used!)");
    }

    // Bad: Using POST for all operations, including retrieval
    @PostMapping("/getUser")
    public ResponseEntity<UserDTO> getUserViaPost(@RequestBody String idPayload) {
        System.out.println("Fetching user: " + idPayload + " (Bad: POST used for GET)");
        // ... fetch logic ...
        return ResponseEntity.ok(new UserDTO());
    }
}
```

## Rule 2: Design Clear and Consistent Resource URIs

Title: Use Nouns for Resources and Maintain URI Consistency
Description: Design URIs that are intuitive and clearly represent resources. Use nouns (e.g., `/users`, `/orders`) instead of verbs. Keep URIs consistent in style (e.g., lowercase, hyphenated or camelCase for path segments).

**Good example:**

```
GET /users                           // Get all users
GET /users/{userId}                  // Get a specific user
GET /users/{userId}/orders           // Get all orders for a specific user
GET /users/{userId}/orders/{orderId} // Get a specific order for a user
POST /users                          // Create a new user
```

**Bad Example:**

```
GET /getAllUsers
GET /fetchUserById?id={userId}
POST /createNewUser

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jabrena/cursor-rules-spring-boot](https://github.com/jabrena/cursor-rules-spring-boot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
