---
trigger: always_on
description: Authoritative guide for AI-assisted work on this repository.
---

# XChart – Copilot Instructions

Authoritative guide for AI-assisted work on this repository.
Keep this file updated as new decisions are made.

---

## Project Overview

XChart is a lightweight Java charting library (`org.knowm.xchart`).
Current version: **4.0.0-SNAPSHOT** | Java target: **11** | Build: **Maven ≥ 3.9.0**

---

## Module Layout

```
xchart/          ← core library (the only deliverable jar)
xchart-demo/     ← standalone demo app; references xchart as a dep
```

---

## Package Structure

| Package | Purpose |
|---------|---------|
| `org.knowm.xchart` | Public API: chart types, series, builders |
| `org.knowm.xchart.internal.chartpart` | Rendering: `Chart`, `Plot_*`, `PlotContent_*`, `Legend_*`, `Axis*` |
| `org.knowm.xchart.internal.series` | Abstract series base classes |
| `org.knowm.xchart.internal.style` | `SeriesColorMarkerLineStyle`, cycler |
| `org.knowm.xchart.style` | Concrete stylers (`PieStyler`, `CategoryStyler`, …) |
| `org.knowm.xchart.style.theme` | Theme implementations |
| `org.knowm.xchart.style.colors` | Color utilities / `FontColorDetector` |
| `org.knowm.xchart.style.lines` | `SeriesLines` |
| `org.knowm.xchart.style.markers` | `SeriesMarkers` |

---

## Naming Conventions (strict — all chart types follow this)

Every chart type `Foo` requires this exact set of classes:

| Role | Class name | Extends |
|------|-----------|---------|
| Chart | `FooChart` | `Chart<FooStyler, FooSeries>` |
| Builder | `FooChartBuilder` | `ChartBuilder<FooChartBuilder, FooChart>` |
| Styler | `FooStyler` | `Styler` or `AxesChartStyler` |
| Series | `FooSeries` | appropriate series base (see below) |
| Plot | `Plot_Foo<ST, S>` | `Plot_<ST, S>` |
| Plot content | `PlotContent_Foo<ST, S>` | `PlotContent_<ST, S>` |
| Legend | `Legend_Foo<ST, S>` | `Legend_<ST, S>` |

Render style enums live **inside** the series class (e.g. `CategorySeries.CategorySeriesRenderStyle`).

---

## Series Hierarchy

```
Series (abstract)
  └─ AxesChartSeries (abstract) – has xMin/xMax/yMin/yMax, stroke, lineColor
       ├─ AxesChartSeriesNumericalNoErrorBars
       └─ AxesChartSeriesCategory – adds xData, yData, errorBars
            └─ CategorySeries, HeatMapSeries, …
  └─ (direct) PieSeries, DialSeries, RadarSeries, …
```

- `Series.DataType` enum: `Number`, `Date`, `String`
- `calculateMinMax()` **must** be called whenever series data changes

---

## Chart Constructor Pattern (all chart types)

```java
// 1. plain size
public FooChart(int width, int height) {
    super(width, height, new FooStyler());
    axisPair = new AxisPair<>(this);   // omit for non-axes charts (Pie, Dial, Radar)
    plot     = new Plot_Foo<>(this);
    legend   = new Legend_Foo<>(this);
}

// 2. custom Theme instance
public FooChart(int width, int height, Theme theme) {
    this(width, height);
    styler.setTheme(theme);
}

// 3. ChartTheme enum
public FooChart(int width, int height, ChartTheme chartTheme) {
    this(width, height, chartTheme.newInstance(chartTheme));
}

// 4. Builder (always delegates to #3)
public FooChart(FooChartBuilder chartBuilder) {
    this(chartBuilder.width, chartBuilder.height, chartBuilder.chartTheme);
    setTitle(chartBuilder.title);
    // set axis titles etc. from builder fields
}
```

---

## Styler Pattern

```java
// Field (private, initialized in setAllStyles())
private SomeType fieldName;

// Getter — blank line before return
public SomeType getFieldName() {

  return fieldName;
}

// Setter — Javadoc with @param, fluent return this
/**
 * One-sentence description.
 *
 * @param fieldName description
 */
public FooStyler setFieldName(SomeType fieldName) {

  this.fieldName = fieldName;
  return this;
}
```

- All fields **must** be initialized in `setAllStyles()`, pulling values from `theme` where applicable.
- `setAllStyles()` must call `super.setAllStyles()` first.
- Never add state to the base `Styler` class unless it truly applies to every chart type.

---

## Code Formatting

- **Formatter**: `fmt-maven-plugin` (google-java-format).
- **Check only**: `mvn fmt:check` (used in CI). Run this to detect violations.
- **NEVER run `mvn fmt:format` project-wide.** It reformats every file, pollutes diffs, and
  causes merge conflicts across all open branches. If a PR has violations, ask the contributor
  to run `mvn fmt:format` on their own changed files only and push a fix commit.
- Indentation: **2 spaces** (google style).
- Imports: sorted by google-java-format; **no wildcard imports** in `src/main`. Tests may use `import static … Assertions.*`.
- One blank line between class members; blank line before `return` in getters.
- **Always** end files with a newline character.

---

## Build & Test

```bash
# Full build + tests
mvn clean verify

# Format source
mvn fmt:format

# Tests only
mvn test -pl xchart

# Skip tests
mvn install -DskipTests
```

### Running demo classes

`xchart-demo` depends on `xchart` via the local Maven repository (not the source tree).
You **must** install `xchart` first, then run the demo:

```bash
mvn install -pl xchart -DskipTests && \
mvn compile exec:java \
  -pl xchart-demo \
  -Dexec.mainClass="org.knowm.xchart.standalone.issues.TestForIssueXXX"
```

Using just `exec:java` without the install step will run with a stale jar and changes won't be visible.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [knowm/XChart](https://github.com/knowm/XChart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
