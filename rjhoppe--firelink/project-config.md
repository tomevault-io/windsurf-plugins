---
trigger: always_on
description: This rule provides a comprehensive set of best practices for developing Go applications, covering code organization, performance, security, testing, and common pitfalls.
---

- # Go Best Practices

  This document outlines best practices for developing Go applications, covering various aspects of the development lifecycle.

- ## 1. Code Organization and Structure

  - ### 1.1 Directory Structure

    - **Recommended Structure:**


      project-name/
      ├── cmd/
      │   └── project-name/
      │       └── main.go  # Application entry point
      ├── internal/
      │   ├── app/         # Application-specific business logic
      │   ├── domain/      # Core domain logic and types
      │   └── pkg/          # Reusable internal packages
      ├── pkg/           # External packages (libraries for other projects)
      ├── api/           # API definitions (protobuf, OpenAPI specs)
      ├── web/           # Web assets (HTML, CSS, JavaScript)
      ├── scripts/       # Build, deployment, or utility scripts
      ├── configs/       # Configuration files
      ├── .gitignore
      ├── go.mod
      ├── go.sum
      └── README.md


    - **Explanation:**

      - `cmd`:  Contains the main applications for your project. Each subdirectory should represent a separate application.
      - `internal`:  Holds code that's private to your application. Other projects shouldn't import these.
        - `internal/app`: High-level application logic.
        - `internal/domain`: Core business logic, data models, and interfaces.
        - `internal/pkg`: Reusable utilities and helpers within the internal codebase.
      - `pkg`: Contains reusable libraries that can be used by other projects. Use this for code you want to share.
      - `api`: Defines API contracts (e.g., Protocol Buffers or OpenAPI/Swagger definitions).
      - `web`: Stores static web assets like HTML, CSS, and JavaScript files.
      - `scripts`: Contains scripts for building, testing, deploying, and other tasks.
      - `configs`: Houses configuration files for various environments.

  - ### 1.2 File Naming Conventions

    - **General:**  Use lowercase and snake_case for file names (e.g., `user_service.go`).
    - **Test Files:**  Append `_test.go` to the name of the file being tested (e.g., `user_service_test.go`).
    - **Main Package:** The file containing the `main` function is typically named `main.go`.

  - ### 1.3 Module Organization

    - **Go Modules:**  Use Go modules for dependency management.  Initialize a module with `go mod init <module-name>`. The module name should reflect the repository path (e.g., `github.com/your-username/project-name`).
    - **Versioning:** Follow semantic versioning (SemVer) for your modules.  Use tags in your Git repository to represent releases (e.g., `v1.0.0`).
    - **Vendoring:** Consider vendoring dependencies using `go mod vendor` to ensure reproducible builds, especially for critical applications. However, be mindful of vendor directory size.

  - ### 1.4 Component Architecture

    - **Layered Architecture:**  Structure your application into layers (e.g., presentation, service, repository, data access). This promotes separation of concerns and testability.
    - **Clean Architecture:** A variation of layered architecture that emphasizes dependency inversion and testability. Core business logic should not depend on implementation details.
    - **Microservices:** For larger applications, consider a microservices architecture where different parts of the application are deployed as independent services.
    - **Dependency Injection:** Use dependency injection to decouple components and make them easier to test. Frameworks like `google/wire` or manual dependency injection can be used.

  - ### 1.5 Code Splitting

    - **Package Organization:**  Group related functionality into packages.  Each package should have a clear responsibility.  Keep packages small and focused.
    - **Interface Abstraction:**  Use interfaces to define contracts between components.  This allows you to swap implementations without changing the code that depends on the interface.
    - **Functional Options Pattern:** For functions with many optional parameters, use the functional options pattern to improve readability and maintainability.

      go
      type Server struct {
          Addr     string
          Port     int
          Protocol string
          Timeout  time.Duration
      }

      type Option func(*Server)

      func WithAddress(addr string) Option {
          return func(s *Server) {
              s.Addr = addr
          }
      }

      func WithPort(port int) Option {
          return func(s *Server) {
              s.Port = port
          }
      }

      func NewServer(options ...Option) *Server {
          srv := &Server{
              Addr:     "localhost",
              Port:     8080,
              Protocol: "tcp",
              Timeout:  30 * time.Second,
          }

          for _, option := range options {
              option(srv)
          }

          return srv
      }

      // Usage
      server := NewServer(WithAddress("127.0.0.1"), WithPort(9000))


- ## 2. Common Patterns and Anti-patterns

  - ### 2.1 Design Patterns

    - **Factory Pattern:** Use factory functions to create instances of complex objects.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rjhoppe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
