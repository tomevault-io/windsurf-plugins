---
trigger: always_on
description: - **Stack**: Spring Boot 3.2 / Java 21 / PostgreSQL / Hibernate 6.3 / Maven
---

# History Talk Backend – AI Guide

## Project Overview
- **Stack**: Spring Boot 3.2 / Java 21 / PostgreSQL / Hibernate 6.3 / Maven
- **Root**: `Source-code/SWD392_FinalProject_HistoryTalk/history-talk-backend`
- **Context path**: `/Historical-tell` — all API paths are `/Historical-tell/v1/...`
- **Swagger UI**: `http://localhost:8080/Historical-tell/api/v1/swagger-ui`
- **Layer order**: Controller → Service → Repository, with DTOs between every layer boundary

## Package Structure
```
com.historyTalk
├── config/          # SecurityConfig, SpringSecurityConfig, OpenApiConfig, SwaggerConfig
├── controller/
│   ├── authentication/   # AuthController
│   ├── character/        # CharacterController
│   ├── chat/             # ChatController
│   └── historicalContext/ # HistoricalContextController, HistoricalContextDocumentController
├── dto/
│   ├── authentication/   # LoginRequest/Response, RegisterRequest/Response, RefreshTokenResponse
│   ├── chat/             # CreateChatSessionRequest, ChatSessionResponse,
│   │                     # MessageResponse, GetMessagesResponse, SendMessageRequest,
│   │                     # SendMessageResponse, ChatHistorySessionItem, ChatHistoryGroupResponse
│   ├── character/        # Create/Update/Response DTOs for character
│   ├── historicalContext/ # Create/Update/Response DTOs for context and document
│   ├── exception/        # InvalidArgumentResponse
│   ├── user/             # UserInformationResponse
│   ├── ApiResponse.java
│   ├── PaginatedResponse.java
│   └── ValidationErrorResponse.java
├── entity/
│   ├── ContextStatus.java (enum – unused, kept for future)
│   ├── enums/     # UserRole (enum: CUSTOMER | STAFF | ADMIN)
│   ├── user/      # User (has UserRole role directly)
│   ├── historicalContext/ # HistoricalContext, HistoricalContextDocument
│   ├── character/ # Character, CharacterDocument
│   ├── chat/      # ChatSession, Message
│   └── quiz/      # Quiz, Question, QuizResult, QuizAnswerDetail
├── exception/
│   ├── BaseException.java (abstract, holds errorCode + HttpStatus)
│   ├── ResourceNotFoundException.java  → 404
│   ├── DataConflictException.java      → 409
│   ├── InvalidRequestException.java    → 400
│   ├── UnauthorizedException.java      → 401
│   ├── SystemException.java            → 500
│   ├── BusinessRuleViolationException  → RuntimeException (uncategorized rule breaks)
│   └── GlobalExceptionHandler.java
├── repository/    # HistoricalContextRepository, HistoricalContextDocumentRepository,
│                  # UserRepository, CharacterRepository, ChatSessionRepository,
│                  # MessageRepository
├── security/      # JwtAuthenticationFilter, JwtTokenProvider, UserPrincipal, AuthenticatedPrincipal
├── service/
│   ├── authentication/ # AuthService (interface), AuthServiceImpl, JwtService, JwtServiceImpl,
│   │                   # CustomUserDetailsService
│   ├── character/      # CharacterService
│   ├── chat/           # ChatSessionService, MessageService, ChatHistoryService, AiServiceClient
│   └── historicalContext/ # HistoricalContextService, HistoricalContextDocumentService
├── mapper/        # (character/, historicalContext/, user/ sub-packages)
└── utils/
    ├── authentication/
    └── SecurityUtils.java  # getUserId(), getRoleName() from SecurityContext
```

## Entity & ID Conventions
- **All primary keys are `UUID` type** with `@GeneratedValue + @UuidGenerator` — Hibernate auto-generates, stored as PostgreSQL native `uuid` column (16 bytes).
- **No `@PrePersist` for ID generation** — only use `@PrePersist` for setting non-null default values (e.g. `isFromAi = false`).
- FK relations use `@ManyToOne` / `@OneToMany(mappedBy=...)` with `FetchType.LAZY`.
- When a service receives an ID from a controller `@PathVariable` (String), convert with `UUID.fromString(id)` before calling repository.
- When mapping entity → DTO, call `.toString()` on UUID fields.

## Exception Hierarchy — Use These, Never Create Ad-hoc
| Class | HTTP | When to use |
|---|---|---|
| `ResourceNotFoundException` | 404 | Entity not found by ID / name |
| `DataConflictException` | 409 | Duplicate name, unique constraint violation |
| `InvalidRequestException` | 400 | Business rule validation failures |
| `UnauthorizedException` | 401 | Bad credentials, invalid/expired token |
| `SystemException` | 500 | Unexpected infrastructure errors |
| `BusinessRuleViolationException` | — (RuntimeException) | Misc rule violations not covered above |

All exceptions extend `BaseException(message, errorCode, httpStatus)` except `BusinessRuleViolationException` which extends `RuntimeException` directly.  
`GlobalExceptionHandler` catches `BaseException` subclasses and returns `ErrorResponse` with `errorCode`, `message`, `status`, `timestamp`. Do NOT return `ApiResponse` for exceptions — use `ErrorResponse`.

## Request/Response Patterns
- All **successful** HTTP responses wrap payloads: `ApiResponse.success(data, "message")` or `ApiResponse.error(message, errorCode)`.
- Paginated list endpoints return `PaginatedResponse<T>` (fields: `content`, `totalElements`, `totalPages`, `currentPage`, `pageSize`, `hasNext`, `hasPrevious`).
- Simple list endpoints (no pagination) return `List<T>` wrapped in `ApiResponse.success(...)`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DinhTQSE/SWD392_FinalProject_HistoryTalk](https://github.com/DinhTQSE/SWD392_FinalProject_HistoryTalk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
