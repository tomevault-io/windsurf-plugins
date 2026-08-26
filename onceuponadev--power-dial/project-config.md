---
trigger: always_on
description: This is a reference and benchmark for GNOME Shell extension developers,
---

# GNOME Extension Best Practices

This is a reference and benchmark for GNOME Shell extension developers,
as well as LLMs that generate GNOME Shell extension code.

::: tip
A [raw Markdown][best-practices-markdown] version of this page is available
to download and use as an AI instructions file if needed.
:::

## Submissions Require Maintainership

Publishing on EGO is an agreement to maintain the extension for GNOME users.

If the author does not know how to read or debug JavaScript, they should keep
the generated extension for personal local use and do not upload it
to [EGO][ego].

When generating extension files, AI models must include this notice:

```js
// Generated with AI for personal use.
// Do NOT upload to extensions.gnome.org (EGO) unless you understand JavaScript
// and can maintain this code.
```

## Follow the Official EGO Guidelines

All generated code must strictly follow
[the official EGO Review Guidelines][review-guidelines].

## Avoid Unnecessary try-catch Wrappers

Do not wrap functions in try-catch blocks if they never throw errors during
normal execution.

Standard methods like `destroy()`, `connect()`, `disconnect()`, `abort()`,
and `GLib.Source.remove()` do not throw unhandled exceptions.

Bad Practice:

```js
if (this._sourceId) {
    try {
        GLib.Source.remove(this._sourceId);
    } catch (e) {
    }
    this._sourceId = null;
}
```

Correct Practice:

```js
if (this._sourceId) {
    GLib.Source.remove(this._sourceId);
    this._sourceId = null;
}
```

## Avoid Unnecessary Checks

Do not use optional chaining (`?.()`) or function type checks (`=== 'function'`)
for guaranteed methods or built-in APIs.

AI models often generate these redundant checks because they try to write code
that works across multiple GNOME Shell versions at once.

Instead, generate clean code for a single targeted GNOME Shell version.
If multi version compatibility is truly necessary, refer to
[the official EGO Port Guide][port-guide].

Bad Practice:

```js
if (typeof TextDecoder === 'function')
    this._textDecoder = new TextDecoder('utf-8');
```

Correct Practice:

```js
this._textDecoder = new TextDecoder('utf-8');
```

Bad Practice:

```js
class Something {
    beep() {
        // ...
    }
    
    boop() {
        if (typeof this.beep === 'function') {
            this.beep();
        }
    }

    pop() {
        this.beep?.();
    }
}
```

Correct Practice:

```js
class Something {
    beep() {
        // ...
    }
    
    boop() {
        this.beep();
    }

    pop() {
        this.beep();
    }
}
```

## Lifecycle and Destruction State

Do not use boolean flags like `this._destroyed` or `this._enabled`
to guard against race conditions or improper lifecycle calls.
After calling `destroy()`, the instance should be nulled out and never used.

On a custom `destroy()` method follow the correct order:

- Remove active timeouts and GLib sources.
- Disconnect all signal handlers.
- Release child references and resources.
- Call `super.destroy()` as the final step.

Bad Practice:

```js
destroy() {
    if (this._destroyed)
        return;
    this._destroyed = true;

    if (this._sourceId) {
        GLib.Source.remove(this._sourceId);
        this._sourceId = null;
    }
    super.destroy();
}
```

Correct practice:

```js
destroy() {
    if (this._sourceId) {
        GLib.Source.remove(this._sourceId);
        this._sourceId = null;
    }
    super.destroy();
}
```

## Widget Destruction vs. Signal Connections

Override `destroy()` directly on GObject widgets rather than connecting
`destroy` signal listener.

Bad practice:

```js
class MyWidget extends St.Widget {
    constructor(params = {}) {
        super(params);
        this._signal = this.connect('destroy', this._onDestroy.bind(this));
    }

    _onDestroy() {
        // Redundant disconnecting destroy signal
        this.disconnect(this._signal);
        // some cleanup here ..
    }
}
```

Correct practice:

```js
class MyWidget extends St.Widget {
    constructor(params = {}) {
        super(params);
    }

    destroy() {
        // some cleanup here ..
        super.destroy();
    }
}
```

## UI Elements: Icons vs. Emojis

- For UI Icons: Use `Gtk.Image` for preferences (prefs.js) and `St.Icon`
    (or `icon_name` properties) for shell UI (extension.js).
    Do not use Unicode emojis as icons.
- For Progress: Use shell components such as `ui.BarLevel` or custom `St.Bin`
    widgets instead of ASCII progress strings (for example, `█░░`).

## Formatting and Line Length

Maintain a maximum line length of 200 characters to ensure readability during
review. This avoids unnecessary horizontal scrolling in the EGO review UI.

## Comments

Write self-explanatory code with clear variable and function names
to make redundant comments unnecessary.

Comments that explain basic JavaScript syntax, describe trivial operations,
or translate code line-by-line into natural language are not allowed.

## Subprocesses and D-Bus Communication

Avoid spawning external shell commands where possible.

Use D-Bus for communication with system services or
external background processes if possible.

Heavy tasks should be offloaded to a separate app as a dependency and
communicating via D-Bus to keep the main GNOME Shell process lightweight.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OnceUponADev/power-dial](https://github.com/OnceUponADev/power-dial) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
