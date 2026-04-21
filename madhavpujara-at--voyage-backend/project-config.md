---
trigger: always_on
description: This rule, presentation_layer_sop.mdc, should be used when you are working on any aspect of the Presentation Layer of the application. This layer is responsible for all interactions with the "outside world." Refer to this rule if your task involves: Handling Incoming Requests: Creating or modifying how the application receives requests, whether they are from HTTP (e.g., REST API controllers, route definitions), WebSockets, Command-Line Interfaces (CLIs), or other external triggers. Input Validat
---

# Standard Operating Procedure (SOP): Presentation Layer Guidelines

## 1. Introduction & Purpose

The Presentation Layer is the outermost layer in a Clean Architecture system. Its primary role is to handle all interactions with the outside world. This includes, but is not limited to:

*   User interfaces (e.g., web pages, mobile app views)
*   External API endpoints (e.g., REST, GraphQL)
*   Command-Line Interfaces (CLI)
*   Message queue consumers or event listeners that initiate actions.

The Presentation Layer is responsible for receiving input from these external sources and presenting data back to them in an appropriate format. It acts as a gateway, translating external requests into calls to the Application Layer and formatting Application Layer responses for external consumption.

## 2. Core Responsibilities

The Presentation Layer has several key responsibilities:

*   **Request Handling**: Managing incoming requests from various sources (e.g., HTTP methods like GET, POST, PUT, DELETE; WebSocket messages; CLI commands).
*   **Input Validation**: Rigorously ensuring that all incoming data is valid, well-formed, and meets expected criteria (e.g., data types, formats, required fields) before any further processing.
*   **Data Transformation (Request)**: Mapping incoming request data (e.g., HTTP request bodies, query parameters, CLI arguments) to Data Transfer Objects (DTOs) or simple data structures expected by the Application Layer's use cases.
*   **Application Layer Interaction**: Invoking the appropriate use cases or application services within the Application Layer, passing the validated and transformed input data.
*   **Data Transformation (Response)**: Mapping data received from the Application Layer (e.g., Response DTOs, domain entities) into formats suitable for the client (e.g., JSON, XML, HTML, plain text).
*   **Response Generation**: Constructing and sending responses back to the client. For HTTP, this includes setting appropriate status codes (e.g., 200 OK, 201 Created, 400 Bad Request, 404 Not Found, 500 Internal Server Error) and headers.
*   **Error Handling**: Catching errors (both business exceptions and unexpected errors) propagated from the Application Layer and translating them into user-friendly error responses suitable for the client, without leaking sensitive internal details.
*   **(Optional) User Interface Rendering**: If the layer is responsible for rendering UI components or views (e.g., in server-side rendered web applications or desktop applications).

## 3. Recommended Directory Structure (within `presentation/`)

A well-organized Presentation Layer might have the following subdirectories:

*   **`controllers/`** (or `handlers/` for generic request handlers, `resolvers/` for GraphQL):
    *   **Purpose**: Contains the primary request-handling logic. Controllers orchestrate the flow of incoming requests to the appropriate Application Layer use cases and manage the response.
    *   **Details**: Each controller method typically corresponds to a specific endpoint, action, or user interaction. They should remain "thin" by delegating business logic.

*   **`routes/`**:
    *   **Purpose**: Defines the mapping between external entry points (e.g., API endpoints, CLI commands, event topics) and the corresponding controller methods or handlers.
    *   **Details**: This is where you configure how an incoming request like `GET /products/:id` is routed to `ProductController.getProductById()`.

*   **`validation/`**:
    *   **Purpose**: Houses schemas, rules, custom validation functions, or classes responsible for validating incoming request data.
    *   **Details**: Validation logic should be applied early in the controller before passing data to the Application Layer. Libraries like Zod, Joi, or class-validator are commonly used here.

*   **`interfaces/`** (or `types/`):
    *   **Purpose**: Defines TypeScript interfaces or types specifically for the Presentation Layer's request and response structures, especially if they differ from Application Layer DTOs or require client-specific shaping.
    *   **Details**: This helps ensure type safety and clear contracts for data exchanged with clients.

*   **(Optional) `mappers/`** (or `viewModels/`):
    *   **Purpose**: Used for transforming Application Layer DTOs or domain objects into presentation-specific formats, often called View Models. This is useful when the data structure required by the client (e.g., a specific UI component) differs significantly from what the Application Layer provides.
    *   **Details**: Mappers ensure that the Presentation Layer can adapt data to various client needs without altering the Application Layer.

*   **(Optional) `ui/`** or **`views/`**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/madhavpujara-at) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
