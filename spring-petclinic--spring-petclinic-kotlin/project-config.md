---
trigger: always_on
description: Manages pet owners, their pets, and pet types:
---

# Spring Petclinic Kotlin — Project Description

## 1. Project Overview and Purpose

**Spring Petclinic Kotlin** is a Kotlin-language implementation of the canonical [Spring Petclinic](https://github.com/spring-projects/spring-petclinic) sample application. It serves as a reference project for demonstrating how to build a full-stack web application using **Kotlin** and **Spring Boot**.

The application is a simplified veterinary clinic management system that allows users to:
- Search, create, and update **pet owners**
- Manage **pets** (add, edit, assign pet types)
- Record **vet visits** for each pet
- Browse a list of **veterinarians** and their **specialties**

This project is maintained under the [spring-petclinic](https://github.com/spring-petclinic) GitHub organization and is one of several technology-stack forks of the original Spring Petclinic application.

---

## 2. Technology Stack

| Category | Technology |
|---|---|
| **Language** | Kotlin 2.3.20 |
| **Framework** | Spring Boot 4.0.4 (Spring Framework 6) |
| **Build Tool** | Gradle 8+ with Kotlin DSL (`build.gradle.kts`) |
| **JVM Target** | Java 17 |
| **Web Layer** | Spring MVC + Thymeleaf templates |
| **UI Framework** | Bootstrap 5.3.8, Font Awesome 4.7.0, LESS stylesheets |
| **Persistence** | Spring Data JPA + Hibernate |
| **Databases** | H2 (in-memory, default) and MySQL (optional) |
| **Caching** | JCache (JSR-107) API with Spring `@EnableCaching` |
| **Validation** | Jakarta Bean Validation (Hibernate Validator) |
| **Testing** | JUnit 5, Mockito, AssertJ, Spring Boot Test (WebMvcTest, DataJpaTest) |
| **Load Testing** | Apache JMeter (`petclinic_test_plan.jmx`) |
| **Containerization** | Docker (Dockerfile + Google Jib plugin) |
| **Dev Tools** | Spring Boot DevTools (development-only) |
| **Actuator** | Spring Boot Actuator (all endpoints exposed) |

---

## 3. Architecture Overview

The application follows a classic **layered architecture** with a clear separation of concerns:

```
┌─────────────────────────────────────────────────────┐
│                  Presentation Layer                  │
│   ┌─────────────┐  ┌────────────┐  ┌──────────────┐ │
│   │ Controllers │  │ Thymeleaf  │  │ Static Assets│ │
│   │  (@Controller)│  │ Templates  │  │  (CSS, fonts)│ │
│   └──────┬──────┘  └────────────┘  └──────────────┘ │
│          │                                            │
├──────────┼────────────────────────────────────────────┤
│          ▼                                            │
│                  Business / Model Layer                │
│   ┌─────────────┐  ┌────────────┐  ┌──────────────┐ │
│   │  Entities   │  │ Validators │  │  Formatters  │ │
│   │  (@Entity)  │  │            │  │              │ │
│   └──────┬──────┘  └────────────┘  └──────────────┘ │
│          │                                            │
├──────────┼────────────────────────────────────────────┤
│          ▼                                            │
│                  Data Access Layer                     │
│   ┌──────────────────────────────────────────────┐   │
│   │         Repositories (Spring Data JPA)        │   │
│   │   (interface extends org.springframework.     │   │
│   │    data.repository.Repository)                │   │
│   └──────────────────────┬───────────────────────┘   │
│                          │                            │
├──────────────────────────┼────────────────────────────┤
│                          ▼                            │
│                  Database Layer                        │
│   ┌──────────────────────────────────────────────┐   │
│   │   H2 (default)  │  MySQL (profile-based)     │   │
│   └──────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

### Key Architectural Decisions

- **`proxyBeanMethods = false`** on `@SpringBootApplication` and `@Configuration` for improved startup performance (proxy-free `@Configuration` mode)
- **Constructor injection** via Kotlin's primary constructor syntax (no `@Autowired` needed)
- **Kotlin-specific compiler flag** `-Xjsr305=strict` for strict nullability handling with Jakarta annotations
- **Deferred JPA repository bootstrap** (`spring.data.jpa.repositories.bootstrap-mode=deferred`) for parallel startup
- **Open-in-view disabled** (`spring.jpa.open-in-view=false`) to avoid the anti-pattern of lazy-loading in views

---

## 4. Key Modules and Their Responsibilities

### 4.1 Model (`org.springframework.samples.petclinic.model`)

Base entity classes shared across the application:

| File | Responsibility |
|---|---|
| `BaseEntity.kt` | Abstract base with `@Id`, auto-generated identity, and `isNew` property |
| `NamedEntity.kt` | Extends `BaseEntity` with a `name` property (used by `PetType`, `Specialty`) |
| `Person.kt` | Extends `BaseEntity` with `firstName` and `lastName` (used by `Owner`, `Vet`) |

### 4.2 Owner Module (`org.springframework.samples.petclinic.owner`)

Manages pet owners, their pets, and pet types:

| File | Responsibility |
|---|---|
| `Owner.kt` | JPA entity; owns a `Set<Pet>`; has address, city, telephone |
| `Pet.kt` | JPA entity; belongs to an `Owner`; has a `PetType`, `birthDate`, and `Set<Visit>` |
| `PetType.kt` | JPA entity extending `NamedEntity` (e.g., "cat", "dog") |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spring-petclinic/spring-petclinic-kotlin](https://github.com/spring-petclinic/spring-petclinic-kotlin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
