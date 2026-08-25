---
trigger: always_on
description: How to write Bonito widgets and apps. Distilled from the reviewed code in
---

# Bonito best practices

How to write Bonito widgets and apps. Distilled from the reviewed code in
Bonito core and BonitoBook (`MonacoEditor` / `EvalEditor` / `CellEditor` /
`FileEditor` / `Book` are the reference implementations). Code-style basics
(naming, no `_` prefixes, multiple dispatch, UPPERCASE consts) live in the
project-level CONVENTIONS.md; this file is about *architecture*: how state,
rendering, and Julia↔JS communication should be organized.

Related developer docs:
- [`skills/electron-test.md`](skills/electron-test.md) — drive a Bonito app in an
  Electron window (open, interact, assert on the DOM, capture logs/screenshots).
- [`js_dependencies/STABILITY_FIXES.md`](js_dependencies/STABILITY_FIXES.md) — the
  JS client stability fixes (findings `J1`–`J15`); the `J#` tags in the JS source
  point here, and `test/reconnect.jl` exercises the reconnect cluster.

## 1. A widget is a struct of Observables

A widget is a plain Julia struct whose mutable state is `Observable`s.
The constructor builds the observables and wires internal listeners.
`Bonito.jsrender(session, widget)` turns it into DOM. All behavior is
exposed as Julia functions taking the widget:

```julia
struct FileEditor
    editor::EvalEditor
    current_file::Observable{String}
end

function open_file!(editor::FileEditor, filepath::String; line::Union{Int,Nothing}=nothing)
    editor.current_file[] = filepath
    set_source!(editor.editor, read(filepath, String))
    isnothing(line) || send(editor.editor; type = "goto-line", line = line)
    toggle!(editor.editor; editor = true)
end
```

The consequences of this shape:

- **The widget is a value.** You can hold it in a field, pass it as an
  argument, compose it into a bigger widget (`CellEditor` wraps
  `EvalEditor` wraps `MonacoEditor`), and drive it from anywhere that has
  a reference — REPL included.
- **Features are functions, not JS snippets.** "Open this file in the
  editor" is `open_file!(editor, path)`, not an `evaljs` that pokes the
  DOM. If you find yourself writing `Bonito.evaljs(session,
  js"window.something.do(...)")` to trigger app behavior from Julia, the
  design is wrong — the behavior belongs on a widget you can call.
- **Rendering is a pure consequence of state.** `jsrender` reads the
  observables and builds DOM that *reacts* to them. It must be safe to
  render the same widget into a fresh session after a reload and end up
  in the same visual state.

Construct observables in the constructor, not in `jsrender`: `jsrender`
runs once per session/tab, so any state created there is per-tab and
silently diverges between tabs. Per-tab state is occasionally right (a
local "is this panel expanded here" toggle) — make that an explicit,
commented choice.

## 2. State ownership: app object down, never globals up

An application has one root object that owns everything (BonitoBook's
`Book`): the widget instances, the domain state, the long-lived
observables. Sub-components receive what they need **as constructor /
function arguments**. Anything you need later is reachable through the
object graph:

```julia
fe = get_file_editor(book)        # book.widgets["file_editor"]
open_file!(fe, path; line = 42)
```

Hard rules:

- **No module-level mutable state for app/widget wiring.** No
  `const REGISTRY = Dict{Session,Foo}()`, no `Ref` caches that exist only
  to avoid passing an argument three layers down. If a function needs the
  editor, it takes the editor (or the app object) as an argument. Globals
  are for static config tables (`const TOOL_ICONS = Dict(...)`), never
  for reaching live objects.
- **No `window.*` namespaces on the JS side either.** A JS-global
  controller (`window._myController.do(...)`) is the JS spelling of the
  same mistake. State lives in JS class instances that were *given* their
  DOM nodes and observables (§4); Julia reaches them through observables,
  not through `evaljs` + global lookup.
- If two distant components must talk, the *app object* mediates: it owns
  both, so it can wire `on(a.something) do ... b ... end` at construction
  time. Distance in the DOM is irrelevant; distance in the object graph
  is what you design.

## 3. Julia ↔ JS communication

Three tiers — use the smallest one that fits:

**a) One value, one direction → a plain Observable.**

```julia
DOM.button("Save"; onclick = js"event => $(save_clicked).notify(true)")
on(session, save_clicked) do _
    save(book)
end
```

Julia → JS side effects use `onjs`:

```julia
onjs(session, button.content, js"x => $(button_dom).innerText = x")
```

**b) A stream of structured messages → one Observable{Dict} per direction
plus a message dispatcher.** This is the `EvalEditor` pattern:

```julia
struct EvalEditor
    js_to_julia::Observable{Dict{String,Any}}
    julia_to_js::Observable{Dict{String,Any}}
    ...
end

send(editor::EvalEditor; msg...) =
    editor.julia_to_js[] = Dict{String,Any}(string(k) => v for (k,v) in pairs(msg))

function process_message(editor::EvalEditor, message::Dict)
    if message["type"] == "new-source"
        ...
    elseif message["type"] == "run"
        run!(editor)
    elseif message["type"] == "multi"
        foreach(m -> process_message(editor, m), message["data"])
    else

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SimonDanisch/Bonito.jl](https://github.com/SimonDanisch/Bonito.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
