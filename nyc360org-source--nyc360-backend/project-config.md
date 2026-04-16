---
trigger: always_on
description: This document provides a detailed overview of the `NYC360.Backend` project, covering its core specifics, implemented features, technology stack, architectural patterns, and coding conventions.
---

# NYC360.Backend Project Documentation

This document provides a detailed overview of the `NYC360.Backend` project, covering its core specifics, implemented features, technology stack, architectural patterns, and coding conventions.

## Project Overview

The `NYC360.Backend` project serves as the robust backend service for a platform primarily focused on the New York City (NYC) ecosystem. Its main purpose is to manage and facilitate various interactions related to community engagement, event organization, content posting, and comprehensive user management. The application provides a scalable and maintainable foundation for a dynamic user experience.

## Core Technologies and Frameworks

*   **Language:** C#
*   **Framework:** .NET (specifically ASP.NET Core for API development)
*   **Object-Relational Mapper (ORM):** Entity Framework Core (used for data access and persistence, managing database interactions and migrations).
*   **Caching:** Redis (integrated for distributed caching, enhancing performance and scalability).
*   **Email Services:** MailKit (utilized for sending emails, supporting various email-related functionalities).
*   **Authentication & Authorization:** JSON Web Tokens (JWT) for secure authentication, with support for Refresh Tokens and Role-Based Access Control (RBAC). OAuth2 integration, specifically with Google, is also supported.
*   **Database:** (Likely SQL Server, PostgreSQL, or another relational database, inferred from Entity Framework Core usage and migration capabilities, though not explicitly stated in directory structure).

## Architectural Overview

The project adheres to a layered architectural style, closely following Clean Architecture principles to ensure a strong separation of concerns, testability, and maintainability.

*   **NYC360.API (Presentation Layer):**
    *   **Responsibility:** Handles HTTP requests and responses, routing, serialization/deserialization, and API-specific configurations.
    *   **Components:** Contains API endpoints (controllers), Data Transfer Objects (DTOs), and web application startup configurations (`Program.cs`, `appsettings.json`).
    *   **Dependencies:** Depends on the `NYC360.Application` layer.

*   **NYC360.Application (Application Layer):**
    *   **Responsibility:** Orchestrates business logic, defines application-specific business rules, and acts as an intermediary between the UI and the Domain layer. It defines interfaces for infrastructure services.
    *   **Components:** Implements CQRS (Command Query Responsibility Segregation) patterns, handles use cases, and contains service interfaces (`Contracts`).
    *   **Dependencies:** Depends on the `NYC360.Domain` layer and defines contracts for the `NYC360.Infrastructure` layer.

*   **NYC360.Domain (Domain Layer):**
    *   **Responsibility:** Encapsulates the core business logic and rules of the application. It is the heart of the system and contains domain entities.
    *   **Components:** Contains `Entities`, `Enums`, `DTOs` (for domain-specific data), `Constants`, and generic `Wrappers` for standardized responses.
    *   **Dependencies:** Has no dependencies on other layers (it is the innermost layer).

*   **NYC360.Infrastructure (Infrastructure Layer):**
    *   **Responsibility:** Provides concrete implementations for the interfaces defined in the `NYC360.Application` layer. It handles external concerns such as data persistence, external APIs, file storage, and email services.
    *   **Components:** Includes implementations for `Cache` (Redis), `Email` (MailKit), `Identity` (JWT generation, password hashing), `OAuth` (Google integration), `Persistence` (Entity Framework Core `DbContexts`, `Migrations`, `Repositories`, `Seeders`), `RSS` services, and `Storage`.
    *   **Dependencies:** Depends on the `NYC360.Application` layer.

## Key Implemented Features

The project incorporates a variety of features spanning multiple functional domains:

*   **User Management:**
    *   User Registration (Regular Users and Organizations)
    *   User Login and Logout
    *   Password Management (Reset, Change Password)
    *   Email Confirmation
    *   Two-Factor Authentication (2FA)
    *   User Roles and Permissions

*   **Authentication & Authorization:**
    *   JWT Token Generation and Validation
    *   Refresh Token Mechanism
    *   Role-Based Access Control (RBAC)
    *   Google OAuth Integration

*   **Community Management:** Functionality for creating, managing, and interacting with communities.

*   **Event Management:** Capabilities for creating, listing, and managing events.

*   **Content Management:**
    *   Posting Content (e.g., articles, updates)
    *   Commenting on Posts
    *   Managing Post Categories

*   **Flagging System:**
    *   Mechanism for flagging inappropriate content or users.
    *   Flag Reason Types and Statuses.

*   **Location Services:** Management and categorization of locations (`LocationType`).

*   **RSS Integration:**
    *   RSS Feed Source Management.
    *   Background Service for RSS Feed Discovery and Processing.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nyc360org-source) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
