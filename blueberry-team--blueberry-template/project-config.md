---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Blueberry Template is a Flutter starter template (Korean-language project) with clean architecture, Material 3 theming, Riverpod 3.0 state management, easy_localization, and Firebase Crashlytics.

- **Flutter Version:** 3.35.6 (managed via FVM)
- **Primary Language:** Korean (documentation and comments in Korean)
- **State Management:** Riverpod 3.0
- **Min SDK:** 3.8.1

## Development Commands

### Setup & Dependencies

```bash
# Install Flutter version via FVM (required for this project)
fvm install

# Install dependencies
fvm flutter pub get

# Add new packages (always use this instead of manual pubspec.yaml edits)
fvm flutter pub add package_name
fvm flutter pub add package_name --dev  # for dev dependencies
```

### Running & Testing

```bash
# Run the app
fvm flutter run

# Run tests with coverage
flutter test --coverage

# Run a specific test file
flutter test test/path/to/test_file.dart

# Run a specific test by name
flutter test --name "test name"

# Lint analysis
flutter analyze --no-fatal-infos

# Format code
dart format .

# Build Android APK
flutter build apk
```

### Code Generation (when using freezed, json_serializable, etc.)

```bash
fvm flutter pub run build_runner build --delete-conflicting-outputs
```

## Architecture Overview

### Feature-Based Modular Structure

This codebase follows a **feature-first architecture** where each feature is self-contained:

```
lib/
├── core/              # Global app concerns
│   ├── controllers/   # Global state (e.g., ThemeController)
│   └── themes/        # Design system (AppColors, AppTypography, AppTheme)
└── features/          # Feature modules (isolated & independent)
    ├── todo/
    ├── github/
    └── [feature]/
        ├── controllers/  # Riverpod state management
        ├── models/       # Data models
        ├── screens/      # UI screens
        ├── widgets/      # Feature-specific widgets (optional)
        └── repositories/ # API/data layer (optional - see guidelines below)
```

### When to Create a Repository Layer

**Create `repositories/` when:**
- Making REST API calls (see `features/github/repositories/github_repository.dart` as example)
- Implementing GraphQL queries
- Working with local databases (SQLite, Hive)
- Need to abstract complex data operations shared across multiple controllers
- Using Firebase/Supabase SDKs but want to organize code better (코드 정리 목적으로 Repository 생성 가능)

**Skip `repositories/` when:**
- Managing simple UI state only
- The abstraction doesn't add value (YAGNI 원칙)

**Example: REST API pattern (github feature)**

```dart
// ✅ Repository 계층 (features/github/repositories/github_repository.dart)
class GitHubRepository {
  static const _baseUrl = 'https://api.github.com';

  Future<GithubRepoModel> getRepo({
    required String owner,
    required String repo,
  }) async {
    final url = Uri.parse('$_baseUrl/repos/$owner/$repo');
    final response = await http.get(url);

    if (response.statusCode != 200) {
      throw Exception('Failed to load repository: ${response.statusCode}');
    }

    final data = jsonDecode(response.body) as Map<String, dynamic>;
    return GithubRepoModel.fromJson(data);
  }
}

// ✅ Controller에서 Repository 사용 (features/github/controllers/github_controller.dart)
final githubProvider = AsyncNotifierProvider<GitHubNotifier, GithubRepoModel>(
  GitHubNotifier.new,
);

class GitHubNotifier extends AsyncNotifier<GithubRepoModel> {
  final _repository = GitHubRepository();

  @override
  Future<GithubRepoModel> build() {
    return _repository.getRepo(owner: 'blueberry-team', repo: 'blueberry_template');
  }

  Future<void> refresh() async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(
      () => _repository.getRepo(owner: 'blueberry-team', repo: 'blueberry_template'),
    );
  }
}
```

### Provider 작성 규칙

**동기(Synchronous) 상태 - NotifierProvider**

```dart
// ✅ GOOD: 동기 상태는 NotifierProvider 사용
final todoProvider = NotifierProvider<TodoController, List<Todo>>(
  TodoController.new,
);

class TodoController extends Notifier<List<Todo>> {
  @override
  List<Todo> build() => [];  // 동기 초기값

  void add(String title) {
    state = [...state, Todo(title)];
  }
}
```

**비동기(Asynchronous) 상태 - AsyncNotifierProvider**

```dart
// ✅ GOOD: 비동기 상태는 AsyncNotifierProvider 사용
final userProvider = AsyncNotifierProvider<UserController, User?>(
  UserController.new,
);

class UserController extends AsyncNotifier<User?> {
  final _userRepository = UserRepository();
  final _authRepository = AuthRepository();

  @override
  Future<User?> build() async {
    // 비동기 초기화 (API 호출, DB 읽기 등)
    return await _userRepository.getCurrentUser();
  }

  Future<void> signIn(String email, String password) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      return await _authRepository.signIn(email, password);
    });
  }

  Future<void> signOut() async {
    state = const AsyncLoading();
    await _authRepository.signOut();
    state = const AsyncData(null);
  }
}
```

**사용 예시:**

```dart
// 동기 Provider
final todos = ref.watch(todoProvider);

// 비동기 Provider

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blueberry-team/blueberry_template](https://github.com/blueberry-team/blueberry_template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
