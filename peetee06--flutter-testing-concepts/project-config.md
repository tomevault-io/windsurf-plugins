---
trigger: always_on
description: Use these rules when writing or updating tests or working with mocks
---

# Testing Rules

## General

- Use the dart test tool to run tests
- Use direct class references for group descriptions (e.g., `group(MyClass, () {...})`).
- For provider instances or other objects (e.g., `usernameProvider`), use quoted strings instead (e.g., `group('usernameProvider', () {...})`), as they would otherwise display as "Instance of [...]" in test output.
- ALWAYS write helper methods for pumping the widget under test or reading/listening to providers
- Avoid structural or obvious comments like "Arrange, Act, Assert", "verify xyz"
- Match against full instances instead of just properties when using freezed classes

## Mocking

We're using the `mockito` package for mocking.

### Creating mocks

1. Look at the `mocks.dart` file if the class already has a mock
   1. If yes, use it
   2. If no, add it and run the dart build runner to generate it

If the file does not exist, yet, create it and follow this structure:

```dart

@GenerateMocks([
  SomeClass,
])
class GeneratedMocks {}

```

Avoid adding the mocks to the respective test files. ONLY add them to the `mocks.dart` file.

### Using mocks

Follow this outline to use mocks:

```dart
void main() {
  late MockRestClient client;

  setUp(() {
    client = MockRestClient();

    when(client.someMethod('someParameter')).thenAnswer((_) => 'some result');
  });
}
```

IMPORTANT: Put any helper methods you create in the same scope as the mocks so we don't have to pass them in separately.

## Overriding Riverpod Providers

Use the following convention for overriding the different providers. All examples apply to the `overrides` in `ProviderScope`, as well as `ProviderContainer`:

### Value Providers

```dart
overrides: [
  someValueProvider.overrideWithValue(someValue),
],
```

### Future Providers

Use the Builder pattern to create a value so we can test data, loading and error states of the provider.
The Builder's signature is `FutureOr<ReturnTypeProvider> Function(FutureProviderRef<ReturnTypeOfProvider>)`.

```dart
overrides: [
  someFutureProvider.overrideWith(valueBuilder),
],
```

Examples:  
  
Complete with value:

```dart
FutureOr<String> usernameBuilder(FutureProviderRef<String> ref) => 'username';
// ...
overrides: [
  usernameProvider.overrideWith(usernameBuilder),
],
/// ...
```

Complete with error:

```dart
final error = Exception();
FutureOr<String> usernameBuilder(FutureProviderRef<String> ref) => throw error;
// ...
overrides: [
  usernameProvider.overrideWith(usernameBuilder),
],
/// expect `error` to be thrown
```

Do not complete:

```dart
FutureOr<String> usernameBuilder(FutureProviderRef<String> ref) => Completer<String>().future;
// ...
overrides: [
  usernameProvider.overrideWith(usernameBuilder),
],
/// ...
```

### Notifier Providers

Use `overrideWithBuild` to override the build method of a notifier. This allows testing different states (loading, error, success) without creating fake classes.

The builder's signature is `FutureOr<ReturnType> Function(Ref ref, NotifierType notifier)`.

Use `_` wildcards for unused parameters.

Examples:  
Complete with value:

```dart
overrides: [
  usernameControllerProvider.overrideWithBuild(
    (_, __) => 'username',
  ),
];
```

Complete with error:

```dart
final error = Exception();
// ...
overrides: [
  usernameControllerProvider.overrideWithBuild(
    (_, __) => throw error,
  ),
];
// ...
// expect `error` to be thrown
```

With a delay:

```dart
overrides: [
  usernameControllerProvider.overrideWithBuild(
    (_, __) async {
      await Future<void>.delayed(const Duration(seconds: 1));
      return 'username';
    },
  ),
];
```

Not completing = infinitely in loading state:

```dart
overrides: [
  usernameControllerProvider.overrideWithBuild(
    (_, __) => Completer<String>().future,
  ),
];
```

Testing state transitions (e.g., error then retry succeeds):

Use a list of callbacks with `removeAt(0)` to ensure the build method is called the expected number of times. If called more than expected, the empty list will throw.

```dart
final buildResults = <String Function()>[
  () => throw Exception('Error'),
  () => 'username',
];
// ...
overrides: [
  usernameControllerProvider.overrideWithBuild(
    (_, __) => buildResults.removeAt(0)(),
  ),
];
// ...
// first render shows error
// trigger retry, now shows 'username'
```

### Testing Riverpod family providers

Family providers are tested similarly to other providers. But instead of `container.read(provider)` we have to do `container.read(provider(parameter))`.
For overriding, we do `provider(parameter).overrideWith(...)`. The same goes for `container.listen`.

## Unit Tests

When testing riverpod providers or notifiers, use `ProviderContainer.test()` which automatically disposes the container when the test completes.

### Testing Riverpod value providers

Create a helper method that reads and returns the value of the provider.  
Example:

```dart
String getUsername({required User user}) {
  final container = ProviderContainer.test(
    overrides: [
      userProvider.overrideWithValue(user),
    ],
  );
  return container.read(usernameProvider);
}

test('some test', () {
  final username = getUsername(user: User());
  expect(username, 'username');
});
```

### Testing Riverpod future providers


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Peetee06/flutter-testing-concepts](https://github.com/Peetee06/flutter-testing-concepts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
