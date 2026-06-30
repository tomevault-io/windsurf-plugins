---
trigger: always_on
description: `flutter_turtle` is a Flutter package that implements [turtle graphics](https://en.wikipedia.org/wiki/Turtle_graphics) using a [Logo](https://en.wikipedia.org/wiki/Logo_(programming_language))-inspired DSL (Domain-Specific Language). It uses a `CustomPainter` to draw graphics on a canvas by interpreting a series of command objects.
---

# GitHub Copilot Instructions for flutter_turtle

## Project Overview

`flutter_turtle` is a Flutter package that implements [turtle graphics](https://en.wikipedia.org/wiki/Turtle_graphics) using a [Logo](https://en.wikipedia.org/wiki/Logo_(programming_language))-inspired DSL (Domain-Specific Language). It uses a `CustomPainter` to draw graphics on a canvas by interpreting a series of command objects.

The package is published on [pub.dev](https://pub.dev/packages/flutter_turtle) and the source is hosted at https://github.com/zonble/flutter_turtle.

## Repository Structure

```
flutter_turtle/
├── lib/
│   ├── flutter_turtle.dart          # Main library entry point (re-exports everything)
│   ├── flutter_turtle_alias.dart    # Convenience aliases
│   └── src/
│       ├── animated_turtle_view.dart    # AnimatedTurtleView widget
│       ├── controllable_turtle_view.dart # ControllableTurtleView widget
│       ├── painter.dart                 # TurtlePainter (CustomPainter) and PaintContext
│       ├── turtle_commands.dart         # TurtleCommand and Instruction interfaces + TurtleCompiler
│       ├── turtle_state.dart            # TurtleState and Macro classes
│       ├── turtle_view.dart             # TurtleView widget
│       └── commands/
│           ├── _exceptions.dart         # Internal exceptions (StopException)
│           ├── _instructions.dart       # Internal instruction implementations
│           ├── commands.dart            # Barrel file re-exporting all commands
│           ├── back.dart                # Back command
│           ├── forward.dart             # Forward command
│           ├── go_to.dart               # GoTo command
│           ├── if.dart                  # If command
│           ├── if_else.dart             # IfElse command
│           ├── label.dart               # Label command
│           ├── left.dart                # Left (turn) command
│           ├── log.dart                 # Log command
│           ├── pen_down.dart            # PenDown command
│           ├── pen_up.dart              # PenUp command
│           ├── pop_state.dart           # PopState command
│           ├── repeat.dart              # Repeat command
│           ├── reset_heading.dart       # ResetHeading command
│           ├── reset_position.dart      # ResetPosition command
│           ├── right.dart               # Right (turn) command
│           ├── run_macro.dart           # RunMacro command
│           ├── save_state.dart          # SaveState command
│           ├── set_color.dart           # SetColor command
│           ├── set_label_height.dart    # SetLabelHeight command
│           ├── set_macro.dart           # SetMacro command
│           ├── set_orientation.dart     # SetOrientation command
│           ├── set_stroke_width.dart    # SetStrokeWidth command
│           └── stop.dart                # Stop command
├── example/                         # Example Flutter app
├── test/
│   └── flutter_turtle_test.dart     # Test file
├── analysis_options.yaml            # Dart linter configuration (uses flutter_lints)
└── pubspec.yaml                     # Package metadata and dependencies
```

## Architecture

The package uses a two-phase pipeline:

### Phase 1 – Compilation (`TurtleCompiler.compile`)

A list of `TurtleCommand` objects is compiled into a flat list of `Instruction` objects. During compilation, `TurtleState` is used to track the turtle's position, heading, pen state, color, stroke width, macros, and the turtle stack. This phase resolves all control-flow (loops, conditionals, macros) eagerly.

### Phase 2 – Rendering (`TurtlePainter.paint`)

The flat list of `Instruction` objects is executed sequentially against a `PaintContext` (which holds a Flutter `Canvas`, a `Paint`, and the center `Offset`). Instructions perform actual drawing on the canvas.

### Key types

| Type | Role |
|------|------|
| `TurtleCommand` | Abstract interface. Implements `createInstruction(TurtleState, Map) → List<Instruction>`. Represents a high-level turtle directive (e.g. `Forward`, `Right`). |
| `Instruction<T>` | Abstract interface. Implements `exec(PaintContext) → T`. Represents a low-level drawing operation. |
| `TurtleState` | Mutable state used **only during compilation**. Holds position, heading (degrees), pen status, color, stroke width, macros, and a stack for `SaveState`/`PopState`. |
| `PaintContext` | Immutable context passed to every `Instruction.exec` call. Contains `Canvas`, `Paint`, and the canvas center `Offset`. |
| `TurtleCompiler` | Static utility that runs the compilation phase. |
| `TurtlePainter` | `CustomPainter` subclass that executes instructions in `paint()`. |
| `TurtleView` | Stateful `Widget` that compiles and renders a list of commands without animation. |
| `AnimatedTurtleView` | Stateful `Widget` that renders commands with a built-in animation driven by an internal `AnimationController`. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zonble/flutter_turtle](https://github.com/zonble/flutter_turtle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
