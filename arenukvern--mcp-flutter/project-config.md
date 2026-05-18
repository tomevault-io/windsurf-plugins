---
trigger: always_on
description: always use for creating or modifing models, dto or data in .dart files
---


# Dart Extension Type Const Models Guide

## Principle Overview

Extension type const models provide type-safe wrappers around primitive types (String, int, Map) while maintaining zero runtime overhead. They're ideal for creating strongly-typed identifiers, value objects, and data containers that need compile-time safety without performance penalties.

## Required Dependency

Always use the `from_json_to_json` package for type-safe JSON handling:

```yaml
dependencies:
  from_json_to_json: ^0.3.0
```

Import in your Dart files:

```dart
import 'package:from_json_to_json/from_json_to_json.dart';
```

## Core Pattern Structure

### Basic String-based ID Pattern

```dart
/// Extension type that represents a [specific domain concept].
///
/// [Brief description of purpose and usage context]
///
/// Provides functionality to handle JSON serialization/deserialization
/// and [specific domain operations].
extension type const EntityId(String value) {
  factory EntityId.fromJson(final value) => EntityId(jsonDecodeString(value));

  String toJson() => value;

  bool get isEmpty => value.isEmpty;
  bool get isNotEmpty => value.isNotEmpty;

  EntityId whenEmptyUse(final EntityId other) => isEmpty ? other : this;

  static const empty = EntityId('');
}
```

### Numeric ID Pattern

```dart
extension type const NumericId(int value) {
  factory NumericId.fromJson(final value) => NumericId(jsonDecodeInt(value));

  int toJson() => value;

  bool get isZero => value == 0;
  bool get isPositive => value > 0;

  NumericId whenZeroUse(final NumericId other) => isZero ? other : this;

  static const zero = NumericId(0);
}
```

### Complex Data Structure Pattern

```dart
/// Extension type that wraps [Map/List/other complex type].
///
/// [Description of the data structure purpose]
extension type const DataModel._(Map<String, dynamic> value) {

  factory DataModel({
    required final String specificField,
    required final int numericField,
    required final List<String> listField,
    required final Map<String, int> mapField,
    required final DateTime dateField,
  }) => DataModel._({
    'field_name': specificField,
    'numeric_field': numericField,
    'list_field': listField,
    'map_field': mapField,
    'date_field': dateField,
  });
  factory DataModel.fromJson(final dynamic json) =>
      DataModel._(jsonDecodeMapAs(json));

  // Provide domain-specific getters using from_json_to_json functions
  String get specificField => jsonDecodeString(value['field_name']);
  int get numericField => jsonDecodeInt(value['numeric_field']);
  List<String> get listField => jsonDecodeListAs<String>(value['list_field']);
  Map<String, int> get mapField => jsonDecodeMapAs<String, int>(value['map_field']);
  DateTime? get dateField => dateTimeFromIso8601String(jsonDecodeString(value['date_field']));
  Duration get durationField => jsonDecodeDurationInSeconds(value['duration_seconds']);

  Map<String, dynamic> toJson() => value;

  static const empty = DataModel({});
}
```

### List-based Model Pattern

```dart
extension type const ItemsList(List<dynamic> value) {
  factory ItemsList.fromJson(final dynamic jsonData) {
    final list = jsonDecodeList(jsonData);
    return ItemsList(list);
  }

  // Type-safe getters
  List<String> get asStrings => jsonDecodeListAs<String>(value);
  List<int> get asInts => jsonDecodeListAs<int>(value);

  bool get isEmpty => value.isEmpty;
  bool get isNotEmpty => value.isNotEmpty;
  int get length => value.length;

  List<dynamic> toJson() => value;

  static const empty = ItemsList([]);
}
```

## Implementation Guidelines

### 1. **Naming Convention**

- Use descriptive names ending with `Id` for identifiers: `BookId`, `ActorId`, `VideoSeriesId`
- Use descriptive names ending with `Model` for complex data: `PricesOverridesModel`
- Use domain-specific names for specialized types: `SessionNumber`, `QuizResults`

### 2. **Required Methods**

- **`fromJson` factory**: Handle JSON deserialization using `from_json_to_json` functions
- **`toJson` method**: Return the underlying value for serialization
- **`empty`/`zero` static const**: Provide a default empty instance

### 3. **JSON Handling with from_json_to_json**

Use the appropriate decode functions from the package:

| Type     | Function                                    | Usage                                       |
| -------- | ------------------------------------------- | ------------------------------------------- |
| String   | `jsonDecodeString(value)`                   | Safe string conversion, empty if null       |
| int      | `jsonDecodeInt(value)`                      | Safe int conversion, 0 if invalid           |
| double   | `jsonDecodeDouble(value)`                   | Safe double conversion, 0.0 if invalid      |
| bool     | `jsonDecodeBool(value)`                     | Safe bool conversion, false if invalid      |
| List     | `jsonDecodeList(value)`                     | Safe list conversion, empty list if invalid |
| List<T>  | `jsonDecodeListAs<T>(value)`                | Type-safe list conversion                   |
| Map      | `jsonDecodeMap(value)`                      | Safe map conversion, empty map if invalid   |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Arenukvern/mcp_flutter](https://github.com/Arenukvern/mcp_flutter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
