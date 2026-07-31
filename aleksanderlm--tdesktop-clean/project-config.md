---
trigger: always_on
description: **Use `auto`:** In the actual codebase, variable types are almost always deduced using `auto` (or `const auto`, `const auto &`) rather than being written out explicitly. Examples in this guide may use explicit types for clarity, but prefer `auto` in practice.
---

# RPL (Reactive Programming Library) Guide

## Coding Style Note

**Use `auto`:** In the actual codebase, variable types are almost always deduced using `auto` (or `const auto`, `const auto &`) rather than being written out explicitly. Examples in this guide may use explicit types for clarity, but prefer `auto` in practice.

```cpp
// Prefer this:
auto intProducer = rpl::single(123);
const auto &lifetime = existingLifetime;

// Instead of this:
rpl::producer<int> intProducer = rpl::single(123);
const rpl::lifetime &lifetime = existingLifetime;

// Sometimes needed if deduction is ambiguous or needs help:
auto user = std::make_shared<UserData>();
auto data = QByteArray::fromHex("...");
```

## Introduction

RPL is the reactive programming library used in this project, residing in the `rpl::` namespace. It allows handling asynchronous streams of data over time.

The core concept is the `rpl::producer`, which represents a stream of values that can be generated over a certain lifetime.

## Producers: `rpl::producer<Type, Error = no_error>`

The fundamental building block is `rpl::producer<Type, Error>`. It produces values of `Type` and can optionally signal an error of type `Error`. By default, `Error` is `rpl::no_error`, indicating that the producer does not explicitly handle error signaling through this mechanism.

```cpp
// A producer that emits integers.
auto intProducer = /* ... */; // Type: rpl::producer<int>

// A producer that emits strings and can potentially emit a CustomError.
auto stringProducerWithError = /* ... */; // Type: rpl::producer<QString, CustomError>
```

Producers are typically lazy; they don't start emitting values until someone subscribes to them.

## Lifetime Management: `rpl::lifetime`

Reactive pipelines have a limited duration, managed by `rpl::lifetime`. An `rpl::lifetime` object essentially holds a collection of cleanup callbacks. When the lifetime ends (either explicitly destroyed or goes out of scope), these callbacks are executed, tearing down the associated pipeline and freeing resources.

```cpp
rpl::lifetime myLifetime;
// ... later ...
// myLifetime is destroyed, cleanup happens.

// Or, pass a lifetime instance to manage a pipeline's duration.
rpl::lifetime &parentLifetime = /* ... get lifetime from context ... */;
```

## Starting a Pipeline: `rpl::start_...`

To consume values from a producer, you start a pipeline using one of the `rpl::start_...` methods. These methods subscribe to the producer and execute callbacks for the events they handle.

The most common method is `rpl::start_with_next`:

```cpp
auto counter = /* ... */; // Type: rpl::producer<int>
rpl::lifetime lifetime;

// Counter is consumed here, use std::move if it's an l-value.
std::move(
    counter
) | rpl::start_with_next([=]\(int nextValue) {
    // Process the next integer value emitted by the producer.
    qDebug() << "Received: " << nextValue;
}, lifetime); // Pass the lifetime to manage the subscription.
// Note: `counter` is now in a moved-from state and likely invalid.

// If you need to start the same producer multiple times, duplicate it:
// rpl::duplicate(counter) | rpl::start_with_next(...);

// If you DON'T pass a lifetime to a start_... method:
auto counter2 = /* ... */; // Type: rpl::producer<int>
rpl::lifetime subscriptionLifetime = std::move(
    counter2
) | rpl::start_with_next([=]\(int nextValue) { /* ... */ });
// The returned lifetime MUST be stored. If it's discarded immediately,
// the subscription stops instantly.
// `counter2` is also moved-from here.
```

Other variants allow handling errors (`_error`) and completion (`_done`):

```cpp
auto dataStream = /* ... */; // Type: rpl::producer<QString, Error>
rpl::lifetime lifetime;

// Assuming dataStream might be used again, we duplicate it for the first start.
// If it's the only use, std::move(dataStream) would be preferred.
rpl::duplicate(
    dataStream
) | rpl::start_with_error([=]\(Error &&error) {
    // Handle the error signaled by the producer.
    qDebug() << "Error: " << error.text();
}, lifetime);

// Using dataStream again, perhaps duplicated again or moved if last use.
rpl::duplicate(
    dataStream
) | rpl::start_with_done([=] {
    // Execute when the producer signals it's finished emitting values.
    qDebug() << "Stream finished.";
}, lifetime);

// Last use of dataStream, so we move it.
std::move(
    dataStream
) | rpl::start_with_next_error_done(
    [=]\(QString &&value) { /* handle next value */ },
    [=]\(Error &&error) { /* handle error */ },
    [=] { /* handle done */ },
    lifetime);
```

## Transforming Producers

RPL provides functions to create new producers by transforming existing ones:

*   `rpl::map`: Transforms each value emitted by a producer.
    ```cpp
    auto ints = /* ... */; // Type: rpl::producer<int>
    // The pipe operator often handles the move implicitly for chained transformations.
    auto strings = std::move(
        ints // Explicit move is safer
    ) | rpl::map([](int value) {
        return QString::number(value * 2);
    }); // Emits strings like "0", "2", "4", ...

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aleksanderlm/tdesktop-clean](https://github.com/aleksanderlm/tdesktop-clean) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
