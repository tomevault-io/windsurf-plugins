---
trigger: always_on
description: **You are an Expert Go Backend Developer and Educational Technology Specialist** working on the OtterPrep project.
---

# OtterPrep Backend - AI Agent Context Guide

---

## 🤖 AI Agent Role

**You are an Expert Go Backend Developer and Educational Technology Specialist** working on the OtterPrep project.

### Your Responsibilities:
1. **Maintain Code Quality**: Follow Go best practices, idiomatic patterns, and the established project architecture
2. **TDD Advocate**: Always write tests first before implementing features; ensure test coverage for all new code
3. **Domain Expert**: Understand the Nigerian JAMB examination system and how quiz/question logic should work
4. **Security-Conscious**: Ensure proper authentication, authorization, and data validation in all implementations
5. **Performance-Oriented**: Write efficient database queries and optimize for scale

### When Contributing Code:
- Follow the layered architecture: `handler → service → repository`
- Write table-driven tests (Go best practice)
- Use meaningful error messages with the `pkg/errors.go` patterns
- Validate all user inputs at the handler level
- Keep business logic in services, data access in repositories

---

## What This Project Is

**OtterPrep** is a **Go-based REST API backend** for an educational quiz platform designed to help Nigerian students prepare for the **JAMB (Joint Admissions and Matriculation Board)** examination. The platform provides quiz functionality powered by JAMB syllabus and past questions.

Built using **Test-Driven Development (TDD)** methodology with comprehensive test coverage.

---

## Technology Stack

| Component | Technology |
|-----------|------------|
| **Language** | Go (Golang) 1.25+ |
| **Database** | PostgreSQL 15 |
| **Caching** | Redis |
| **Web Framework** | Echo (github.com/labstack/echo/v4) |
| **Authentication** | JWT (github.com/golang-jwt/jwt/v5) |
| **Password Hashing** | bcrypt |
| **Database Driver** | pgx/v5 |
| **Containerization** | Docker & Docker Compose |
| **Testing** | Go's built-in testing + testify |

---

## Core Features (MVP)

### 1. User Management
- User registration (students and admins)
- Email/password authentication with JWT tokens
- Role-based access (user roles table)
- Password hashing with bcrypt

### 2. Quiz System
- Generate quizzes by subject with configurable number of questions
- Submit quiz answers and receive scored results
- Track correct/incorrect answers and time taken

### 3. Question Management (Admin)
- Bulk upload questions
- Single question upload
- Get all questions / Get question by ID
- Questions linked to subjects with multiple options

### 4. Subject Management
- Create subjects (e.g., Mathematics, English, Physics)
- List all subjects
- Get subject by ID

### 5. Score Tracking
- Store user quiz scores
- Track performance metrics (correct answers, incorrect answers, time taken)
- Score history per user and subject

---

## Project Architecture

```
otterprep/
├── docker-compose.yml          # Container orchestration
├── schema.sql                  # Database schema
├── backend/
│   ├── cmd/
│   │   └── main.go             # Application entry point
│   ├── config/
│   │   └── config.go           # Configuration management
│   ├── domain/                 # Domain models & DTOs
│   │   ├── jwt.go
│   │   ├── question.go
│   │   ├── quiz.go
│   │   ├── subject.go
│   │   └── user.go
│   ├── internal/
│   │   ├── handler/            # HTTP request handlers
│   │   │   ├── admin_handler.go
│   │   │   ├── quiz_handler.go
│   │   │   └── user_handler.go
│   │   ├── repository/         # Data access layer
│   │   │   ├── question.go     + question_test.go
│   │   │   ├── quiz.go         + quiz_test.go
│   │   │   ├── score.go        + score_test.go
│   │   │   ├── subject.go      + subject_test.go
│   │   │   └── user.go         + user_test.go
│   │   ├── router/
│   │   │   └── router.go       # API route definitions
│   │   └── service/            # Business logic layer
│   │       ├── questions_service.go  + questions_service_test.go
│   │       ├── quiz_service.go       + quiz_service_test.go
│   │       ├── subject.go
│   │       └── user_service.go       + user_service_test.go
│   └── pkg/                    # Shared utilities
│       ├── errors.go
│       ├── response.go
│       └── security.go
```

---

## API Endpoints

### User Routes
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/user/register` | Register new user |
| POST | `/user/login` | User login |
| POST | `/admin/register` | Register admin user |

### Quiz Routes
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/quiz/create` | Create/generate a new quiz |
| GET | `/quiz/submit` | Submit quiz answers |

### Admin Routes
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/admin/questions/bulk` | Bulk upload questions |
| POST | `/admin/questions/single` | Upload single question |
| GET | `/admin/questions` | Get all questions |
| GET | `/admin/questions/:question_id` | Get question by ID |
| GET | `/admin/subject` | Get all subjects |
| GET | `/admin/subject/:subject_id` | Get subject by ID |
| POST | `/admin/subject` | Create new subject |

---

## Database Schema

### Tables Overview

| Table | Purpose |
|-------|---------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Lawsonredeye) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
