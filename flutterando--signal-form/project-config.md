---
trigger: always_on
description: A reactive, schema-based, and signal-like form validation and widget management library for Flutter.
---


# Signal Form - AI System Instructions & Skill

`signal_form` is a reactive, declarative, schema-based form management library for Flutter. It uses a signal-like reactivity model, capturing fields dynamically within a form context, validating asynchronously with debouncing, and offering rich extensions and widgets.

---

## 1. Core Architecture & Lifecycle

### Declarative Form Setup
Forms are created using the `formCtrl` builder. Every `Field<T>` instantiated inside the builder is automatically tracked and captured by the returning `FormController<T>`.

```dart
final form = formCtrl(() => (
  username: Field<String>('username', 'initial_value')
      .required(message: 'Username is required')
      .minLength(3, message: 'Too short'),
  email: Field<String>('email')
      .required(message: 'Email is required')
      .email(message: 'Invalid email'),
));
```

### FormController State
- `fields`: Direct, typed reference to the object returned by the builder.
- `valid`: `true` if the form has no validation errors.
- `errors`: A read-only `Map<String, String>` mapping dot-notation field paths to their active error messages.
- `isDirty`: `true` if any field's value differs from its `initialValue`.
- `isTouched`: `true` if any field in the form has been touched (user focused/blurred/interacted).
- `isValidating`: `true` if any async validator is currently running.
- `isSubmitting`: `true` if the `submit` operation is currently executing its callback.

### Field State
- `value`: The typed value of the field. Writing to `value` triggers the validation and listener lifecycle.
- `error`: Active validation error string, or `null`.
- `isTouched`: `true` if the field has been interacted with.
- `isDirty`: `true` if the value differs from `initialValue`.
- `isLoading`: `true` if the field is running async validation.
- `exposedRules`: Lists individual validator rules marked with `exposed: true` along with their current validity state (e.g. for dynamic checklist UIs).

### Validation Modes & Performance
Fields support configuration for when automatic validation runs:
- `ValidationMode.onChange` (Default): Validates on every keypress/value assignment. Can be debounced using `field.debounce(duration)`.
- `ValidationMode.onBlur`: Validates only after the field loses focus for the first time.
- `ValidationMode.onSubmit`: Validates only during form submission.

### Form Lifecycle & Global Reactivity
- **Memory Management (Dispose)**: The `FormController` and all its captured fields retain active listeners. You **MUST** call `form.dispose()` inside the `dispose()` method of your `StatefulWidget` to release resources and avoid memory leaks.
- **Global Reactivity**: `FormController` extends `ChangeNotifier`, implementing the `Listenable` interface. When you need to react to global changes (e.g., disabling a submit button if `form.valid` is `false`, checking if the form is dirty with `form.isDirty`, or displaying a loading spinner with `form.isSubmitting`), wrap the target widgets in a `ListenableBuilder` and pass the `form` controller as the `listenable`.

```dart
ListenableBuilder(
  listenable: form,
  builder: (context, _) {
    return ElevatedButton(
      onPressed: form.valid && !form.isSubmitting
          ? () => form.submit((ctrl) => save(ctrl.toJson()))
          : null,
      child: form.isSubmitting
          ? const CircularProgressIndicator()
          : const Text('Submit'),
    );
  },
)
```

---

## 2. Built-in Validators (Extensions)

All built-in validators return `this` to allow fluent method chaining on `Field`.

### String Validators (`Field<String>`)
- `.required({String message, bool exposed})`: Checks that the value is not null, empty, or whitespace.
- `.notEmpty({String message, bool exposed})`: Checks that trimmed value is not empty (trims value first).
- `.isEmpty({String message, bool exposed})`: Checks that value is empty or null.
- `.minLength(int min, {String message, bool exposed})`
- `.maxLength(int max, {String message, bool exposed})`
- `.length(int count, {String message, bool exposed})`
- `.email({String message, bool exposed})`
- `.validUrl({String message, bool exposed})`: Checks if string is a valid URL format.
- `.httpUrl({String message, bool exposed})`: Checks if string is a valid HTTP/HTTPS URL.
- `.hostname({String message, bool exposed})`: Checks if string is a valid hostname.
- `.pattern(Pattern pattern, {String message, bool exposed})`: Matches a `Pattern` or `RegExp`.
- `.matchesPattern(Pattern pattern, {String message, bool exposed})`: Matches a `Pattern` or `RegExp`.
- `.alphanumeric({String message, bool exposed})`
- `.numeric({String message, bool exposed})`
- `.contains(String substring, {String message, bool exposed})`
- `.startsWith(String prefix, {String message, bool exposed})`
- `.endsWith(String suffix, {String message, bool exposed})`
- `.mustHaveLowercase({String message, bool exposed})`: Requires at least one lowercase character.
- `.mustHaveUppercase({String message, bool exposed})`: Requires at least one uppercase character.
- `.mustHaveNumber({String message, bool exposed})`: Requires at least one digit.
- `.mustHaveNumbers({String message, bool exposed})`: Requires at least one digit.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Flutterando/signal_form](https://github.com/Flutterando/signal_form) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
