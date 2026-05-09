---
trigger: always_on
description: Elm developers face a tradeoff when using Tailwind CSS:
---

# Type-Safe Tailwind CSS for Elm: Design Document

## Problem Statement

Elm developers face a tradeoff when using Tailwind CSS:

1. **Native Tailwind (class strings)**: Small bundle size, standard tooling, but no type safety. Typos like `"flexz"` or invalid values like `"text-gray-550"` silently fail.

2. **elm-tailwind-modules + elm-css**: Type-safe, parameterized API (`Tw.p_4`, `Tw.text_color Theme.gray_500`), but includes the elm-css runtime in the bundle and generates CSS at runtime rather than build time.

The core tension: **runtime performance vs. compile-time safety**.

## Goals

1. **Type safety**: Catch typos and invalid values at compile time
2. **Parameterized API**: Write `Tw.p s4` not `Tw.p4`, enabling programmatic styling
3. **Small bundle**: No elm-css runtime, no generated Tailwind Elm modules in the bundle
4. **CSS tree shaking**: Only include CSS for classes actually used
5. **Standard tooling**: Use Tailwind's native CSS generation and tooling

## Existing Solutions

### elm-tailwind-modules (matheus23)

**How it works**: Generates Elm functions that produce `Css.Style` values (elm-css). CSS is generated at runtime. Elm's dead code elimination removes unused functions.

**Pros**:
- Type-safe, parameterized API
- Tree shaking via Elm's DCE

**Cons**:
- elm-css runtime in bundle
- Not using Tailwind's optimized CSS output
- Runtime CSS generation overhead

### Native Tailwind with Class Strings

**How it works**: Write class strings directly: `class "flex p-4 text-gray-500"`

**Pros**:
- Smallest bundle (elm-css fully DCE'd out if unused)
- Build-time CSS generation
- Standard Tailwind tooling and documentation

**Cons**:
- No type safety
- No parameterization
- Typos fail silently

### Typed Constants (Middle Ground)

**How it works**: Define string constants for each class:

```elm
flex : String
flex = "flex"

p4 : String
p4 = "p-4"
```

**Pros**:
- Type safety for typos
- Tailwind can scan the string literals

**Cons**:
- No parameterization (`p4` not `p 4`)
- No CSS tree shaking (all strings present in source)

## Pitfalls and Challenges

### Tailwind's Detection Mechanism

Tailwind scans source files as **plain text** using regex. It does not parse or execute code. From the [Tailwind docs](https://tailwindcss.com/docs/content-configuration):

> "We don't actually parse or execute any of your code in the language it's written in, we just use regular expressions to extract every string that could possibly be a class name."

This means:
- `"p-4"` in source → detected
- `"p-" ++ String.fromInt n` → NOT detected (dynamic)
- Function calls like `Tw.p s4` → NOT detected (no string literal)

### No Custom Extractors in Tailwind v4

Tailwind v4 simplified the API and removed custom extractor support. The only options are:
- Configure which files/directories to scan
- Use `@source inline("...")` to safelist specific classes

### Build Order with Vite

With Vite, Tailwind typically scans source files during the build. For scanning compiled output to work, the build order must be:
1. Compile Elm → JS with class strings as literals
2. Scan compiled JS for classes
3. Generate CSS

This ordering isn't guaranteed by default Vite configuration.

### Dynamic Values

Some usages are truly dynamic and cannot be statically analyzed:

```elm
Tw.p model.userPreferredSpacing  -- Spacing value from runtime
Tw.w (compute x)                  -- Computed at runtime
```

These require either:
- Warning the developer
- Falling back to including all possible values
- Restructuring to use a finite set of options

## Proposed Solution: elm-review Extractor

### Overview

A two-part solution:

1. **Elm Package**: Type-safe, parameterized Tailwind API that generates class strings
2. **elm-review Extractor**: Static analysis that extracts all used/possible classes as JSON

The JSON output feeds into Tailwind's `@source inline()` directive or a scanned safelist file.

### Part 1: Elm Package API

```elm
module Tailwind exposing (..)

-- Opaque type for type safety
type Tailwind = Tailwind String

-- Parameterized spacing using Spacing type
p : Spacing -> Tailwind
p spacing = Tailwind ("p-" ++ spacingToString spacing)

m : Spacing -> Tailwind
px : Spacing -> Tailwind
py : Spacing -> Tailwind
-- etc.

-- Type-safe colors using Color and Shade types
type Shade = S50 | S100 | S200 | ... | S900 | S950

-- Colors are functions that take a Shade
blue : Shade -> Color
gray : Shade -> Color
-- etc.

text_color : Color -> Tailwind
text_color color = Tailwind ("text-" ++ colorToString color)

bg_color : Color -> Tailwind
border_color : Color -> Tailwind

-- Simple colors (white, black, transparent)
type SimpleColor = SimpleColor String

text_simple : SimpleColor -> Tailwind
bg_simple : SimpleColor -> Tailwind

-- Variants (prefix child classes) - in Tailwind.Breakpoints module
hover : List Tailwind -> Tailwind
focus : List Tailwind -> Tailwind
md : List Tailwind -> Tailwind
lg : List Tailwind -> Tailwind

-- Combining multiple classes
classes : List Tailwind -> Attribute msg
```

**Usage**:

```elm
import Tailwind as Tw exposing (classes)
import Tailwind.Theme exposing (blue, gray, s4, s8, s500, s900)
import Tailwind.Breakpoints exposing (hover, md)

view model =
    div
        [ classes
            [ Tw.flex

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dillonkearns/elm-tailwind-classes](https://github.com/dillonkearns/elm-tailwind-classes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
