---
trigger: always_on
description: `hck` scaffolds Helm charts and adds resources to charts that already exist.
---

# CLAUDE.md — helm-chart-kit

`hck` scaffolds Helm charts and adds resources to charts that already exist.

<br/>

## Build & Test

```bash
make build           # Build binary → ./bin/hck
make test            # go test ./... -v -race -cover
make cover           # Coverage report
make fmt             # go fmt
make vet             # go vet
golangci-lint run    # Lint (config in .golangci.yml)
```

`helm` must be on `PATH` for `hck check` and the tests that exercise it. Those tests skip when it is absent, so a green run without helm does not mean the render path was covered.

<br/>

## Project Structure

```
cmd/cli/            Cobra commands, one constructor each
internal/catalog/   Resources and presets — data only
internal/render/    Embedded templates + renderer
  templates/chart/      Chart skeleton
  templates/resources/  One directory per resource
internal/values/    Append-only values.yaml merge
internal/schema/    values.schema.json assembly from resource fragments
internal/docs/      values.yaml -> Markdown table
internal/chart/     Chart directory location and inspection
internal/scaffold/  Plan construction and application
internal/check/     helm render + house rules
```

<br/>

## Invariants

These are load-bearing. Breaking one is a defect, not a style choice.

**`values.yaml` is never rewritten.** `internal/values` appends text and nothing else. Do not replace it with a `yaml.Node` round-trip: that preserves keys and comments but eats blank lines and section banners, so every `hck add` would silently reformat the user's file.

**Removal deletes templates and nothing else.** `scaffold.PlanRemove` emits `Delete` entries and a `ValuesOrphaned` list; it never touches `values.yaml` or `values.schema.json`. Two removals are refused without `--force`, and both guard something invisible: one another present resource `Requires` (the chart renders and does not work, and nothing says so until helm runs), and one whose file is `scaffold.Edited` (a template that differs from what hck generates is somebody's work, and a mistyped name should not delete it). A key another present resource also declares is not orphaned — `persistence` belongs to the StatefulSet as much as to the PVC.

**`hck sync` cannot tell a local edit from an hck template that moved on.** Both are simply not the bytes `render.ResourceTemplate` produces now, and `scaffold.Drift` reports exactly that much. This is why the default is a report, why `--write` takes resource names, and why `--write` with neither names nor `--all` is an error rather than a guess. `Unreadable` is a third state on purpose: reporting an unreadable file as edited would invite `--write` to overwrite it.

**`hck sync` compares the chart skeleton too, except the two files the author owns.** `scaffold.skeletonDrift` walks `render.ChartFiles()` and compares everything not listed in `skeletonNotOwned` — so a file added to `templates/chart/` is picked up by default, which is the right default and the dangerous one. `Chart.yaml` grows dependencies and maintainers; `values.yaml` is append-only. Comparing either would report drift on every chart that ever grew, and `--write` would delete what hck never wrote. `TestTheSkeletonSetIsADecision` pins the set so adding a skeleton file forces that call, and `TestTheAuthorsFilesAreNotCompared` holds the two exclusions. This gap was real: `.helmignore` gained a line and `_helpers.tpl` is what every template calls into, and neither was ever compared.

**Templates use `[[ ]]`, Helm uses `{{ }}`.** The generation layer's delimiters are set in `internal/render`. A template rendering with a `[[` still in it is caught by `TestEveryCatalogResourceRenders`.

**`//go:embed all:templates`, not `//go:embed templates`.** Without `all:`, embed silently drops every path segment starting with `_` or `.` — which is exactly `templates/chart/templates/_helpers.tpl`.

**Cross-resource values access is parenthesised.** `(.Values.autoscaling).enabled`, never `.Values.autoscaling.enabled`, because the HPA may not be in the chart. Sprig's `dig` fails here: `.Values` is a `chartutil.Values`, not a `map[string]interface{}`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [somaz94/helm-chart-kit](https://github.com/somaz94/helm-chart-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
