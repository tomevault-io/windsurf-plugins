---
trigger: always_on
description: **CNA** is a C++ reimplementation of the XNA 4.0 programming model, built on SDL3 and a pluggable graphics backend
---

# CNA Project Guidelines — FNA to C++ Porting Instructions

## Project Overview

**CNA** is a C++ reimplementation of the XNA 4.0 programming model, built on SDL3 and a pluggable graphics backend
layer. It is a framework/runtime and abstraction layer — not a game — designed to preserve XNA-style APIs
(`Microsoft::Xna::Framework`) while using modern C++23 internals.

### Source Reference

The authoritative behavioral and API reference is the local FNA source tree:

```text
/rv/data/library/github.com/FNA-XNA/FNA
```

Do **not** treat old CNA code or AI-generated stubs as authoritative if they conflict with the FNA reference API.

---

## Code Generation Rules

### XNA 4.0 API Compliance

When implementing code in the `Microsoft::Xna` namespace:

- **MUST** strictly adhere to the XNA 4.0 API specification as implemented in FNA.
- **MUST** preserve original XNA 4.0 class names, method signatures, and behavior.
- **MUST** use modern C++23 internally while maintaining XNA-style public APIs.
- If implementing functionality that is **NOT** part of the XNA 4.0 API within the `Microsoft::Xna` namespace,
  you **MUST** wrap it with the `NOXNA` macro.

`NOXNA` is defined in `include/CNA/CNAHelper.hpp` as an empty marker macro used to visually tag non-XNA extensions.

---

## Namespaces

Original XNA types must stay in the matching XNA namespace:

```cpp
Microsoft::Xna::Framework::Color
Microsoft::Xna::Framework::Vector3
Microsoft::Xna::Framework::Graphics::Texture2D
Microsoft::Xna::Framework::Graphics::SpriteBatch
```

The `CNA` namespace is for project-specific extensions, helpers, internal backends, and non-XNA additions only.
Do not move original XNA API types into the `CNA` namespace.

---

## API Names Must Match XNA/FNA

Class names, struct names, enum names, method names, operator names, and constant/static member names must
match the XNA/FNA API exactly.

Do not rename API members to make them more C++-like if that would diverge from XNA/FNA.

---

## C# Properties → C++ Convention

C# properties use the established CNA convention:

```cpp
getXProperty()   // getter
setXProperty(…)  // setter
```

Example:

```csharp
// C#
public byte R { get; set; }
```

```cpp
// C++
[[nodiscard]] bytecs getRProperty() const;
void setRProperty(bytecs value);
```

Do not replace C# properties with public fields unless the type already establishes that style.

---

## Type Aliases (SharpRuntime)

When C# source uses .NET primitive type names, preserve the corresponding alias from `SharpRuntime`:

| C# type   | SharpRuntime alias     | Underlying C++ type |
|-----------|------------------------|---------------------|
| `byte`    | `bytecs` / `Byte`      | `uint8_t`           |
| `sbyte`   | `sbytecs` / `SByte`    | `int8_t`            |
| `short`   | `shortcs` / `Int16`    | `int16_t`           |
| `ushort`  | `ushortcs` / `UInt16`  | `uint16_t`          |
| `int`     | `intcs` / `Int32`      | `int32_t`           |
| `uint`    | `uintcs` / `UInt32`    | `uint32_t`          |
| `long`    | `longcs` / `Int64`     | `int64_t`           |
| `ulong`   | `ulongcs` / `UInt64`   | `uint64_t`          |
| `float`   | `Single`               | `float`             |
| `string`  | `String`               | `std::string`       |
| `char`    | `charcs`               | `char16_t`          |

Include `"SharpRuntime/SharpRuntimeHelper.hpp"` to access these. If a needed alias does not yet exist, add
a minimal stub/alias in `sharp-runtime` rather than using a raw C++ type directly in the XNA API surface.

---

## Events

C# events and delegates are modeled through `System::EventHandler<TEventArgs>`:

```cpp
// Declaration (in class, matches C# "public event EventHandler<T> Name;")
System::EventHandler<ExitingEventArgs> Exiting;

// Subscription
Exiting += [](System::Object* sender, const ExitingEventArgs& e) { … };

// Raising
Exiting.Raise(this, args);
// or
Exiting.Invoke(this, args);
```

`EventHandler<T>` stores subscribed callbacks and exposes `Raise()` / `Invoke()`. This is the project-wide
pattern; do not invent a different event mechanism.

---

## Interfaces and Inheritance

Preserve C# interface relationships as C++ abstract base classes.

```csharp
// C#
class Color : IEquatable<Color>, IPackedVector, IPackedVector<uint> { … }
```

```cpp
// C++
struct Color : public Graphics::PackedVector::IPackedVectorT<UInt32> { … };
```

If an exact mapping is not practical, implement equivalent behavior and document the intentional deviation
in the PR description or task report — not in source comments.

---

## IDisposable

C# `IDisposable` is mapped to `System::IDisposable` (from sharp-runtime):

```cpp
class Foo : public System::IDisposable {
public:
    void Dispose() override;
protected:
    void Dispose(bool disposing);   // only when the pattern requires it
};
```

Always check `isDisposed_` before acting; throw `std::runtime_error` if used after disposal.

---

## Visibility Mapping

Map C# visibility intentionally — do not make every member public:

| C#        | C++                                                              |
|-----------|------------------------------------------------------------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openeggbert/cna](https://github.com/openeggbert/cna) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-04 -->
