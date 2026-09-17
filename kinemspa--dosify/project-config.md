---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Dosify is a Flutter-based medication management application that helps users track medications, doses, and schedules. The app supports both online (Firebase) and offline functionality with a sophisticated three-tier storage architecture.

## Development Commands

### Core Flutter Commands
- `flutter run` - Run the app in development mode
- `flutter build apk` - Build Android APK
- `flutter build ios` - Build iOS app
- `flutter clean` - Clean build artifacts
- `flutter pub get` - Install dependencies
- `flutter pub upgrade` - Update dependencies
- `flutter analyze` - Run static analysis
- `flutter test` - Run tests

### Platform-Specific Commands
- `flutter run -d android` - Run on Android device
- `flutter run -d ios` - Run on iOS device
- `flutter run -d chrome` - Run on web browser

### Build Commands
- `flutter build apk --release` - Build release APK
- `flutter build appbundle` - Build Android App Bundle
- `flutter build ios --release` - Build iOS release

## Architecture Overview

### Service-Oriented Architecture
The app uses a layered architecture with dependency injection:

1. **Service Layer**: Core business logic (Firebase, Encryption, Cache)
2. **Screen Layer**: UI components that inherit from `BaseServiceScreen`
3. **Widget Layer**: Reusable UI components
4. **Model Layer**: Data models with serialization support

### Key Services
- **FirebaseService**: Handles cloud and local data storage with encryption
- **EncryptionService**: Manages AES-256 encryption for sensitive data
- **CacheManager**: Multi-level caching for performance optimization
- **QueryOptimizer**: Optimizes Firestore queries with intelligent caching
- **ServiceLocator**: Dependency injection using GetIt

### Data Flow
```
User Action → Screen → Service Layer → Storage (Firebase/Local/Cache)
```

## Storage Architecture

### Three-Tier Storage System
1. **Firebase Firestore** (Primary): Cloud storage with real-time sync
2. **Encrypted Local Storage** (Secondary): AES-256 encrypted SharedPreferences
3. **Unencrypted Local Storage** (Fallback): Basic local storage for emergency access

### Data Persistence Strategy
- All operations work offline-first
- Data syncs to cloud when available
- Automatic failover between storage tiers
- Graceful degradation when services unavailable

## Key Models

### Medication Model
- Supports multiple medication types (tablet, capsule, injection variants)
- Includes inventory tracking and reconstitution calculations
- Handles injection-specific fields (route, diluent, concentration)

### Dose Model
- Tracks dose amounts with unit conversion
- Supports calculation-based dosing
- Links to medication for inventory management

### Schedule Models
- **Schedule**: Basic scheduling functionality
- **MedicationSchedule**: Medication-specific scheduling with inventory integration
- Both models support dose status tracking and calendar integration

## Screen Architecture

### BaseServiceScreen Pattern
All screens inherit from `BaseServiceScreen` which provides:
- Automatic service injection
- Loading state management
- Error handling with retry logic
- Consistent UI patterns

Example usage:
```dart
class MyScreen extends BaseServiceScreen {
  @override
  State<MyScreen> createState() => _MyScreenState();
}

class _MyScreenState extends BaseServiceScreenState<MyScreen> {
  @override
  Widget build(BuildContext context) {
    return buildServiceScaffold(
      appBar: AppBar(title: Text('My Screen')),
      body: () => _buildContent(),
    );
  }
}
```

## Key Dependencies

### Core Dependencies
- `firebase_core`, `firebase_auth`, `cloud_firestore` - Firebase integration
- `encrypt`, `crypto`, `flutter_secure_storage` - Security and encryption
- `provider`, `get_it` - State management and dependency injection
- `shared_preferences` - Local storage
- `uuid` - Unique ID generation

### UI Dependencies
- `flutter_form_builder`, `form_builder_validators` - Form handling
- `table_calendar` - Calendar components
- `intl` - Internationalization support

## Testing

### Running Tests
- `flutter test` - Run all tests
- `flutter test test/widget_test.dart` - Run specific test file
- `flutter test --coverage` - Generate coverage report

### Test Structure
Tests are located in the `test/` directory following Flutter conventions.

## Common Development Patterns

### Service Access
```dart
// Access services through BaseServiceScreen
final medication = await firebaseService.getMedication(medicationId);

// Or through ServiceLocator
final firebaseService = ServiceLocator.get<FirebaseService>();
```

### Error Handling
```dart
await executeWithLoading(
  () async => await firebaseService.addMedication(medication),
  onSuccess: (result) => Navigator.pop(context),
  onError: (error) => setError('Failed to save medication'),
);
```

### Loading States
```dart
setLoading(true);
try {
  // Perform operation
} finally {
  setLoading(false);
}
```

## Performance Considerations

### Caching Strategy
- Memory cache for frequently accessed data (30-minute TTL)
- Persistent cache for offline access

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kinemspa/dosify](https://github.com/kinemspa/dosify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
