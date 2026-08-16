---
trigger: always_on
description: Scope: the SVG rendering pipeline, the invariant-culture serialization boundary, headless export, and the byte-pinned gallery goldens. Assumes [csharp.instructions.md](./csharp.instructions.md) applies. This is where GGNet's correctness is *pinned* — small changes here ripple into 40+ snapshot files, so read this before touching anything that emits markup.
---


# Rendering & Goldens Guide

Scope: the SVG rendering pipeline, the invariant-culture serialization boundary, headless export, and the byte-pinned gallery goldens. Assumes [csharp.instructions.md](./csharp.instructions.md) applies. This is where GGNet's correctness is *pinned* — small changes here ripple into 40+ snapshot files, so read this before touching anything that emits markup.

---

## 1. Two-Stage Pipeline: Compose in C#, Emit in Razor

1. **Compose** — `Scene/ShapeComposer.Compose` walks the geom layers and projects them from data space to screen space, producing `ScreenPrimitive`s. **All projection and annotation-label geometry lives here.** It uses a pooled `StringBuilder` (`Microsoft.Extensions.ObjectPool`) to build path `d` strings without per-shape allocation.
2. **Emit** — the `Area` component (`Components/Area.razor`) is a **dumb walker** over the composed `ScreenPrimitive`s: an exhaustive `switch` that maps each variant to an SVG element. It contains no geometry.

Keep the split: never compute coordinates in a component, never emit SVG from the composer. `Shapes/` holds data-space `internal readonly record struct` shapes (`Circle`, `Line`, `Path`, `Polygon`, `MultiPolygon`, …) implementing `IShape`; `Scene/` holds their screen-space `ScreenPrimitive` counterparts. `Buffers/SortedBuffer<T>` is a binary-search insertion-sorted list — a data-structure helper, **not** an SVG writer.

Both `Shape` (data) and `ScreenPrimitive` (screen) are closed **`union`** types — the walker's `switch` has no discard arm, so adding a primitive is a compile error until every walker handles it (`CS8509` = error; see the C# guide §6).

---

## 2. The Invariant-Culture Choke Point

Culture-sensitive numeric formatting produces broken SVG on comma-decimal locales, and `CA1305` is elevated to **error**. Every numeric SVG attribute flows through a single helper, `Components/SvgFormat.cs`:

```csharp
public static string Num(double value)             => value.ToString(CultureInfo.InvariantCulture);
public static string Num(int value)                => value.ToString(CultureInfo.InvariantCulture);
public static string Attr(FormattableString value) => FormattableString.Invariant(value);
```

**Rules:**

- In `.razor` markup, every numeric attribute goes through `@Num(...)` or `@Attr($"...")` — e.g. `transform=@Attr($"translate({circle.X}, {circle.Y})")`, `viewBox=@Attr(...)`. Never interpolate a raw `double`/`int` into markup; it would format under the ambient culture.
- In C# that emits SVG text directly, use `sb.Append(CultureInfo.InvariantCulture, $"…")`, `value.ToString(CultureInfo.InvariantCulture)`, or NodaTime patterns built with `CreateWithInvariantCulture`.
- **Geometry is always invariant.** Only *label text* may be localized, and only through an explicit `IFormatter<T>` passed via a `formatter:` parameter (§5) — this is the layout/paint numeric boundary.

---

## 3. Headless Export (`GGNet.Headless`)

`IPlotContextExtensions` exposes the pure-SVG export surface:

- `AsStringAsync(selfContained = false)` — renders to a `StringWriter`. When `selfContained` is `true`, it injects `<style>…</style>` (from `ThemeCss.SelfContained(theme)`) right after the opening `<svg>` tag so the file renders standalone; off by default because app-hosted output is styled by the app's stylesheet.
- `SaveAsync(path, selfContained = false)` — writes `AsStringAsync` output to a file.

`ThemeCss.SelfContained` loads the embedded theme, re-roots `.ggnet` selectors onto `svg` (`css.Replace(".ggnet", "svg")`), and throws `GGNetUserException` for an unbundled theme.

Rendering without a browser: `Host` builds an empty `ServiceCollection` and creates a fresh `HeadlessRenderer` **per render** (Blazor renderers are single-threaded). `SVGRenderer` walks the `RenderTreeFrame`s directly, finds the `<svg>` element, strips the Blazor scoped-CSS `b-…` marker attributes (regex), and serializes **only** the svg subtree (the surrounding div + loading indicator belong to the interactive experience). It carries `[SuppressMessage("...", "BL0006")]` because it touches RenderTree types; the self-closing element set is `line, circle, rect, path, stop`.

---

## 4. Goldens — Byte-Pinned SVG via Verify.XunitV3

Render-touching output is pinned byte-for-byte. Wiring in `tests/GGNet.Headless.Tests/`:

- **`VerifyConfig.cs`** (`[ModuleInitializer]`): a scrubber replaces nondeterministic plot ids `gg…` → `ggID` via a regex with a negative lookahead that preserves `--ggnet-*` variables. `Verifier.DerivePathInfo` routes `PublicApiTests` snapshots to `Api/` and everything else to `Gallery/`.
- **`GalleryTests.VerifyPlot`**: `var svg = await plot.AsStringAsync(); XDocument.Parse(svg); await Verifier.Verify(svg, extension: "svg");`. `XDocument.Parse` additionally asserts the output is well-formed pure SVG (not an HTML fragment). One `[Fact]` per geom/scenario, one snapshot each — `Gallery/GalleryTests.<Name>.verified.svg` (~40 files).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pablofrommars/GGNet](https://github.com/pablofrommars/GGNet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
