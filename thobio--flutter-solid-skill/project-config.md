---
trigger: always_on
description: Transform Flutter code into senior-engineer quality mobile applications through SOLID principles, Test-Driven Development (TDD), Clean Architecture, and professional software design patterns. This skill ensures production-ready, maintainable, and scalable Flutter applications.
---

# Flutter SOLID Development Skill

## Overview

Transform Flutter code into senior-engineer quality mobile applications through SOLID principles, Test-Driven Development (TDD), Clean Architecture, and professional software design patterns. This skill ensures production-ready, maintainable, and scalable Flutter applications.

## When to Use This Skill

**ALWAYS use this skill when:**
- Writing any Flutter code (features, widgets, services, repositories)
- Refactoring existing Flutter applications
- Planning or designing app architecture
- Reviewing code quality and performance
- Debugging Flutter-specific issues
- Creating unit, widget, or integration tests
- Making architectural or design decisions
- Implementing state management solutions
- Working with API integrations and data layers

## Core Workflow: TDD-First Approach

**CRITICAL:** Always follow the Red-Green-Refactor cycle:

### Step 1: RED - Write Failing Test First
```dart
test('should fetch user profile from repository', () async {
  // Arrange
  when(mockRepository.getUser(any))
      .thenAnswer((_) async => Right(tUser));
  
  // Act
  final result = await useCase(Params(userId: tUserId));
  
  // Assert
  expect(result, Right(tUser));
  verify(mockRepository.getUser(tUserId));
  verifyNoMoreInteractions(mockRepository);
});
```

### Step 2: GREEN - Write Minimum Code to Pass
```dart
class GetUser implements UseCase<User, Params> {
  final UserRepository repository;
  
  GetUser(this.repository);
  
  @override
  Future<Either<Failure, User>> call(Params params) {
    return repository.getUser(params.userId);
  }
}
```

### Step 3: REFACTOR - Improve Design
- Apply SOLID principles
- Extract value objects
- Remove code smells
- Ensure clean architecture boundaries

## Flutter-Specific Architecture

### Layer Structure

```
lib/
├── core/
│   ├── error/
│   │   ├── exceptions.dart
│   │   └── failures.dart
│   ├── usecases/
│   │   └── usecase.dart
│   ├── utils/
│   └── network/
│       └── network_info.dart
├── features/
│   └── [feature_name]/
│       ├── data/
│       │   ├── datasources/
│       │   │   ├── [feature]_local_datasource.dart
│       │   │   └── [feature]_remote_datasource.dart
│       │   ├── models/
│       │   │   └── [feature]_model.dart
│       │   └── repositories/
│       │       └── [feature]_repository_impl.dart
│       ├── domain/
│       │   ├── entities/
│       │   │   └── [feature]_entity.dart
│       │   ├── repositories/
│       │   │   └── [feature]_repository.dart
│       │   └── usecases/
│       │       └── [specific_usecase].dart
│       └── presentation/
│           ├── bloc/
│           │   ├── [feature]_bloc.dart
│           │   ├── [feature]_event.dart
│           │   └── [feature]_state.dart
│           ├── pages/
│           │   └── [feature]_page.dart
│           └── widgets/
│               └── [feature]_widget.dart
└── injection_container.dart
```

## SOLID Principles for Flutter

### 1. Single Responsibility Principle (SRP)
**Each class/widget has ONE reason to change.**

❌ **Bad:** Widget doing business logic
```dart
class UserProfilePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // WRONG: HTTP call in widget
    final response = await http.get('api/user');
    final user = json.decode(response.body);
    return Text(user['name']);
  }
}
```

✅ **Good:** Separated concerns
```dart
// Widget only displays UI
class UserProfilePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocBuilder<UserBloc, UserState>(
      builder: (context, state) {
        return state.when(
          loaded: (user) => UserProfileView(user: user),
          loading: () => LoadingWidget(),
          error: (message) => ErrorWidget(message),
        );
      },
    );
  }
}

// Bloc handles state
class UserBloc extends Bloc<UserEvent, UserState> {
  final GetUser getUser;
  
  UserBloc({required this.getUser}) : super(UserInitial());
  
  @override
  Stream<UserState> mapEventToState(UserEvent event) async* {
    if (event is GetUserRequested) {
      yield UserLoading();
      final result = await getUser(Params(userId: event.userId));
      yield result.fold(
        (failure) => UserError(failure.message),
        (user) => UserLoaded(user),
      );
    }
  }
}

// UseCase handles business logic
class GetUser implements UseCase<User, Params> {
  final UserRepository repository;
  
  GetUser(this.repository);
  
  @override
  Future<Either<Failure, User>> call(Params params) {
    return repository.getUser(params.userId);
  }
}
```

### 2. Open/Closed Principle (OCP)
**Open for extension, closed for modification.**

✅ **Good:** Use abstract classes and composition
```dart
// Abstract datasource
abstract class AuthDataSource {
  Future<UserModel> login(LoginParams params);
  Future<void> logout();
}

// Implementation can be extended without modifying interface
class FirebaseAuthDataSource implements AuthDataSource {
  final FirebaseAuth firebaseAuth;
  
  FirebaseAuthDataSource(this.firebaseAuth);
  
  @override

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thobio/flutter-solid-skill-](https://github.com/thobio/flutter-solid-skill-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
