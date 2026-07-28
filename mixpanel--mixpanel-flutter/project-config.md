---
trigger: always_on
description: This document provides specific instructions for generating tests for the Mixpanel Flutter SDK. Follow these patterns to ensure consistency with the existing test suite.
---

# Test Generation Instructions for Mixpanel Flutter SDK

## Overview
This document provides specific instructions for generating tests for the Mixpanel Flutter SDK. Follow these patterns to ensure consistency with the existing test suite.

## Test Structure Pattern

```dart
import 'package:flutter/services.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mixpanel_flutter/mixpanel_flutter.dart';

void main() {
  const MethodChannel channel = MethodChannel('mixpanel_flutter');
  late Mixpanel mixpanel;
  final List<MethodCall> methodCalls = [];

  TestWidgetsFlutterBinding.ensureInitialized();

  setUp(() {
    mixpanel = Mixpanel('YOUR_MIXPANEL_TOKEN');
    methodCalls.clear();
    TestDefaultBinaryMessengerBinding.instance.defaultBinaryMessenger
        .setMockMethodCallHandler(channel, (MethodCall methodCall) async {
      methodCalls.add(methodCall);
      // Return appropriate mock values based on method
      switch (methodCall.method) {
        case 'getDistinctId':
          return 'distinct_id_1';
        case 'getDeviceId':
          return 'device_id_1';
        case 'getAnonymousId':
          return 'anonymous_id_1';
        default:
          return null;
      }
    });
  });

  tearDown(() {
    TestDefaultBinaryMessengerBinding.instance.defaultBinaryMessenger
        .setMockMethodCallHandler(channel, null);
  });

  // Test groups go here
}
```

## Core Test Patterns

### 1. Basic Method Call Test
```dart
test('methodName should invoke platform method with correct arguments', () async {
  await mixpanel.methodName('param1', 'param2');
  
  expect(methodCalls, hasLength(1));
  expect(
    methodCalls[0],
    isMethodCall(
      'methodName',
      arguments: <String, dynamic>{
        'param1': 'param1',
        'param2': 'param2',
      },
    ),
  );
});
```

### 2. Validation Test Pattern
```dart
test('methodName should not invoke platform method with invalid input', () async {
  // Test empty string
  await mixpanel.methodName('');
  expect(methodCalls, isEmpty);

  // Test null (if applicable)
  await mixpanel.methodName(null);
  expect(methodCalls, isEmpty);

  // Test whitespace
  await mixpanel.methodName('   ');
  expect(methodCalls, isEmpty);
});
```

### 3. Map/Dictionary Parameter Test
```dart
test('methodName with properties should format correctly', () async {
  final properties = {'key1': 'value1', 'key2': 123, 'key3': true};
  await mixpanel.methodName('param', properties);
  
  expect(
    methodCalls[0],
    isMethodCall(
      'methodName',
      arguments: <String, dynamic>{
        'param': 'param',
        'properties': properties,
      },
    ),
  );
});
```

### 4. Optional Parameter Test
```dart
test('methodName should handle null optional parameters', () async {
  await mixpanel.methodName('required', null);
  
  expect(
    methodCalls[0],
    isMethodCall(
      'methodName',
      arguments: <String, dynamic>{
        'required': 'required',
        'optional': {},  // SDK converts null maps to empty maps
      },
    ),
  );
});
```

## Specific Patterns for SDK Features

### People/Group Accessor Tests
```dart
test('getPeople should return People instance', () {
  final people = mixpanel.getPeople();
  expect(people, isA<People>());
  expect(people, isNotNull);
});

test('getGroup should return MixpanelGroup instance', () {
  final group = mixpanel.getGroup('groupKey', 'groupId');
  expect(group, isA<MixpanelGroup>());
  expect(group, isNotNull);
});
```

### Super Properties Tests
```dart
test('registerSuperProperties should merge properties correctly', () async {
  await mixpanel.registerSuperProperties({'prop1': 'value1'});
  expect(methodCalls, hasLength(1));
  
  await mixpanel.registerSuperProperties({'prop2': 'value2'});
  expect(methodCalls, hasLength(2));
  
  // Both calls should be registerSuperProperties
  expect(methodCalls[0].method, 'registerSuperProperties');
  expect(methodCalls[1].method, 'registerSuperProperties');
});
```

### Time-based Tests
```dart
test('timeEvent should track event timing', () async {
  await mixpanel.timeEvent('Timed Event');
  
  expect(
    methodCalls[0],
    isMethodCall(
      'timeEvent',
      arguments: <String, dynamic>{
        'eventName': 'Timed Event',
      },
    ),
  );
});
```

## Test Coverage Requirements

Each new method should have tests for:

1. **Happy Path**: Valid inputs produce expected platform calls
2. **Validation**: Invalid inputs (empty strings, null where not allowed) are rejected
3. **Edge Cases**: 
   - Very long strings
   - Special characters in strings
   - Empty collections
   - Null optional parameters
4. **Type Safety**: Different property types (String, int, double, bool, List, Map)

## Common Validation Rules

The SDK validates strings using `_MixpanelHelper.isValidString()`:
- Not null
- Not empty after trimming
- Contains at least one non-whitespace character

Test these cases:
```dart
// Invalid strings that should not trigger platform calls
''          // empty
'   '       // whitespace only
'\t\n'      // whitespace characters

// Valid strings that should trigger platform calls
'a'         // single character

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mixpanel/mixpanel-flutter](https://github.com/mixpanel/mixpanel-flutter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
