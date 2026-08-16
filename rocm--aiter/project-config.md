---
trigger: always_on
description: Scope: **GEMM and MOE configs only.** Read this before adding, moving,
---

# Triton kernel configs — rules for automated edits

Scope: **GEMM and MOE configs only.** Read this before adding, moving,
renaming, or tuning a GEMM or MOE JSON file under
`aiter/ops/triton/configs/`, or before touching
`utils/gemm_config_utils.py` or `utils/moe_config_utils.py`.

Out of scope, do not touch without an explicit request: `configs/conv/`,
`configs/hstu_attn/`, and the flat attention / GMM / MHC / MLA files at the top
of `configs/`. They have their own loaders and are unaffected by anything here.

The tree is **mid-migration** from a flat, arch-prefixed layout to a nested
`<arch>/<backend>/<op>/<d_type>/` layout. Both layouts are live, but **the legacy flat
layout is deprecated and will be removed** — treat it as read-only history, not
as a place to add things.

Two non-negotiables:

1. **Tuning values live in JSON, never in Python.** No `setdefault`, no inline
   dict literals, no arch-conditional constants, no hardcoded fallback configs.
   If a value is missing, fix the JSON.
2. **New configs go in the target layout** unless their family is still in the
   legacy directory (see §6).

`GEMM-AFP4WFP4` (gfx950 triton, gfx950/gfx1250 gluon) is the only migrated
family and the worked reference — copy its shape when in doubt.

---

## 1. Layouts

### Target layout (use for all new configs)

```
configs/<arch>/<backend>/<op>/<d_type>/DEFAULT.json
configs/<arch>/<backend>/<op>/<d_type>/<CONFIG_NAME>-<suffix>.json
```

| Segment     | Values                                                    |
| ----------- | --------------------------------------------------------- |
| `<arch>`    | `gfx942`, `gfx950`, `gfx1250`, `gfx1151`, `gfx1200`, `gfx1201` |
| `<backend>` | `triton` or `gluon`                                        |
| `<op>`      | `gemm` or `moe`                                            |
| `<d_type>`  | `config_name.lower().replace("-", "_")` — `GEMM-AFP4WFP4` → `gemm_afp4wfp4`. The transform lives in `gemm_config_utils._dtype_dir()` |
| filename    | **no arch prefix** — the arch is the directory. The default is literally `DEFAULT.json`; specialized files keep the `<CONFIG_NAME>-` stem |

```
configs/gfx950/triton/gemm/gemm_afp4wfp4/DEFAULT.json
configs/gfx950/triton/gemm/gemm_afp4wfp4/GEMM-AFP4WFP4-N=8192-K=8192.json
configs/gfx950/gluon/gemm/gemm_afp4wfp4/DEFAULT.json
configs/gfx1250/gluon/gemm/gemm_afp4wfp4/DEFAULT.json
```

The `<arch>/<backend>/moe/` directories exist but are empty, held open with
`.gitkeep`. Keep them. **No MOE config has been migrated and no MOE resolver
understands the nested layout yet** — see §5.

### Legacy layout — deprecated, pending removal

```
configs/gemm/<arch>-<CONFIG_NAME>[-<suffix>].json
configs/gemm/gluon/<arch>-<CONFIG_NAME>[-<suffix>].json
configs/moe/<arch>-MOE-<dtype_str>.json
configs/moe/<arch>-A8W4.json
configs/moe/<arch>-MOE_ROUTING_SIGMOID_TOPK1.json
```

Regenerate rather than trusting this listing:
`git ls-tree -r --name-only HEAD aiter/ops/triton/configs/`

Still authoritative for every family not yet migrated. For GEMM it is reached
through the fallback chain in §2; for MOE it is the *only* path that works.
The GEMM fallback is temporary — anything left in `configs/gemm/` when the
legacy candidates are dropped from `gemm_config_utils.py` will stop resolving.

---

## 2. GEMM resolution order — `get_gemm_config()`

`utils/gemm_config_utils.py` picks a directory by probing for the *default*
config file (`DEFAULT.json` in the nested layout, `<arch>-<CONFIG_NAME>.json`
in legacy) in order and taking the first hit. Specialized files are then read
from that same directory.

**`backend=None`** (what every caller uses today):

1. `configs/<arch>/triton/gemm/<d_type>/DEFAULT.json`
2. `configs/<arch>/gluon/gemm/<d_type>/DEFAULT.json`
3. `configs/gemm/<arch>-<CONFIG_NAME>.json`  *(legacy)*

**`backend="triton"|"gluon"`**:

1. `configs/<arch>/<backend>/gemm/<d_type>/DEFAULT.json`
2. `configs/gemm/<backend>/<arch>-<CONFIG_NAME>.json`  *(legacy)*
3. `configs/gemm/<arch>-<CONFIG_NAME>.json`  *(legacy)*

If nothing matches, the last legacy candidate is used and the missing-default
assertion fires there — so error messages still point at `configs/gemm/`.

The legacy candidates are marked `# TODO(satya): legacy, remove` and are
scheduled for deletion. Do not write new code that depends on them resolving.

Consequences to keep in mind:

- **A directory is chosen as a unit.** The unit is the family's `<d_type>/`
  directory. Splitting a config family across `<arch>/triton/gemm/<d_type>/`
  and legacy `configs/gemm/` silently drops the specialized files in whichever
  directory loses the probe. Move a family wholesale or not at all. Worse: a
  `<d_type>/` directory with specialized files but **no `DEFAULT.json` is
  invisible** — the probe keys only on `DEFAULT.json` and falls through to
  legacy, ignoring everything in the directory.
- **`backend=None` prefers `triton` over `gluon`.** On an arch with only a
  gluon default (currently gfx1250 `GEMM-AFP4WFP4`), lookup falls through to
  gluon. Adding `configs/gfx1250/triton/gemm/gemm_afp4wfp4/DEFAULT.json` later
  would change which file gfx1250 resolves to — verify that is intended.
- Results are cached twice: `functools.lru_cache` on the full argument

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ROCm/aiter](https://github.com/ROCm/aiter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
