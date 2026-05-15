---
trigger: always_on
description: **CRITICAL**: Before starting any work or making significant changes, AI assistants MUST:
---

# Horcrux Cursor Rules

## Project Documentation

### Always Read README and CONTRIBUTING Guide
**CRITICAL**: Before starting any work or making significant changes, AI assistants MUST:
- Read `README.md` to understand the project overview, purpose, and current state
- Read `CONTRIBUTING.md` to understand development workflows, code generation requirements, and contribution guidelines
- Reference these documents when:
  - Starting a new feature or task
  - Encountering questions about project structure or conventions
  - Needing to understand build processes or tooling
  - Working with code generation (Freezed, build_runner, etc.)

## Service and Repository Architecture

### When to Use Repositories
Create a **Repository** class when data access meets ANY of these criteria:
- Complex caching logic (in-memory + persistence)
- Stream management for reactive updates
- Multiple specialized queries (e.g., by ID, by status, filtered lists)
- Multiple services need the same data access
- Might swap storage implementations (SharedPreferences → SQLite)
- Would be 100+ lines of data access code

**Example: VaultRepository**
```dart
final vaultRepositoryProvider = Provider<VaultRepository>((ref) {
  final repository = VaultRepository();
  ref.onDispose(() => repository.dispose());
  return repository;
});

class VaultRepository {
  final StreamController<List<Vault>> _controller = StreamController.broadcast();
  List<Vault>? _cache;
  
  Stream<List<Vault>> get stream => _controller.stream;
  Future<List<Vault>> getAll() async { /* load + cache */ }
  Future<Vault?> getById(String id) async { /* query cache */ }
  Future<void> save(Vault vault) async { /* persist + notify */ }
}
```

### When to Use Services Only
Use **Service-only** architecture (no repository) when:
- Simple CRUD operations (read/write SharedPreferences)
- Only one service needs this data
- Minimal or no caching logic
- No complex stream management
- Under 100 lines of data access

**Example: Service-Only Pattern**
```dart
final myServiceProvider = Provider<MyService>((ref) {
  return MyService(ref.read(otherServiceProvider));
});

class MyService {
  final OtherService _otherService;
  MyService(this._otherService);
  
  Future<void> doSomething() async {
    final prefs = await SharedPreferences.getInstance();
    // Simple data access inline
  }
}
```

### Service Pattern (Business Logic)
All services MUST:
- Be instance classes (not static)
- Use dependency injection via Riverpod
- Have a corresponding Provider
- Accept dependencies via constructor

**Example: Service with Dependencies**
```dart
final myServiceProvider = Provider<MyService>((ref) {
  return MyService(
    ref.read(repositoryProvider),
    ref.read(otherServiceProvider),
  );
});

class MyService {
  final MyRepository _repository;
  final OtherService _otherService;
  
  MyService(this._repository, this._otherService);
  
  Future<Result> doBusinessLogic() async {
    // Validation, orchestration, business rules
    final data = await _repository.getData();
    return _otherService.process(data);
  }
}
```

### Breaking Circular Dependencies
When services depend on each other, add explicit types:

```dart
// Both providers need explicit types to break inference cycle
final Provider<ServiceA> serviceAProvider = Provider<ServiceA>((ref) {
  final ServiceB serviceB = ref.read(serviceBProvider);
  return ServiceA(serviceB);
});

final Provider<ServiceB> serviceBProvider = Provider<ServiceB>((ref) {
  final ServiceA serviceA = ref.read(serviceAProvider);
  return ServiceB(serviceA);
});
```

### Don't Create Thin Wrappers
❌ **Bad: Thin repository that just delegates**
```dart
class KeyRepository {
  final Ref _ref;
  Future<String?> getKey() => KeyService.getKey();
  Future<void> clearKey() {
    await KeyService.clearKey();
    _ref.invalidate(keyProvider);
  }
}
```

✅ **Good: Use service directly with providers**
```dart
final keyServiceProvider = Provider<KeyService>((ref) => KeyService());

final currentKeyProvider = FutureProvider<String?>((ref) async {
  final keyService = ref.watch(keyServiceProvider);
  return await keyService.getCurrentKey();
});
```

## Code Verification Requirements (MANDATORY)

**CRITICAL**: AI assistants MUST verify code quality after EVERY change before responding to the user.

**⚠️ REMEMBER**: Every UI change requires:
1. Check for golden tests → Update goldens if they exist
2. Run `dart format .` 
3. Run ReadLints and `flutter analyze`
4. Commit UI changes + golden images together

### ⚠️ COMMIT REQUIREMENTS (ESPECIALLY FOR PRs) ⚠️

**BEFORE EVERY COMMIT** (especially when working on an open PR):

1. **Code Generation** ⚠️ MANDATORY - If you modified any `@freezed` classes or `@GenerateMocks` annotations:
   - **MUST run**: `flutter pub run build_runner build --delete-conflicting-outputs`
   - **MUST format**: `dart format .` (ALWAYS run after codegen - generated code needs formatting)
   - **MUST commit**: Generated files with your changes
2. **Format Code**: `dart format .` - MUST run before every commit
3. **Check Linter**: Use ReadLints tool on modified files - MUST be clean
4. **Run Analyzer**: `flutter analyze` - MUST show "No issues found!"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mplorentz/horcrux](https://github.com/mplorentz/horcrux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
