---
trigger: always_on
description: This document provides essential information for agentic coding agents working in this Dart package repository.
---

# AGENTS.md - Coding Guidelines for pocketbase_plus

This document provides essential information for agentic coding agents working in this Dart package repository.

## Project Overview

Pocketbase Plus is a Dart package that automates model generation for PocketBase projects. It fetches collection schemas from PocketBase and generates type-safe Dart model classes with CRUD helper methods.

**PocketBase SDK Version:** Requires PocketBase Dart SDK >=0.23.0 (compatible with PocketBase v0.23.0+)

**Breaking Changes from 0.18.x:**
- `SchemaField` is now `CollectionField`
- `CollectionModel.schema` is now `CollectionModel.fields`
- `RecordModel.created/updated` properties are deprecated; use `r.get<String>('created')` instead
- `pb.admins` is deprecated; use `pb.collection('_superusers')` for admin authentication

**Generated Files:**
- `<collection>.dart` - Model class with fields, constructor, fromModel, toMap, copyWith, equality, toString, file URL helpers
- `<collection>_extension.dart` - Extension methods for CRUD operations (pocketbase extensions)
- `models.dart` - Barrel file exporting all models and extensions

**Generated Model Features:**
- Collection ID/name constants (`collectionId`, `collectionName`)
- Static CRUD methods (`getOne`, `getList`, `getFullList`, `getFirst`, `create`, `update`, `delete`)
- Paginated result method (`getPaginated`) - returns `PaginatedResult<T>` with metadata
- Realtime subscriptions (`subscribe`, `subscribeAll`)
- Type-safe filter builders (`Model.f.name.eq('value')`)
- Equality and hash code overrides
- `toString()` override
- `copyWith()` method
- File URL helpers (for file fields)
- Auth helpers (for auth collections)

## Build/Lint/Test Commands

### Running the Generator
```bash
# Run with default config (./pocketbase.yaml)
dart run pocketbase_plus:main

# Run with custom config file
dart run pocketbase_plus:main --config pubspec.yaml
dart run pocketbase_plus:main -c path/to/config.yaml

# Override output directory
dart run pocketbase_plus:main --output ./lib/generated
dart run pocketbase_plus:main -o ./lib/generated

# Skip extension generation (no CRUD methods)
dart run pocketbase_plus:main --no-extensions

# Skip barrel file generation
dart run pocketbase_plus:main --no-barrel

# Show help
dart run pocketbase_plus:main --help
```

### Testing
```bash
# Run all tests
dart test

# Run a specific test file
dart test test/pocketbase_plus_test.dart

# Run tests with verbose output
dart test --reporter=expanded

# Run tests with coverage
dart test --coverage=coverage
```

### Linting & Analysis
```bash
# Run static analysis
dart analyze

# Check for issues without fixing
dart analyze --fatal-infos

# Format code (check without modifying)
dart format --output=none --set-exit-if-changed .
```

### Formatting
```bash
# Format all Dart files
dart format .

# Format specific directory
dart format lib/
dart format bin/

# Format specific file
dart format lib/src/models.dart
```

### Package Management
```bash
# Get dependencies
dart pub get

# Upgrade dependencies
dart pub upgrade

# Check for outdated dependencies
dart pub outdated
```

## Code Style Guidelines

### Naming Conventions

**Files & Directories:**
- Snake_case for file names: `models.dart`, `main.dart`
- Directory names should be snake_case: `lib/src/`

**Variables & Parameters:**
- camelCase for variables and parameters: `collection`, `outputDirectory`
- Use descriptive names: `configPath` not `path`

**Classes & Types:**
- PascalCase for class names: `CollectionModel`, `UsersModel`
- Enums use PascalCase with `Enum` suffix: `GenderEnum`, `OverallStateEnum`
- Model classes end with `Model` suffix: `UsersModel`, `MatchesModel`

**Constants:**
- UPPER_CASE for static const String fields: `static const String Id = 'id';`
- Use `// ignore_for_file: constant_identifier_names` for generated code

**Functions:**
- camelCase for function names: `capName()`, `removeSnake()`
- Private functions prefix with underscore: `_privateFunction()`

### Imports

```dart
// Dart SDK imports first
import 'dart:io';
import 'dart:convert';

// Package imports second (alphabetically)
import 'package:args/args.dart';
import 'package:path/path.dart' as pp;
import 'package:pocketbase/pocketbase.dart';
import 'package:yaml/yaml.dart';

// Relative imports last
import 'src/models.dart';
```

**Import Aliases:**
- Use when needed to avoid conflicts: `import 'package:path/path.dart' as pp;`

### Types & Null Safety

**Explicit Typing:**
```dart
// Always specify types explicitly
final String domain;
final List<CollectionField> fields;
final Map<String, dynamic> data;

// Avoid var when type isn't obvious
String collectionName = 'users';  // Good
var collectionName = 'users';      // Avoid
```

**Nullable Types:**
```dart
// Use ? for optional/nullable fields
final String? name;
final DateTime? deletedAt;
final List<String>? mimeTypes;

// Non-nullable for required fields
final String phoneNumber;
final DateTime createdAt;
```

**Factory Constructors:**
```dart
factory UsersModel.fromModel(RecordModel r) {
  return UsersModel(
    id: r.id,
    created: DateTime.parse(r.get<String>('created')),
    name: r.data['name'],
  );
}
```

### Supported Field Types


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [seifalmotaz/pocketbase_plus](https://github.com/seifalmotaz/pocketbase_plus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
