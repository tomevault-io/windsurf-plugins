---
trigger: always_on
description: For tasks requiring working with user facing UI components.
---

# Telegram Desktop UI Styling

## Style Definition Files

UI element styles (colors, fonts, paddings, margins, icons, etc.) are defined in `.style` files using a custom syntax. These files are located alongside the C++ source files they correspond to within specific UI component directories (e.g., `Telegram/SourceFiles/ui/chat/chat.style`).

Definitions from other `.style` files can be included using the `using` directive at the top of the file:
```style
using "ui/basic.style";
using "ui/widgets/widgets.style";
```

The central definition of named colors happens in `Telegram/SourceFiles/ui/colors.palette`. This file allows for theme generation and loading colors from various sources.

### Syntax Overview

1.  **Built-in Types:** The syntax recognizes several base types inferred from the value assigned:
    *   `int`: Integer numbers (e.g., `lineHeight: 20;`)
    *   `bool`: Boolean values (e.g., `useShadow: true;`)
    *   `pixels`: Pixel values, ending with `px` (e.g., `borderWidth: 1px;`). Generated as `int` in C++.
    *   `color`: Named colors defined in `colors.palette` (e.g., `background: windowBg;`)
    *   `icon`: Defined inline using a specific syntax (see below). Generates `style::icon`.
    *   `margins`: Four pixel values for margins or padding. Requires `margins(top, right, bottom, left)` syntax (e.g., `margin: margins(10px, 5px, 10px, 5px);` or `padding: margins(8px, 8px, 8px, 8px);`). Generates `style::margins` (an alias for `QMargins`).
    *   `size`: Two pixel values for width and height (e.g., `iconSize: size(16px, 16px);`). Generates `style::size`.
    *   `point`: Two pixel values for x and y coordinates (e.g., `textPos: point(5px, 2px);`). Generates `style::point`.
    *   `align`: Alignment keywords (e.g., `textAlign: align(center);` or `iconAlign: align(left);`). Generates `style::align`.
    *   `font`: Font definitions (e.g., `font: font(14px semibold);`). Generates `style::font`.
    *   `double`: Floating point numbers (e.g., `disabledOpacity: 0.5;`)

    *Note on Borders:* Borders are typically defined using multiple fields like `border: pixels;` (for width) and `borderFg: color;` (for color), rather than a single CSS-like property.

2.  **Structure Definition:** You can define complex data structures directly within the `.style` file:
    ```style
    MyButtonStyle { // Defines a structure named 'MyButtonStyle'
      textPadding: margins; // Field 'textPadding' expects margins type
      icon: icon;         // Field 'icon' of type icon
      height: pixels;     // Field 'height' of type pixels
    }
    ```
    This generates a `struct MyButtonStyle { ... };` inside the `namespace style`. Fields will have corresponding C++ types (`style::margins`, `style::icon`, `int`).

3.  **Variable Definition & Inheritance:** Variables are defined using `name: value;` or `groupName { ... }`. They can be of built-in types or custom structures. Structures can be initialized inline or inherit from existing variables.

    **Icon Definition Syntax:** Icons are defined inline using the `icon{...}` syntax. The generator probes for `.svg` files or `.png` files (including `@2x`, `@3x` variants) based on the provided path stem.
    ```style
    // Single-part icon definition:
    myIconSearch: icon{{ "gui/icons/search", iconColor }};
    // Multi-part icon definition (layers drawn bottom-up):
    myComplexIcon: icon{
      { "gui/icons/background", iconBgColor },
      { "gui/icons/foreground", iconFgColor }
    };
    // Icon with path modifiers (PNG only for flips, SVG only for size):
    myFlippedIcon: icon{{ "gui/icons/arrow-flip_horizontal", arrowColor }};
    myResizedIcon: icon{{ "gui/icons/logo-128x128", logoColor }}; // Forces 128x128 for SVG
    ```

    **Other Variable Examples:**
    ```style
    // Simple variables
    buttonHeight: 30px;
    activeButtonColor: buttonBgActive; // Named color from colors.palette

    // Variable of a custom structure type, initialized inline
    defaultButton: MyButtonStyle {
      textPadding: margins(10px, 15px, 10px, 15px); // Use margins(...) syntax
      icon: myIconSearch; // Assign the previously defined icon variable
      height: buttonHeight; // Reference another variable
    }

    // Another variable inheriting from 'defaultButton' and overriding/adding fields
    primaryButton: MyButtonStyle(defaultButton) {
      icon: myComplexIcon; // Override icon with the multi-part one
      backgroundColor: activeButtonColor; // Add a field not in MyButtonStyle definition
    }

    // Style group (often used for specific UI elements)
    chatInput { // Example using separate border properties and explicit padding
      border: 1px;                          // Border width
      borderFg: defaultInputFieldBorder;    // Border color (named color)
      padding: margins(5px, 10px, 5px, 10px); // Use margins(...) syntax for padding field
      backgroundColor: defaultChatBg;       // Background color
    }
    ```

## Code Generation

A code generation tool processes these `.style` files and `colors.palette` to create C++ objects.
- The `using` directives resolve dependencies between `.style` files.
- Custom structure definitions (like `MyButtonStyle`) generate corresponding `struct MyButtonStyle { ... };` within the `namespace style`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aleksanderlm/tdesktop-clean](https://github.com/aleksanderlm/tdesktop-clean) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
