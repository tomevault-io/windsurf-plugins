---
trigger: always_on
description: This document describes how to build TUI applications with oat-latte.
---

# oat-latte — Agent Reference

This document describes how to build TUI applications with oat-latte.
It is written for AI coding agents that need accurate, complete context to generate correct code without guessing.

---

## Module path

```
github.com/antoniocali/oat-latte
```

Sub-packages:

| Import path | Contents |
|---|---|
| `github.com/antoniocali/oat-latte` | Core interfaces, `Canvas`, `Buffer`, `FocusManager`, geometry types |
| `github.com/antoniocali/oat-latte/latte` | `Style`, `Color`, `BorderStyle`, `Theme`, built-in themes, named color palette |
| `github.com/antoniocali/oat-latte/layout` | `VBox`, `HBox`, `Grid`, `Stack`, `Border`, `Padding`, `VFill`, `HFill`, `FlexChild`, `AlignChild`, `ScrollView`, `VGap`, `HGap` |
| `github.com/antoniocali/oat-latte/widget` | `Text`, `Title`, `Button`, `CheckBox`, `EditText`, `List`, `ComponentList`, `Label`, `ProgressBar`, `StatusBar`, `NotificationManager`, `Dialog`, `Divider` |

---

## Core concepts

### Component

Every element in an oat-latte UI implements `Component`:

```go
type Component interface {
    Measure(c Constraint) Size
    Render(buf *Buffer, region Region)
}
```

The render pipeline is a strict two-pass system:

1. **Measure** — the parent asks the child for its desired size given a `Constraint` (available `MaxWidth`/`MaxHeight`, `-1` means unconstrained).
2. **Render** — the parent hands the child its allocated `Region` and the child draws into a `Buffer`.

Never skip Measure before Render. Never store the `Buffer` or `Region` between frames.

### Layout

A `Component` that holds children implements `Layout`:

```go
type Layout interface {
    Component
    Children() []Component
    AddChild(child Component)
}
```

The framework's tree walkers (theme propagation, focus collection, ID lookup) rely on `Layout.Children()`. Custom container types must implement it.

### Focusable

Interactive components implement `Focusable`:

```go
type Focusable interface {
    Component
    SetFocused(focused bool)
    IsFocused() bool
    HandleKey(ev *KeyEvent) bool // return true = event consumed
}
```

Embed `oat.FocusBehavior` to get `SetFocused`/`IsFocused` for free.

`HandleKey` must return `true` if it consumed the event. Returning `false` tells the canvas to try the next handler (focus cycling, global shortcuts).

### BaseComponent

Embed in every custom component:

```go
type MyWidget struct {
    oat.BaseComponent  // provides ID, Style, FocusStyle, Title, EnsureID(), EffectiveStyle()
    oat.FocusBehavior  // provides SetFocused(), IsFocused()
}
```

Call `e.EnsureID()` in the constructor to auto-assign a unique ID.

`EffectiveStyle(focused bool)` merges `FocusStyle` over `Style` when focused — use this in `Render`.

### Geometry

```go
Size{Width, Height int}                   // desired or allocated size in cells
Region{X, Y, Width, Height int}           // rectangle on screen
Constraint{MaxWidth, MaxHeight int}       // -1 = unconstrained
Insets{Top, Right, Bottom, Left int}      // padding / margin
```

### Anchor

`oat.Anchor` is the **horizontal-axis** positioning type. It is used by `ProgressBar.WithPercentage`, `Border.WithTitle`, and `Divider.WithMaxSize` (for `AxisVertical` dividers).

```go
oat.AnchorLeft    // default — left edge
oat.AnchorCenter  // centred horizontally
oat.AnchorRight   // right edge
```

### VAnchor

`oat.VAnchor` is the **vertical-axis** positioning type. It is used by `Divider.WithMaxSizeV` (for `AxisHorizontal` dividers).

```go
oat.VAnchorTop     // default — top edge
oat.VAnchorMiddle  // centred vertically
oat.VAnchorBottom  // bottom edge
```

The two types are kept separate so APIs that accept horizontal placement cannot accidentally receive a `VAnchor` value and vice versa — the compiler enforces correct axis usage.

### HAlign / VAlign

`oat.HAlign` and `oat.VAlign` are **cross-axis widget positioning** types. They control where a widget sits *within its allocated slot* in a box layout — as opposed to `Anchor`/`VAnchor` which control where a piece of text or a rule sits *within its widget*.

`HAlign` is used by **VBox** (vertical box distributes rows; each child needs horizontal placement):

```go
oat.HAlignFill    // 0 — fill the full allocated width (default, unchanged behaviour)
oat.HAlignLeft    // shrink to desired width, pin left
oat.HAlignCenter  // shrink to desired width, centre horizontally
oat.HAlignRight   // shrink to desired width, pin right
```

`VAlign` is used by **HBox** (horizontal box distributes columns; each child needs vertical placement):

```go
oat.VAlignFill    // 0 — fill the full allocated height (default, unchanged behaviour)
oat.VAlignTop     // shrink to desired height, pin top
oat.VAlignMiddle  // shrink to desired height, centre vertically
oat.VAlignBottom  // shrink to desired height, pin bottom
```

Every built-in widget exposes fluent `WithHAlign` / `WithVAlign` methods that return the concrete widget type, so alignment can be set inline in a builder chain:

```go
btn := widget.NewButton("Save", fn).WithHAlign(oat.HAlignRight)
lbl := widget.NewText("note").WithVAlign(oat.VAlignBottom)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antoniocali/oat-latte](https://github.com/antoniocali/oat-latte) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
