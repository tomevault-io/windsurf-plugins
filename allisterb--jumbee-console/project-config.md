---
trigger: always_on
description: The project Jumbee.Console at @src/Jumbee.Console is a .NET library for building advanced TUIs that focuses on performance and usability. Architecturally, it combines the layout and windowing features from the retained-mode ConsoleGUI library at
---


# About this project
The project Jumbee.Console at @src/Jumbee.Console is a .NET library for building advanced TUIs that focuses on performance and usability. Architecturally, it combines the layout and windowing features from the retained-mode ConsoleGUI library at 
@ext/C-sharp-console-gui-framework/ConsoleGUI/ with the text styling and formatting and rendering and widget features and controls from the immediate-mode Spectre.Console library at @ext/spectre.console/src/Spectre.Console. 

The initial plan created a bridge between the two libraries by implementing `IAnsiConsole` from Spectre.Console in the `AnsiConsoleBuffer` class at @src/Jumbee.Console/AnsiConsoleBuffer.cs to store Spectre.Console control output instead of writing it to the console immediately, 
and a `SpectreControl` class for wrapping Spectre.Console controls as ConsoleGUI `IControl` to be used with ConsoleGUI control and layout classes.
Currently there are many controls which are implemented natively in Jumbee.Console. `SpectreControl` now inherits from the base `Jumbee.Console.Control` class that provides the base functionality for all Jumbee.Console controls that display output and receive user input.
Support for managing the user interface and updating and animating controls is provided by the `UI` class, which runs a single dedicated UI thread via the `Dispatcher`. 
For each frame the dispatcher reads a work queue, dispatches input, and redraws the UI, firing Paint events that controls use to render their state. 
All UI state mutation and rendering happen on this one UI thread, so there is no shared lock. Code on other threads marshals work onto the UI thread via `UI.Invoke`, `UI.Post`, or `UI.InvokeAsync`. 
Control property setters do this marshalling automatically via `SetAtomicProperty`.

Control layout is handled using `Jumbee.Console.Layout` derived classes that wrap ConsoleGUI layout classes. Drawing conflicts from concurrent updates in ConsoleGUI layout classes are mitigated using the UI.Invoke method 
to synchronize concurrent requests for changes to a layout control before redrawing and propagating the changes upward to parent containers.

## Project design and architecture

Read all the markdown docs at @docs/internal/*.md to understand the project architecture and the integration between ConsoleGUI and Spectre.Console in Jumbee.Console.

### Control class
Controls are represented by the common `Jumbee.Console.Control` class. 

### ControlFrame class
A `Control` can have an optional `ControlFrame` object in its `Frame` property. The `ControlFrame` class has a single `Control` as its child and draws borders, margins, scrollbars, a titlebar, and other control adornments around its child control, combining the drawing logic
of ConsoleGUI classes like Border, Margin, and VerticalScrollPanel.

### Layout class
Controls and ControlFrames can be placed in Jumbee.Console.Layout classes for arrangement. This class wraps existing ConsoleGUI layout controls like ConsoleGUI.Controls.Grid.
It implements the ILayout interface which provides a common interface for all layout classes to be used in Jumbee.Console.

### CompositeControl class
A `CompositeControl` is a `Control` that has multiple child Controls arranged in an ILayout The CompositeControl coordinates layout and input handling among its child controls.

### RenderableControl class
The RenderableControl implements Spectre.Console.IRenderable and is designed for new control implementations that want to use the Spectre.Console text styling and layout and rendering
features. It uses an AnsiConsoleBuffer to render to a ConsoleBuffer which is used by ConsoleGUI to draw the control to the console screen.

### SpectreControl class
The SpectreControl class is a generic class that wraps an existing Spectre.Console IRenderable control as a ConsoleGUI IControl. It uses the AnsiConsoleBuffer to render the Spectre control to a buffer, 

### Control implementation considerations
Note the following important considerations when deriving from these classes:

- Any public property or method that changes the visual state of a control must request a redraw: use `SetAtomicProperty` for simple scalar properties (it does the equality check, assign, and invalidate), 
- or call `Invalidate()` directly. `Invalidate()` marks the control dirty so it is re-rendered on the next frame.
- All UI state and rendering live on the single UI thread; there is no UI lock. Do not add locks for UI state. To mutate a control from a background thread, marshal the change onto the UI thread with `UI.Invoke` (inline if already on the UI thread, else posted), `UI.Post`, or `UI.InvokeAsync`.
- Atomic scalar properties may be written directly (via `SetAtomicProperty`), accepting the rare one-frame tear for multi-field structs. Non-atomic mutations (collections, or mutating a wrapped Spectre control via `UpdateContent`) must be marshaled onto the UI thread with `UI.Invoke`; this is what lets controls use a plain `Dictionary`/`List` instead of concurrent collections. Reads of collection state should likewise happen on the UI thread.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [allisterb/Jumbee.Console](https://github.com/allisterb/Jumbee.Console) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
