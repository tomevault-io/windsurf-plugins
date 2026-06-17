---
trigger: always_on
description: Context for AI assistants working on this project. Read this first.
---

# CLAUDE.md

Context for AI assistants working on this project. Read this first.

## What this is

A static, browser-only tool that converts JSON to Dart model classes. Inspired by https://javiercbk.github.io/json_to_dart/ but tuned for real-world Flutter codebases — the conventions match how Flutter teams actually write models by hand (named-constructor `fromJson`, nullable fields, defensive parsing, opinionated folder layout).

**Pure static site** — no backend, no build step. Push the folder to GitHub Pages and it runs.

## Tech stack

- HTML + CSS + ES modules (vanilla JS, no framework)
- JSZip (loaded from CDN) for ZIP downloads
- LocalStorage for project persistence
- `<dialog>` element for modals (modern browser only)

## File map

```
index.html              # UI shell — three tabs: Generator / Projects / Docs
styles/
  theme.css             # Color tokens (dark + light themes via [data-theme])
  main.css              # Layout
  components.css        # Buttons, cards, fields, modals, code preview
js/
  parser.js             # JSON value → typed model tree (type inference)
  naming.js             # Class names + folder layout (the api/param rules)
  generator.js          # Tree → Dart files (THE HEART — most logic lives here)
  convertService.js     # Returns the convert_service.dart string template
  downloader.js         # triggerDownload + buildGenerationZip
  projects.js           # localStorage project CRUD + buildProjectZip
  storage.js            # localStorage wrapper (single key: j2d_advanced_v1)
  docs.js               # Markdown docs generator (per-class + index)
  app.js                # UI controller — wires everything to DOM events
```

## Generation conventions (CRITICAL — don't violate)

These are the rules the user explicitly requested. They override default `json_to_dart` behavior.

### Naming

| Mode | Root class | Root folder/file |
|---|---|---|
| `api` (response) | `Api<Name>Model` | `api_<name>/api_<name>_model.dart` |
| `param` (request) | `Pm<Name>Model` | `pm_<name>/pm_<name>_model.dart` |

- Sub-classes are always `<Name>Model` (PascalCase + `Model` suffix).
- File naming: snake_case + `_model.dart`.
- **Folder rule:** a sub-class with its own nested objects gets its own folder under the parent. A leaf sub-class is a flat file in the parent's folder.

### Type inference (matches user's spec)

| JSON | Dart |
|---|---|
| `null`                 | `dynamic` (NOT `Null?`) |
| `[]` or all-null list  | `List<dynamic>?` |
| Mixed-type list        | `List<dynamic>?` |
| `[1, 2.0]`             | `List<double>?` (only int+double promote; everything else mixes → dynamic) |
| Whole number           | `int?` |
| Fractional number      | `double?` |

### Class shape

```dart
class FooModel {
  String? id;          // nullable fields
  String? name;

  FooModel({this.id, this.name});             // default constructor

  FooModel.fromJson(Map<String, dynamic> json) {  // NAMED CONSTRUCTOR, not factory
    id = json['id'];
    name = json['name'];
  }

  Map<String, dynamic> toJson() {
    final data = <String, dynamic>{};
    data['id'] = id;
    data['name'] = name;
    return data;
  }
}
```

- Nested objects in `fromJson`: wrap in `if (json['key'] != null) { field = ChildModel.fromJson(json['key']); }`.
- Lists of objects in `fromJson`: `if (json['key'] != null) { field = <ChildModel>[]; json['key'].forEach((v) { field!.add(ChildModel.fromJson(v)); }); }`.
- Object/list-of-object in `toJson`: ALWAYS wrapped in null check (otherwise `!.toJson()` crashes), regardless of mode.

### Param mode `toJson` — null-safe

In `param` mode, **every** assignment is wrapped in `if (field != null)`. The user wants empty-keyed payloads, never `{ "field": null }`.

### ConvertService option

When enabled:
1. Every `fromJson` primitive assignment uses `ConvertService.convert<Type>(json[...])`.
2. List-of-primitive assignments use `ConvertService.parse<Type>List(json[...])`.
3. Each generated file imports `package:<pkg>/utils/convert_service.dart` (or relative `utils/convert_service.dart` if no package name).
4. The downloaded ZIP includes `utils/convert_service.dart`.

Mapping:
- `String` → `convertString`, `int` → `convertInt`, `double` → `convertDouble`, `num` → `convertNum`, `bool` → `convertBool`
- `List<String>` → `parseStringList`, `List<int>` → `parseIntList`, `List<double>` → `parseDoubleList`, `List<num>` → `parseNumList`, `List<bool>` → `parseBoolList`

The bundled `convert_service.dart` includes nullable variants, `convertDateTime`, `convertMap`, and a generic `parseList<T>(json, fromJson)`. See `js/convertService.js`.

### Capabilities (each toggleable)

Static helpers added inside the class:

- `parseItem(json)` — single-object safe parse, returns `Class()` on failure.
- `parseItems(json)` — handles **both** `[{}, {}]` AND `{ "data": [{}, {}] }`.
- `copyItem(source)` — deep copy via `fromJson(toJson())`.
- `copyItems(list)` — list version.
- `toString()` override — picks first existing field of: `name`, `title`, `label`, `id`; else first String field; else first field.

## Where the rules are enforced

- **Naming + folder layout** → `js/naming.js` (`nameForNode`, `planFileLayout`, `relativeImport`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mahshook-HITEK/json_to_dart_advanced](https://github.com/Mahshook-HITEK/json_to_dart_advanced) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
