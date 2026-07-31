---
trigger: always_on
description: For tasks requiring changing or adding user facing phrases and text parts.
---

# Telegram Desktop Localization

## Coding Style Note

**Use `auto`:** In the actual codebase, variable types are almost always deduced using `auto` (or `const auto`, `const auto &`) rather than being written out explicitly. Examples in this guide may use explicit types for clarity, but prefer `auto` in practice.

```cpp
// Prefer this:
auto currentTitle = tr::lng_settings_title(tr::now);
auto nameProducer = GetNameProducer(); // Returns rpl::producer<...>

// Instead of this:
QString currentTitle = tr::lng_settings_title(tr::now);
rpl::producer<QString> nameProducer = GetNameProducer();
```

## String Resource File

Base user-facing English strings are defined in the `lang.strings` file:

`Telegram/Resources/langs/lang.strings`

This file uses a key-value format with named placeholders:

```
"lng_settings_title" = "Settings";
"lng_confirm_delete_item" = "Are you sure you want to delete {item_name}?";
"lng_files_selected" = "{count} files selected"; // Simple count example (see Pluralization)
```

Placeholders are enclosed in curly braces, e.g., `{name}`, `{user}`. A special placeholder `{count}` is used for pluralization rules.

### Pluralization

For keys that depend on a number (using the `{count}` placeholder), English typically requires two forms: singular and plural. These are defined in `lang.strings` using `#one` and `#other` suffixes:

```
"lng_files_selected#one" = "{count} file selected";
"lng_files_selected#other" = "{count} files selected";
```

While only `#one` and `#other` are defined in the base `lang.strings`, the code generation process creates C++ accessors for all six CLDR plural categories (`#zero`, `#one`, `#two`, `#few`, `#many`, `#other`) to support languages with more complex pluralization rules.

## Translation Process

While `lang.strings` provides the base English text and the keys, the actual translations are managed via Telegram's translations platform (translations.telegram.org) and loaded dynamically at runtime from the API. The keys from `lang.strings` (including the `#one`/`#other` variants) are used on the platform.

## Code Generation

A code generation tool processes `lang.strings` to create C++ structures and accessors within the `tr` namespace. These allow type-safe access to strings and handling of placeholders and pluralization. Generated keys typically follow the pattern `tr::lng_key_name`.

## String Usage in Code

Strings are accessed in C++ code using the generated objects within the `tr::` namespace. There are two main ways to use them: reactively (returning an `rpl::producer`) or immediately (returning the current value).

### 1. Reactive Usage (rpl::producer)

Calling a generated string function directly returns a reactive producer, typically `rpl::producer<QString>`. This producer automatically updates its value whenever the application language changes.

```cpp
// Key: "settings_title" = "Settings";
auto titleProducer = tr::lng_settings_title(); // Type: rpl::producer<QString>

// Key: "confirm_delete_item" = "Are you sure you want to delete {item_name}?";
auto itemNameProducer = /* ... */; // Type: rpl::producer<QString>
auto confirmationProducer = tr::lng_confirm_delete_item( // Type: rpl::producer<QString>
    tr::now, // NOTE: tr::now is NOT passed here for reactive result
    lt_item_name,
    std::move(itemNameProducer)); // Placeholder producers should be moved
```

### 2. Immediate Usage (Current Value)

Passing `tr::now` as the first argument retrieves the string's current value in the active language (typically as a `QString`).

```cpp
// Key: "settings_title" = "Settings";
auto currentTitle = tr::lng_settings_title(tr::now); // Type: QString

// Key: "confirm_delete_item" = "Are you sure you want to delete {item_name}?";
const auto currentItemName = QString("My Document"); // Type: QString
auto currentConfirmation = tr::lng_confirm_delete_item( // Type: QString
    tr::now, // Pass tr::now for immediate value
    lt_item_name, currentItemName); // Placeholder value is a direct QString (or convertible)
```

### 3. Placeholders (`{tag}`)

Placeholders like `{item_name}` are replaced by providing arguments after `tr::now` (for immediate) or as the initial arguments (for reactive). A corresponding `lt_tag_name` constant is passed before the value.

*   **Immediate:** Pass the direct value (e.g., `QString`, `int`).
*   **Reactive:** Pass an `rpl::producer` of the corresponding type (e.g., `rpl::producer<QString>`). Remember to `std::move` the producer or use `rpl::duplicate` if you need to reuse the original producer afterwards.

### 4. Pluralization (`{count}`)

Keys using `{count}` require a numeric value for the `lt_count` placeholder. The correct plural form (`#zero`, `#one`, ..., `#other`) is automatically selected based on this value and the current language rules.

*   **Immediate (`tr::now`):** Pass a `float64` or `int` (which is auto-converted to `float64`).
    ```cpp
    int count = 1;
    auto filesText = tr::lng_files_selected(tr::now, lt_count, count); // Type: QString
    count = 5;
    filesText = tr::lng_files_selected(tr::now, lt_count, count); // Uses "files_selected#other"
    ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aleksanderlm/tdesktop-clean](https://github.com/aleksanderlm/tdesktop-clean) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
