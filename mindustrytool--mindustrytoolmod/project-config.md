---
trigger: always_on
description: This project is a **Mindustry game mod**, not a backend application.
---

# AGENTS.md

## Project Rules

This project is a **Mindustry game mod**, not a backend application.

Prioritize:

1. Correctness
2. Simple game-mod architecture
3. Clear ownership and lifecycle
4. Declarative Solim UI
5. Automatic reactive bindings
6. Localized structural updates
7. Performance optimization

Do not introduce complexity unless it provides clear value.

---

# Internationalization (i18n) — Mandatory

## Core Rule

**All user-visible text must be translatable.**

Never hardcode display text in Java code unless it is explicitly non-user-facing.

Primary translation bundle:

```text
assets/bundles/bundle.properties
```

This applies to all user-visible text, including:

* Buttons
* Labels
* Menus
* Dialogs
* Tooltips
* Notifications
* Chat and player messages
* Errors and warnings
* Status messages
* Command responses
* Validation messages
* Empty and loading states
* Help text
* Settings descriptions

### Static text

```java
Core.bundle.get("translation.key");
```

### Dynamic text

Use bundle formatting instead of string concatenation:

```java
Core.bundle.format("translation.key", value1, value2);
```

❌ Bad:

```java
player.sendMessage("Welcome, " + player.name + "!");
```

✅ Good:

```properties
# Welcome message displayed to a player.
# {0} is the player's display name.
message.welcome=Welcome, {0}!
```

```java
player.sendMessage(Core.bundle.format("message.welcome", player.name));
```

## Adding Translation Keys

When introducing new user-visible text:

1. Check whether an existing key already represents the same concept.
2. Reuse it when appropriate.
3. Otherwise create a meaningful new key.
4. Add it to `assets/bundles/bundle.properties`.
5. Add a descriptive comment directly above the key.
6. Use the key in code instead of hardcoding text.

### Translation Comments

**Every translation key must have a descriptive comment directly above it.**

Comments should explain:

* What the text means
* Where it is displayed
* When it should be used
* Important context for translators
* Placeholder meanings such as `{0}` and `{1}`

❌ Bad:

```properties
# Save
button.save=Save
```

✅ Good:

```properties
# Displayed on a button that saves the current configuration or changes.
button.save=Save
```

Group comments do **not** replace comments for individual keys.

### Key Naming

Use lowercase, dot-separated keys.

Preferred categories:

```text
ui.*
button.*
menu.*
dialog.*
message.*
error.*
warning.*
status.*
command.*
setting.*
```

Rules:

* Use meaningful names.
* Group keys by feature or purpose.
* Follow existing project conventions.
* Do not create duplicate keys for the same concept.
* Avoid vague names such as `text1`, `label`, or `message2`.

## Exceptions

These generally do not require translation:

* Internal logs
* Debug messages
* Developer-only errors
* Class, method, and variable names
* Configuration keys
* API fields
* Database values
* Protocol and machine-readable strings

If text can be shown to a player or end user, it must be translated.

---

# HTTP Client — Mandatory

All HTTP requests must use `mindustrytool.services.Request`.

Do not construct HTTP clients directly outside `Request.java`, including:

* `HttpURLConnection`
* `URL.openConnection()`
* Other direct HTTP connection APIs

Use either existing facades:

```java
MindustryTool.getSession();
Github.getReleases();
```

Or own a configured `Request` instance:

```java
private final Request api = Request.builder()
        .baseUrl(Config.API_URL)
        .timeout(Duration.ofSeconds(10))
        .authProvider(authProvider)
        .build();
```

Direct connection construction is forbidden outside `Request.java`.

---

# Java Compatibility — Mandatory

## Language vs Runtime

The project supports **Java 17 language syntax** through its compiler/desugaring toolchain, but runs against a **Java 8 runtime environment**.

You may use supported modern language syntax such as:

* `var`
* Switch expressions
* Text blocks

However, **do not use Java standard library APIs introduced after Java 8** unless they are explicitly provided by an included library or backport.

### Common Replacements

| Do not use                             | Use instead                                            |
| -------------------------------------- | ------------------------------------------------------ |
| `java.util.function.*` (`Function`, `Supplier`, `Consumer`, `Predicate`, `BiFunction`, …) | `arc.func.*` (`Func`, `Func2`, `Prov`, `Cons`, `Cons2`, `Boolf`, `Boolf2`) — `java.util.function` default/static methods need D8 desugaring companions (`Function$-CC`) that crash on Android under Mindustry's `ModClassLoader` |
| `List.of()`, `Set.of()`, `Map.of()`    | Java 8 collections, `Arrays.asList()`, Arc collections |
| `stream.toList()`                      | `collect(Collectors.toList())`                         |
| `String.isBlank()`                     | `trim().isEmpty()`                                     |
| `String.strip()`                       | `trim()`                                               |
| `Optional.isEmpty()`                   | `!isPresent()`                                         |
| `Predicate.not()`                      | Lambda                                                 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MindustryTool/MindustryToolMod](https://github.com/MindustryTool/MindustryToolMod) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
