---
trigger: always_on
description: validatesPresenceOf("name", message="Required");
---

# Wheels Framework

CFML MVC framework with ActiveRecord ORM. The framework itself lives in `vendor/wheels/` (NOT a dependency — this repo IS the framework). The repo also contains a demo app under `app/` you can hand-test against.

## Code Map (where things live)

```
vendor/wheels/                  Framework core (model/, controller/, view/, dispatch/, migrator/, middleware/, …)
vendor/wheels/tests/specs/      Framework test suite — what CI runs across every engine × DB
app/                            Demo app (models, controllers, views, migrations) — exercise framework changes here
tests/specs/                    Demo-app test suite (separate from the framework suite)
cli/lucli/                      The `wheels` binary — branded LuCLI runtime + Module.cfc (MCP tools)
cli/lucli/services/deploy/      `wheels deploy` (Kamal port — see .ai/wheels/deploy.md)
cli/lucli/tests/specs/          CLI test suite
config/settings.cfm             Demo-app config (routes.cfm, environment.cfm, services.cfm-if-present)
plugins/                        DEPRECATED — legacy plugin system; modern packages live in vendor/<name>/
.ai/wheels/                     Deep reference docs Claude searches when needed
.claude/commands/               Wheels-bot prompts (.github/workflows/bot-*.yml runs these)
```

**Branding:** the project name is **Wheels** (not "CFWheels"). The rebrand happened at v3.0. Use "Wheels" in code, comments, commits, PRs, and docs.

## Before Reporting a Change Complete

| If you touched | Run | Required? |
|---|---|---|
| `vendor/wheels/**` | `bash tools/test-local.sh` (full) or `bash tools/test-local.sh <area>` | Always |
| `app/**` only | Demo-app specs via `wheels test` | Always |
| `cli/lucli/**` | `bash tools/test-cli-local.sh` | Always |
| Anything cross-engine-risky (closures, `obj.map()`, reserved scopes, struct literals, mixins) | `tools/test-matrix.sh adobe2023 mysql` AND `tools/test-matrix.sh lucee7 mysql` | If touched code matches any anti-pattern below |
| Added/changed a migration | `wheels migrate latest && wheels migrate down && wheels migrate up` | Always |
| Changed a public framework API | `grep -r` callers under `vendor/wheels`, `app`, `tests`, `cli/lucli/tests` | Always |

Type checks and a green test suite verify *code correctness*. They do NOT verify *feature correctness* for UI changes — if you changed a view/form/route, hand-test it in a browser or say so explicitly.

## Cross-Engine Invariants (apply to every change in `vendor/wheels/`)

The framework must run on Lucee 5/6/7, Adobe CF 2018/2021/2023/2025, and BoxLang. These rules cause more bugs than anything else combined.

1. **`obj.map()` resolves to the built-in struct member function** on Lucee/Adobe — not your CFC method. Use `mapInstance()` on the Injector, or rename your method.
2. **`application` scope doesn't accept function members on Adobe CF.** Pass a plain struct context instead.
3. **Closure `this` captures the declaring scope** — use `var ctx = {ref: obj}` to share references across closures.
4. **`obj["key"]()` inside closures crashes Adobe CF 2021/2023's parser.** Split: `var fn = obj["key"]; fn();`.
5. **Inline closure as constructor named arg** (`new Foo(callback = function(){...})`) crashes Adobe CF with `ArrayStoreException: ASTcffunction`. **Worse: it takes down the entire TestBox bundle** because `getComponentMetadata()` triggers eager compilation. Hoist: `var fn = function(){...}; new Foo(callback = fn);`.
6. **Adobe CF copies arrays by value in struct literals.** `{arr = myArray}` then mutating `arr` inside a closure won't affect the original. Use parent struct ref: `{owner = parentStruct}` then `owner.arr`.
7. **`private` mixin functions are not integrated.** `$integrateComponents()` only copies `public` methods into model/controller objects. ALL helpers in `vendor/wheels/model/*.cfc`, view helpers, etc. MUST use `public` access with `$` prefix for internal scope. BoxLang passes; Lucee/Adobe fail.
8. **`Left(str, 0)` crashes Lucee 7.** Guard: `len > 0 ? Left(str, len) : ""`.
9. **`toBeInstanceOf("component")` fails on BoxLang** — returns the FQN, not the literal `"component"`. Use `toBeWheelsModel()` for finder results.
10. **Adobe CF 2023 and 2025 reject the `arguments` scope as `attributeCollection` on *any* built-in CFML tag.** Affects every `cfheader` / `cfcache` / `cfcontent` / `cfmail` / `cfdirectory` / `cffile` / `cflocation` / `cfhtmlhead` / `cfimage` / `cfdbinfo` / `cfinvoke` / `cfwddx` / `cfzip` wrapper. Covers both the string-interpolated (`attributeCollection = "#arguments#"`) and direct-struct (`attributeCollection = arguments`) forms. Adobe 2023/2025 throw — `cfheader`'s message is `"Failed to add HTML header"`; other tags surface their own — and `$header()` is catastrophic because it runs on every request. Copy to a plain struct first: `local.args = {}; for (local.key in arguments) { local.args[local.key] = arguments[local.key]; }`. Lucee 6/7, BoxLang, and Adobe 2018/2021 accept both forms; Adobe 2023/2025 require the plain struct. The 13 sites in `vendor/wheels/Global.cfc` were patched uniformly in [#2750](https://github.com/wheels-dev/wheels/pull/2750).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wheels-dev/wheels](https://github.com/wheels-dev/wheels) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
