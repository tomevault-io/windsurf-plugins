---
trigger: always_on
description: This is a local Flutter logging package that provides structured, configurable logging for the EduStation application.
---

# mutlu_log Package - Cursor AI Rules

## Package Overview
This is a local Flutter logging package that provides structured, configurable logging for the EduStation application.

## Core Principles
1. **Always use mutlu_log for logging** - Never use `print()` or `dart:developer.log()` directly
2. **Tag everything** - Every log should have a descriptive tag
3. **Use appropriate levels** - Match log severity to the level
4. **Include context** - Use the `data` parameter for structured information
5. **Never log sensitive data** - Passwords, tokens, PII must not be logged

## Import Statement
```dart
import 'package:mutlu_log/mutlu_log.dart';
```

## Log Levels (Use Appropriately)

### AppLog.d() - Debug
Use for detailed debugging information that's only needed during development.
```dart
AppLog.d('User tapped button', tag: 'UI', data: {'button_id': 'submit'});
AppLog.d('Cache hit for key', tag: 'Cache', data: {'key': 'user_123'});
```

### AppLog.i() - Info
Use for general informational messages about app flow and state changes.
```dart
AppLog.i('User logged in', tag: 'Auth', data: {'user_id': userId});
AppLog.i('Navigation to screen', tag: 'Navigation', data: {'route': '/profile'});
```

### AppLog.w() - Warning
Use for potentially problematic situations that aren't errors.
```dart
AppLog.w('API response took longer than expected', tag: 'Performance', data: {'duration_ms': 3000});
AppLog.w('Using cached data due to network error', tag: 'Cache');
```

### AppLog.e() - Error
Use for errors and exceptions. Always include error and stackTrace when available.
```dart
AppLog.e(
  'Failed to load user profile',
  tag: 'Profile',
  data: {'user_id': userId},
  error: exception,
  stackTrace: stackTrace,
);
```

## Standard Tags by Feature

Use these standard tags for consistency:

- **Network**: All HTTP/API calls (automatically used by DioLogInterceptor)
- **Auth**: Authentication and authorization
- **Database**: Local database operations
- **Cache**: Caching operations
- **Storage**: File and secure storage operations
- **Navigation**: Route changes and navigation
- **UI**: User interface interactions
- **ChatWS**: WebSocket for chat
- **Firebase**: Firebase services (messaging, analytics, crashlytics)
- **Payment**: Payment processing
- **Profile**: User profile operations
- **[FeatureName]**: Use the feature name for feature-specific logs (e.g., 'PrizeWheel', 'Events', 'Training')

## When to Add Logging

### DO Log:
- ✅ Entry/exit of important operations
- ✅ API calls (automatic via DioLogInterceptor)
- ✅ State changes in providers
- ✅ User actions that trigger business logic
- ✅ Error conditions and exceptions
- ✅ Performance-critical operations
- ✅ Background tasks and scheduled jobs
- ✅ WebSocket connections and messages

### DON'T Log:
- ❌ Widget build methods (too verbose)
- ❌ Getters/setters (unless complex logic)
- ❌ Simple utility functions
- ❌ Every line of code (log strategically)
- ❌ Sensitive data (passwords, tokens, PII)

## Common Patterns

### In Providers (Riverpod)
```dart
class MyFeatureProvider extends StateNotifier<MyState> {
  Future<void> loadData() async {
    AppLog.d('Loading data started', tag: 'MyFeature');
    
    try {
      final result = await repository.fetchData();
      AppLog.i('Data loaded successfully', tag: 'MyFeature', data: {'count': result.length});
      state = state.copyWith(data: result);
    } catch (e, stackTrace) {
      AppLog.e('Failed to load data', tag: 'MyFeature', error: e, stackTrace: stackTrace);
      state = state.copyWith(error: e.toString());
    }
  }
}
```

### In Repositories
```dart
class UserRepository {
  Future<User> getUser(int userId) async {
    AppLog.d('Fetching user from API', tag: 'UserRepo', data: {'user_id': userId});
    
    try {
      final response = await networkHelper.get(path: '/users/$userId');
      // DioLogInterceptor automatically logs request/response
      return User.fromJson(response.data);
    } catch (e, stackTrace) {
      AppLog.e('Failed to fetch user', tag: 'UserRepo', error: e, stackTrace: stackTrace);
      rethrow;
    }
  }
}
```

### In Use Cases
```dart
class LoginUseCase {
  Future<LoginResult> execute(String email, String password) async {
    AppLog.i('Login attempt', tag: 'Auth', data: {'email': email});
    // Note: NEVER log passwords
    
    try {
      final result = await authRepository.login(email, password);
      AppLog.i('Login successful', tag: 'Auth', data: {'user_id': result.userId});
      return result;
    } catch (e, stackTrace) {
      AppLog.e('Login failed', tag: 'Auth', error: e, stackTrace: stackTrace);
      rethrow;
    }
  }
}
```

### In WebSocket/Stream Handlers
```dart
void _handleWebSocketMessage(Map<String, dynamic> data) {
  AppLog.d('WebSocket message received', tag: 'ChatWS', data: {'type': data['type']});
  
  try {
    final message = Message.fromJson(data);
    _messageController.add(message);
  } catch (e, stackTrace) {
    AppLog.e('Failed to parse WebSocket message', tag: 'ChatWS', error: e, stackTrace: stackTrace);
  }
}
```

### For Performance Monitoring
```dart
Future<void> heavyOperation() async {
  final stopwatch = Stopwatch()..start();
  
  try {
    await performOperation();

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Melsaeed276) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
