---
trigger: always_on
description: This document provides a detailed overview of the `GalgameManager.Server` application, the server-side component of the PotatoVN project. It is intended for AI agents and developers needing a deeper understanding of the server's architecture, features, and key code areas.
---

# GalgameManager.Server - Detailed Knowledge Base

This document provides a detailed overview of the `GalgameManager.Server` application, the server-side component of the PotatoVN project. It is intended for AI agents and developers needing a deeper understanding of the server's architecture, features, and key code areas.

## 1. Overview

*   **Component Name:** `GalgameManager.Server`
*   **Type:** Server Application (ASP.NET Core Web API)
*   **Primary Role:** To provide a backend for the `GalgameManager` client, enabling data synchronization and backup across devices.
*   **Part of Solution:** `GalgameManager.sln` (Project Path: `GalgameManager.Server/GalgameManager.Server.csproj`)
*   **Technology Stack:** ASP.NET Core, Entity Framework Core.
*   **Database:** PostgreSQL.
*   **Object Storage:** S3-compatible (e.g., Minio).

## 2. Core Features

*   **User Authentication:** Manages user accounts and authentication using JWT.
*   **Data Synchronization:**
    *   Provides RESTful API endpoints for the client to upload and download game-related data.
    *   Synchronizes game information (metadata, cover art links).
    *   Synchronizes game play status (`PlayType`).
    *   Synchronizes detailed play history (`PlayLog`, representing daily play duration) and total play time (`TotalPlayTime`).
    *   Synchronizes total play count (`PlayCount`).
    *   Synchronizes user comments and ratings for games.
    *   Synchronizes character information associated with games.
    *   Synchronizes staff (developer/creator) information.
*   **Data Storage:**
    *   Uses PostgreSQL to store structured data (user info, game metadata, play logs, etc.).
    *   Uses S3-compatible object storage for files like game cover images and character images.
*   **API for Client:** Exposes a RESTful API for the `GalgameManager` client. Client-side DTOs (e.g., `PotatoVN.Client.Model.GalgameUpdateDto`) are typically generated from this API's specification (e.g., OpenAPI/Swagger).

## 3. Architecture and Key Components

*   **`GalgameManager.Server.csproj`**: The MSBuild project file. Defines dependencies, build configurations.
*   **`Program.cs`**: Application entry point for ASP.NET Core, configures services and middleware.
*   **`appsettings.json`**: Configuration file (database connection strings, JWT keys, Minio settings are typically set via environment variables or user secrets for security).
*   **`Controllers/`**: Contains API controllers that handle incoming HTTP requests.
    *   **`GalgameController.cs`**: Handles all CRUD (Create, Read, Update, Delete) operations related to galgames.
        *   `GetGalgamesAsync`: Fetches a paginated list of games modified after a given timestamp.
        *   `GetGalgameAsync`: Fetches detailed information for a single game.
        *   `AddOrUpdateGalgameAsync`: Creates a new game or updates an existing one. This is the primary endpoint for synchronizing game data from the client. It accepts a `GalgameUpdateDto`.
        *   `AddPlayLogAsync`: Adds a specific play log entry for a game.
        *   `DeleteGalgameAsync`: Deletes a game.
    *   **`UserController.cs`**: Handles user registration, login, and profile management.
    *   **`OssController.cs`**: Likely handles direct interactions or signed URL generation for Object Storage.
    *   **`StaffController.cs`**: Handles CRUD operations for staff information.
*   **`Services/`**: Contains service classes that encapsulate business logic.
    *   **`GalgameService.cs` (`IGalgameService`)**: Implements the core logic for managing galgame data.
        *   `AddOrUpdateGalgameAsync(int userId, GalgameUpdateDto payload)`: This method is crucial for synchronization. It takes the `GalgameUpdateDto` from the controller, finds or creates a `Galgame` entity, and updates its properties based on the DTO. This includes mapping `payload.PlayCount`, `payload.TotalPlayTime`, and converting `payload.PlayTime` (list of `PlayLogDto`) to `PlayLog` entities.
    *   **`UserService.cs` (`IUserService`)**: Manages user-related operations.
    *   **`OssService.cs` (`IOssService`)**: Handles interactions with the S3-compatible object storage.
*   **`Repositories/`**: Contains repository classes that abstract data access logic (typically interacting with Entity Framework Core).
    *   `GalgameRepository.cs` (`IGalgameRepository`)
    *   `PlayLogRepository.cs` (`IPlayLogRepository`)
    *   `CharacterRepository.cs` (`ICharacterRepository`)
    *   `UserRepository.cs` (`IUserRepository`)
*   **`Data/`**:
    *   **`DataContext.cs`**: The Entity Framework Core `DbContext` class. Defines the database schema and provides access to tables (`DbSet` properties like `Galgames`, `Users`, `PlayLogs`).
*   **`Models/`**: Contains data model classes (entities) that map to database tables and DTOs (Data Transfer Objects) used for API communication.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GoldenPotato137/PotatoVN](https://github.com/GoldenPotato137/PotatoVN) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
