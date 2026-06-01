---
trigger: always_on
description: nf-test assertion patterns for scdownstream — how to write platform-independent, targeted tests
---


# nf-test Assertion Patterns

## Core principle

Omit **platform-sensitive** content from snapshots when outputs are **non-deterministic** (randomised ML, unstable floats in matrices, live APIs): use **`structural`** snapshots (`versions` YAML + `adata.yaml`, no `process.out` MD5s).

When a module is **deterministic** (see [`docs/reproducibility.md`](../docs/reproducibility.md)) and **emits H5AD**, **`hash`** snapshots **include** `process.out` file MD5s plus `adata.yaml` — that is intentional regression coverage, not “raw float arrays” in the snap.

## Assertion order — sequential, no `assertAll`

All `then {}` blocks use **plain sequential `assert` statements** in this fixed order:

1. `assert process.success` / `assert workflow.success` — bare, so the real failure mode is visible before deeper checks
2. `def adata = anndata(...)` — when an output H5AD (or similar) is used in field checks or in `adata.yaml` inside `snapshot()`; extract once if referenced **2+** times
3. **Field checks** — specific assertions before the snapshot (e.g. `assert "X_pca" in adata.obsm`)
4. `assert snapshot(...).match()` — last

**Why this order**: missing fields fail in step 3 with a short message; the snapshot is the final regression net.

If a test **must** open AnnData only after success and you keep `def adata` before `assert success` for historical reasons, treat **success before field checks and snapshot** as the hard rule when refactoring.

**Never use `assertAll`** — it groups all failures into one report which makes CI output harder to read when a process fails.

## Stub tests

Stub tests always use the simple pattern:

```groovy
test("Should run without failures - stub") {
    options '-stub'
    then {
        assert process.success
        assert snapshot(process.out).match()
    }
}
```

## Non-stub tests — full example

```groovy
then {
    assert process.success
    def adata = anndata(process.out.h5ad[0][1])
    assert "X_pca" in adata.obsm          // field checks BEFORE snapshot
    assert "X_pca" in adata.uns
    assert snapshot(
        process.out,
        path(process.out.versions[0]).yaml,
        adata.yaml
    ).match()                              // snapshot LAST — comprehensive regression
}
```

## `def adata` variable

When `anndata(path)` would appear more than once in the same `then {}` block, always extract it:

```groovy
def adata = anndata(process.out.h5ad[0][1])
// then use adata.obsm, adata.yaml, with(adata) { ... }
```

For subworkflow tests, the path comes from `workflow.out`:

```groovy
def adata = anndata(workflow.out.h5ad[0][1])
// or workflow.out.integrations[0][1] etc.
```

## `with(adata)` for grouped checks

When asserting multiple properties of the same AnnData object, use `with(adata)`:

```groovy
assert process.success
def adata = anndata(process.out.h5ad[0][1])
with(adata) {
    assert "batch" in obs.colnames
    assert "label" in obs.colnames
    assert !("counts" in layers)
}
assert snapshot(process.out, path(process.out.versions[0]).yaml, adata.yaml).match()
```

## What to include in `snapshot()`

Strategy names are **identical** to [`docs/reproducibility.md`](../docs/reproducibility.md) → **Test strategy (this branch)**. The doc assigns each module/subworkflow to a strategy; this table is the **default `snapshot(...)` shape per strategy** (modules use `process.*`; subworkflows use `workflow.out.*` as in the examples below).

### Module strategies — default snapshot contents

| Strategy | Typical `snapshot(...)` arguments |
|----------|----------------------------------|
| `hash` | `process.out`, `path(process.out.versions[0]).yaml`, `adata.yaml` — module **emits H5AD** |
| `hash (no H5AD output)` | `process.out`, `path(process.out.versions[0]).yaml` — **no** `adata.yaml` |
| `hash + structural` | Same as **`hash`** (label in doc marks modules like `scanpy/filter` with multiple scenarios) |
| `structural` | `path(process.out.versions[0]).yaml`, `adata.yaml` — **omit** `process.out` |
| `column names` | e.g. `path(process.out.versions[0]).yaml`, `adata.obs.colnames` — **omit** `process.out` MD5s |
| `column names only` | Same as **`column names`** (shorthand in `docs/reproducibility.md` module tables only) |
| `obs CSV + file names` | Per module row in doc (obs artefacts + versions; not full H5AD hash) |
| `range assertion` | Imperative range checks **plus** the snapshot mix given in that doc row |
| `stub only` | No non-stub test; stub only: `snapshot(process.out).match()` |

### Subworkflow tests

Patterns vary by workflow. Use the **Subworkflows** table in `docs/reproducibility.md` as the source of truth. Common building blocks:

```groovy
// Versions as YAML (often + adata.yaml); omit full workflow.out when unstable
assert snapshot(
    workflow.out.versions.collect { path(it).yaml },
    adata.yaml
).match()
```

```groovy
// Deterministic outputs + versions + schema
assert snapshot(
    workflow.out,
    workflow.out.versions.collect { path(it).yaml },
    adata.yaml
).match()
```

**Passthrough / unstaged H5AD paths:** `snapshot(workflow.out)` only — see later section.

## Common mistake — dropping `process.out` from **`hash`** modules that emit H5AD


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nf-core/scdownstream](https://github.com/nf-core/scdownstream) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
