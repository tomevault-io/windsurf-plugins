---
trigger: always_on
description: Food Scan AI is a nutrition analysis platform with a **Flutter mobile app** (`flutter-app/`) and **Spring Boot backend** (`spring-backend/`). The app scans food product labels and meals using device cameras, leverages Google Cloud Vertex AI (Gemini 2.0 Flash) for intelligent analysis, and tracks daily nutritional intake. Firebase handles authentication, PostgreSQL stores persistent data, and both components communicate via REST APIs.
---

# AI Coding Agent Instructions for Food Scan AI

## Project Overview

Food Scan AI is a nutrition analysis platform with a **Flutter mobile app** (`flutter-app/`) and **Spring Boot backend** (`spring-backend/`). The app scans food product labels and meals using device cameras, leverages Google Cloud Vertex AI (Gemini 2.0 Flash) for intelligent analysis, and tracks daily nutritional intake. Firebase handles authentication, PostgreSQL stores persistent data, and both components communicate via REST APIs.

## Architecture & Key Patterns

### Flutter App Structure (MVVM Pattern)

- **`lib/viewmodels/`**: All ViewModels extend `BaseViewModel` (ChangeNotifier) for state management via Provider
  - Pattern: `ProductAnalysisViewModel`, `MealAnalysisViewModel`, `DailyIntakeViewModel`, etc.
  - Injected with repositories in [main.dart](flutter-app/lib/main.dart) using `ChangeNotifierProvider` and `ChangeNotifierProxyProvider`

- **`lib/repositories/`**: Repository pattern with interfaces
  - All data access abstracted behind interfaces (e.g., `AiRepositoryInterface`, `UserRepositoryInterface`)
  - Implementations: `SpringBackendRepository` for AI analysis, `IntakeRepository`, `UserRepository`
  - `ApiClient` handles HTTP requests with Firebase auth tokens

- **`lib/views/screens/`** and **`lib/views/widgets/`**: UI layer consumes ViewModels using `Consumer<ViewModel>`

- **State Management**: Provider exclusively - all providers registered in [main.dart](flutter-app/lib/main.dart) `_registerProviders()`

### Spring Boot Backend Structure

- **`controllers/`**: REST endpoints return `ApiResponse<T>` wrapper with `status`, `message`, `data` fields
  - Pattern: `@PostMapping` endpoints accept `MultipartFile` for image uploads
  - Example: [AiAnalysisController.java](spring-backend/src/main/java/com/scanmyfood/backend/controllers/AiAnalysisController.java) `/api/ai/analyze/product` and `/api/ai/analyze/image`

- **`services/`**: Business logic layer with interface-implementation pattern
  - `VertexAiServiceImpl` implements `AiService`, uses Google Cloud Vertex AI SDK
  - `AiResponseProcessingService` transforms raw AI JSON responses into typed POJOs

- **`models/`**: JPA entities (User, DailyIntake, ScanHistory) with Lombok annotations
  - Use `@Entity`, `@Table`, `@OneToOne`, `@ManyToOne` relationships
  - All entities have `createdAt`/`updatedAt` with `@PrePersist`/`@PreUpdate`
  - Firebase UID is used as user identifier (`firebaseUid` field)

- **`mapper/`**: MyBatis mappers for complex queries
  - Pattern: `@Mapper` interfaces with XML definitions in [src/main/resources/mapper/](spring-backend/src/main/resources/mapper/)
  - Example: [UserIntakeMapper.java](spring-backend/src/main/java/com/scanmyfood/backend/mapper/UserIntakeMapper.java) for daily intake aggregations

- **`configurations/`**: Spring beans for Firebase, Vertex AI, CORS
  - [VertexAIConfig.java](spring-backend/src/main/java/com/scanmyfood/backend/configurations/VertexAIConfig.java) initializes Google Cloud clients

### API Communication Pattern

1. Flutter calls `SpringBackendRepository` methods (e.g., `analyzeProductImages()`)
2. Multipart requests sent to backend `/api/ai/*` endpoints with auth tokens
3. Backend delegates to `VertexAiServiceImpl` → calls Gemini model
4. Response processed by `AiResponseProcessingService` → returns `ApiResponse<ProductAnalysisResponse>`
5. Flutter deserializes JSON using model `fromJson()` factories

### Data Models Synchronization

Keep Flutter and Spring models in sync:
- Flutter: [lib/models/](flutter-app/lib/models/) (Dart classes with `fromJson`/`toJson`)
- Spring: [src/main/java/.../models/](spring-backend/src/main/java/com/scanmyfood/backend/models/) (POJOs with Lombok)
- JSON uses **snake_case** (enforced by `PropertyNamingStrategies.SnakeCaseStrategy` in Spring)

## Critical Configuration Files

- **Backend**: [application.properties](spring-backend/src/main/resources/application.properties)
  - PostgreSQL: `spring.datasource.url=jdbc:postgresql://localhost:5432/scanmyfood`
  - Google Cloud: `google.cloud.project-id`, `google.cloud.location`
  - Storage: `storage.type=local` (or `s3`/`gcs` for production)
  - Service account key: `firebase-service-account.json` in `src/main/resources/`

- **Flutter**: [pubspec.yaml](flutter-app/pubspec.yaml) and `.env` file
  - `.env` contains `API_BASE_URL=http://localhost:8080/api` (default if not set)
  - Firebase config in [firebase_options.dart](flutter-app/lib/firebase_options.dart)

## Development Workflows

### Running the App

**Backend**:
```bash
cd spring-backend
./mvnw spring-boot:run  # or mvn spring-boot:run
```
Server runs on port 8080. Ensure PostgreSQL is running and database `scanmyfood` exists.

**Flutter**:
```bash
cd flutter-app
flutter pub get
flutter run
```

### Testing & Debugging


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nikhileshmeher0204/ScanMyFood](https://github.com/nikhileshmeher0204/ScanMyFood) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
