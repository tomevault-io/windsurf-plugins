---
trigger: always_on
description: Follow these project rules
---

## Project Overview

*   **Type:** cursor_project_rules
*   **Description:** I'm developing a multi-tenant, event-driven mobile point of sale (POS) system targeting informal traders in Africa, starting with South Africa. This open-source initiative aims to provide affordable and scalable POS solutions tailored for small-scale vendors who need efficient sales tracking and management functionalities. The system emphasizes a simple, intuitive, and touch-friendly interface with offline capabilities and real-time data synchronization to accommodate users with limited technical expertise.
*   **Primary Goal:** Build a robust, multi-tenant mobile POS system that leverages an event-driven architecture (with Apache Kafka) to ensure accurate real-time sales tracking, secure authentication including multi-factor options, and effective inventory management with offline synchronization and conflict resolution.

## Project Structure

### Framework-Specific Routing

*   **Directory Rules:**

    *   **React Native with Expo:** While traditional web frameworks have explicit routing patterns (e.g., Next.js app or pages directory), mobile projects using Expo typically organize navigation using a combination of a dedicated navigation folder (often powered by React Navigation) and a screens directory. Each screen (e.g., HomeScreen, SalesScreen, InventoryScreen) represents a route in the application.
    *   Example: Use a `screens/` folder where each file (e.g., `HomeScreen.js`, `SalesScreen.js`) represents a distinct view; navigation configurations are kept in `navigation/` (e.g., `AppNavigator.js`).

### Core Directories

*   **Versioned Structure:**

    *   **Mobile Frontend:**

        *   `screens/` → Contains individual screen components such as onboarding, authentication, dashboard, and feature-specific pages.
        *   `components/` → Reusable UI components (buttons, cards, input fields).
        *   `navigation/` → Navigation configuration using React Navigation.

    *   **Backend (Spring Boot):**

        *   `src/main/java/` → Core Java application files including controllers, services, and repositories.
        *   `src/main/resources/` → Application configuration files and resources.

### Key Files

*   **Stack-Versioned Patterns:**

    *   **React Native/Expo:**

        *   `App.js` → Root file initializing navigation and global providers.
        *   `screens/HomeScreen.js` → Main dashboard view showing key metrics.

    *   **Java Spring Boot:**

        *   `src/main/java/com/yourcompany/Application.java` → Entry point for the Spring Boot application.
        *   `src/main/resources/application.properties` → Configuration settings including database and security parameters.

## Tech Stack Rules

*   **Version Enforcement:**

    *   **Expo@[Latest]:** Adhere to the latest Expo SDK guidelines. The project must use the recommended directory structure (i.e., segregating screens, components, and navigation) and avoid mixing web-specific routing conventions.
    *   **Java Spring Boot@[Stable]:** Follow conventional multi-layered architecture patterns (Controller, Service, Repository) and use the latest stable version for security and performance enhancements.
    *   **PostgreSQL:** Implement tenant-specific schema isolation and enforce connection pooling best practices.
    *   **Docker & Kubernetes:** Use Docker for containerization with multi-stage builds for optimization, and Kubernetes for orchestrating scalable deployments, ensuring proper resource limits and rolling updates.
    *   **Apache Kafka:** Set up topic partitioning and secure broker communication to sustain high-throughput, event-driven data pipelines.
    *   **GitHub Actions:** Automate builds, tests, and deployments using caching and validation steps to ensure quality and fast feedback loops.

## PRD Compliance

*   **Non-Negotiable:**

    *   "We are building a multi-tenant, event-driven mobile point of sale (POS) system for small-scale informal traders in Africa, starting with South Africa. This open-source initiative is designed to serve vendors in markets and on the streets by providing a simple, affordable, and efficient way to track sales and manage inventory." (Refer to PRD for detailed requirements on multi-tenancy, offline functionality, real-time synchronization, and security measures such as multi-factor authentication.)

## App Flow Integration

*   **Stack-Aligned Flow:**

    *   **React Native with Expo Auth & Feature Flow:**

        *   Onboarding and authentication flows are implemented in screens (e.g., `OnboardingScreen.js`, `SignInScreen.js`) with role-based redirection (Admin, Vendor, Cashier).
        *   Home dashboard (`HomeScreen.js`) displays key metrics and alerts (low stock, sales trends) to guide user actions.
        *   Sales and inventory management screens operate with real-time data integration using backend APIs and event handling via Apache Kafka.

## Best Practices

*   **React Native**

    *   Use a component-based architecture to ensure reusability and maintainability.
    *   Leverage React Navigation for smooth, declarative routing and screen transitions.
    *   Optimize performance using lazy loading and efficient list rendering (e.g., FlatList).

*   **Expo**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [soffoalbert/infologic-pos](https://github.com/soffoalbert/infologic-pos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
