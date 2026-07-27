---
trigger: always_on
description: This is an **Android application** demonstrating the **Model-View-Intent (MVI)** architectural pattern using **Kotlin Coroutines** and **Flow**. The project showcases a user management system with features to view, add, search, and remove users.
---

# GitHub Copilot Instructions for MVI-Coroutines-Flow

## Project Overview

This is an **Android application** demonstrating the **Model-View-Intent (MVI)** architectural pattern using **Kotlin Coroutines** and **Flow**. The project showcases a user management system with features to view, add, search, and remove users.

### Key Technologies
- **Language**: Kotlin 2.0.20
- **Architecture**: MVI (Model-View-Intent) + Clean Architecture
- **Concurrency**: Kotlin Coroutines & Flow
- **Functional Programming**: Arrow-kt for functional error handling
- **Dependency Injection**: Koin (master branch), Dagger Hilt (dagger_hilt branch - obsolete)
- **Testing**: JUnit, MockK, Kotlinx-Coroutines-Test
- **Minimum SDK**: API 21 (Android 5.0)
- **Target SDK**: API 35
- **Build System**: Gradle with Kotlin DSL

## MVI Pattern Implementation

### Core Concepts

The MVI pattern in this project follows these principles:

1. **Intent** (`MviIntent`): Represents user actions/intentions
   - Immutable objects that describe what the user wants to do
   - Examples: `ViewIntent.Refresh`, `ViewIntent.RemoveUser`, `ViewIntent.EmailChanged`

2. **Model/State** (`MviViewState`): Represents the UI state
   - Immutable, Parcelable data classes
   - Single source of truth for the UI
   - Example: `ViewState(isLoading, users, error)`

3. **View** (`MviView`): Renders the state
   - Activities/Fragments that implement `MviView<I, S, E>`
   - Observes `StateFlow<ViewState>` and `Flow<SingleEvent>`
   - Sends intents to ViewModel via `processIntent()`

4. **ViewModel** (`AbstractMviViewModel<I, S, E>`):
   - Processes intents through reactive streams
   - Produces partial state changes
   - Reduces partial changes into complete state
   - Emits single events for side effects (toasts, navigation)

### MVI Flow Architecture

```
User Action → ViewIntent → ViewModel.processIntent()
                ↓
         Intent Processor (Flow transformations)
                ↓
         PartialStateChange
                ↓
         State Reducer (scan operator)
                ↓
         ViewState (StateFlow) → View.render()
```

### State Management Pattern

```kotlin
// ViewState: Immutable UI state
data class ViewState(
  val isLoading: Boolean,
  val users: List<UserItem>,
  val error: UserError?
) : MviViewState

// ViewIntent: User actions
sealed interface ViewIntent : MviIntent {
  object Initial : ViewIntent
  object Refresh : ViewIntent
  data class RemoveUser(val user: UserItem) : ViewIntent
}

// SingleEvent: One-time events (toasts, navigation)
sealed interface SingleEvent : MviSingleEvent {
  data class ShowError(val error: UserError) : SingleEvent
  object NavigateToAddUser : SingleEvent
}
```

## Project Structure

### Module Architecture

The project follows **Clean Architecture** with multi-module structure:

```
├── app/                          # Application module (DI setup, initializers)
├── mvi/
│   ├── mvi-base/                # Core MVI abstractions (ViewModel, View, Intent, State)
│   └── mvi-testing/             # Testing utilities for MVI
├── domain/                       # Business logic layer
│   ├── model/                   # Domain models (User, Email, FirstName, LastName)
│   ├── repository/              # Repository interfaces
│   └── usecase/                 # Use cases (AddUser, RemoveUser, ObserveUsers, etc.)
├── data/                         # Data layer implementation
│   ├── remote/                  # API services (UserApiService, DTOs)
│   ├── mapper/                  # Mappers between layers
│   └── UserRepositoryImpl       # Repository implementation
├── feature-main/                 # Main screen feature (list users)
├── feature-add/                  # Add user feature
├── feature-search/               # Search users feature
├── core/                         # Core utilities (Mapper, EitherNes, Extensions)
├── core-ui/                      # UI utilities (ViewBinding, Extensions, Navigator)
└── test-utils/                   # Test utilities
```

### Dependency Flow
```
app → features → domain ← data
       ↓
    mvi-base
       ↓
    core, core-ui
```

## Coding Conventions & Patterns

### 1. Functional Error Handling with Arrow-kt

Use `Either` and `EitherNes` (Either with NonEmptySet) for error handling:

```kotlin
// EitherNes = Either<NonEmptySet<Error>, Value>
typealias EitherNes<E, A> = Either<NonEmptySet<E>, A>

// Domain layer
suspend fun addUser(user: User): Either<UserError, Unit>

// Validation with error accumulation
fun validateUser(email: String?, firstName: String?, lastName: String?): 
  EitherNes<UserValidationError, User> =
  Either.zipOrAccumulateNonEmptySet(
    Email.create(email),
    FirstName.create(firstName),
    LastName.create(lastName)
  ) { e, f, l -> User(email = e, firstName = f, lastName = l) }
```

### 2. Flow Operators & Patterns

**Key Flow patterns used:**

```kotlin
// Defer creation until subscription
defer { observeUsersUseCase() }

// Start with initial value
flow.startWith(PartialStateChange.Loading)

// FlatMap variants
flow.flatMapLatest { } // Cancel previous, switch to new

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kotlin-Android-Open-Source/MVI-Coroutines-Flow](https://github.com/Kotlin-Android-Open-Source/MVI-Coroutines-Flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
