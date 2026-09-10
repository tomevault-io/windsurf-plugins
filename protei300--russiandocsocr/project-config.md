---
trigger: always_on
description: Normative for **every** port, not only Go.
---

# Conventions

Normative for **every** port, not only Go.

The Go port is not the goal — it is the executable specification for the .NET,
Kotlin and C++ ports. Every rule here is resolved in favour of "all languages write
this the same way", even where that costs idiomatic Go. The measure of success:
a later port should require **zero design decisions**. If one does, that decision
belonged here.

Read `DEVIATIONS.md` alongside this. Anything you hit that is in neither is a gap in
the design, not a language problem.

---

## 1. Go features to avoid, and what to use instead

| Avoid | Why | Use instead |
|---|---|---|
| `panic`/`recover` as control flow | C#/Kotlin would use exceptions and the shapes diverge immediately | explicit error returns (D-02) |
| bare goroutines + `sync.WaitGroup` | invites a rewrite when transliterated | `errgroup.Group` with `SetLimit`, nothing else |
| channels + `select` as a design element | C# reaches for `Task.WhenAll`, Kotlin for `awaitAll` — three different-looking programs | channels **only** for the lease pool, which maps cleanly to `BlockingCollection`/`Channel` |
| `context.Context` in every signature | `CancellationToken` matches, Kotlin's coroutine context does not | `ctx` only at boundaries: CLI entry, HTTP handler, worker job, `ProcessImg`. Never in pure functions |
| functional options (`WithVerbose(…)`) | idiomatic Go, noisy in both others | an `Options` struct + `DefaultOptions()` → C# record with init props, Kotlin data class with defaults |
| variadic `any` option bags | maps to nothing pleasant | explicit typed context structs (`PostContext{PaddingMeta, ImgShape, Resize, Upsample}`) |
| **embedding as virtual dispatch** | Go embedding is **not** virtual: a "subclass" overriding `nmsIndices` silently calls the base method | flatten the hierarchy (§5) |
| generics beyond `[T any]` on containers | Go's inference gaps produce call sites unlike C#'s and Kotlin's | avoid; a few duplicated 5-line loops beat three different generic designs |
| `iota` integer enums | Kotlin wants `enum class`, C# an `enum`; three serialisations | **string constants** for every tag in `model.json`, and for device, OCR mode and stage names |
| `init()` | no analogue; hidden global setup | an explicit `Load()` called from `main` |
| package-level mutable state | fights DI in the other two | one documented exception: the runtime pool singleton, because Python has one too |
| JSON naming policies / converters | Go, `System.Text.Json` and `kotlinx.serialization` have three different defaults, and ~60 wire names must match byte-for-byte | hand-write **every** `json:"…"` / `[JsonPropertyName]` / `@SerialName` |
| `time.Duration` in wire structs | serialises differently in all three | float seconds, 4 dp, as Python does |
| `map` iteration where order matters | Go randomises; this is **correctness, not style** | `map` for lookup only; anything reaching output goes through an explicit ordered key slice |

That last row is load-bearing: the OCR dict feeds both field ordering and the
service's search text.

---

## 2. The `model.json` tag dispatch

The library is config-driven, and that is the single most portable thing about it.
The same fourteen `model.json` files must drive all four languages **unchanged**.

* DTOs carry the **exact** JSON keys. Every optional numeric is **nullable**
  (`*float64` / `double?` / `Double?`) so "absent" is distinguishable from zero —
  `BlankIndex` legitimately *is* 0 and `Threshold` defaults to 0.5.
* Exactly three functions, one `switch` each. No reflection, no attributes, no DI
  container, no self-registering `init()`:
  * `newPreprocessor(in InputInfo)` — `Classification`, `YOLO`, `YOLOOBB`, `OCR`, `OCRv2`
  * `newPostprocessor(out OutputInfo, workDir string)` — `BinaryClassification`,
    `MultiLabelClassification`, `Metric`, `YOLODetector`, `PerClassYOLODetector`,
    `YOLOOBBDetector`, `YOLOSegmentor`, `OCR`, `OCRFV`, `OCRProbs`
  * `newModel(cfg, pre, sess, post)` — `YOLODetection`, `YOLOSegmentation`,
    `YOLOOBBDetection`, default `UnifiedModel`
* One construction expression per case, **cases in Python's `match` order**, recorded
  in `MAPPING.md`. Go `switch` / C# switch expression / Kotlin `when` then diff
  line-for-line.
* Unknown tag → error naming it (D-06). Unimplemented known tags → wired,
  returning `ErrNotImplemented`.
* **Normalise backslashes.** The shipped artifacts contain
  `"Centers": "resources\\centers.npz"` and `models\Borders` in
  `models_path.yaml` — Windows separators inside *data*. On Linux a backslash is an
  ordinary filename character, so `DocTypeAngles` dies at construction: only in a
  container, never on a Windows dev box. Python normalises in code rather than
  re-shipping the models; every port must do the same.
* Read `model.json` as BOM-free UTF-8 (D-10).

---

## 3. The two parallel groups

Both are structured fan-out/join over a fixed set. The mandated statement shape,
identical in all languages: one launch statement per member **in Python's source
order** → one join → one deterministic collection loop indexed by position → the
concurrent-group timing record.

| | Go | C# | Kotlin |
|---|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [protei300/RussianDocsOCR](https://github.com/protei300/RussianDocsOCR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
