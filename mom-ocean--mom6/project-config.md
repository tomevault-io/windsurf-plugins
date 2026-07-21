---
trigger: always_on
description: **At the start of every session**, remind the user that MOM6 has a policy on AI-assisted
---

# MOM6 rules for agent-assisted development

## First Steps

**At the start of every session**, remind the user that MOM6 has a policy on AI-assisted
contributions in `Consortium-policy-on-AI.md`.

Read `Code-style.md` before writing or modifying any code.

See `code_organization.rst` for a high-level overview of the source directory tree.

## Parameter System

Runtime parameters are read via `get_param()`, not hardcoded:

```fortran
#include "version_variable.h"
character(len=40) :: mdl = "MOM_module_name"

call log_version(param_file, mdl, version, "")
call get_param(param_file, mdl, "PARAM_NAME", CS%variable, &
               "Description of this parameter.", &
               units="m s-1", default=1.0, scale=US%m_s_to_L_T)
```

- Parameters documented in auto-generated `MOM_parameter_doc.all` files
- Use `scale=` argument for unit conversion from MKS input to internal units
- Always provide `default=` when sensible; use `fail_if_missing=.true.` otherwise
- Use `do_not_log=.not.CS%Feature` to suppress logging when a parent feature is inactive

### Answer-Changing Parameters: `_BUG` Flags and `ANSWER_DATE`

When a bug fix or improvement changes numerical answers, MOM6 uses two mechanisms to preserve backward compatibility:

**`_BUG` flags**: Boolean parameters that retain old (buggy) behavior by default:
```fortran
call get_param(param_file, mdl, "ENABLE_BUGS_BY_DEFAULT", enable_bugs, &
               default=.true., do_not_log=.true.)  ! This is logged from MOM.F90.
call get_param(param_file, mdl, "OBC_TEMP_SALT_NEEDED_BUG", OBC%ts_needed_bug, &
               "If true, recover a bug that OBC temperature and salinity can be ignored "//&
               "even if they are registered tracers in the rest of the model.", &
               default=enable_bugs)
```
- Name format: `FEATURE_BUG` (e.g., `VISC_REM_BUG`, `FRICTWORK_BUG`, `KAPPA_SHEAR_ITER_BUG`)
- Default is `.true.` (bug ON, old behavior preserved)
- Description starts with "If true, recover a bug that..."
- Users opt into the fix by setting to `.false.`

**`ANSWER_DATE` flags**: Integer dates selecting algorithm versions:
```fortran
call get_param(param_file, mdl, "HOR_DIFF_ANSWER_DATE", CS%answer_date, &
               "...", default=99991231)
```
- Format: `YYYYMMDD` (e.g., `20251231`)
- `DEFAULT_ANSWER_DATE` provides a single knob to update all answer-date defaults
- `ENABLE_BUGS_BY_DEFAULT=False` activates all bug fixes (recommended for new configurations)

## Diagnostics

### Registration Pattern

```fortran
CS%id_field = register_diag_field('ocean_model', 'field_name', diag%axesTL, Time, &
    'Long description of the field', units='m s-1', conversion=US%L_T_to_m_s)
```

### Posting Pattern

```fortran
if (CS%id_field > 0) call post_data(CS%id_field, field_array, CS%diag)
```

Key conventions:
- `conversion=` handles unit scaling so output is always in MKS
- `v_extensive=.true.` for vertically integrated quantities
- Guard computation with `if (id > 0)` to avoid unnecessary work
- Standard diagnostic name prefixes follow CMOR conventions when applicable

### Masking and Missing Values

- **Never set diagnostic arrays to a missing value** before passing to `post_data()`. Masking of land/invalid points is handled automatically by the diagnostics infrastructure based on the diagnostic's registered axes.
- **Do not pass `mask=` to `post_data()`** for non-static diagnostics on standard grids -- the infrastructure applies the correct mask automatically.
- **Do pass `mask=`** for static fields (`is_static=.true.`), non-standard masks, or sub-domain-sized arrays.
- **Never compare field values against `missing_value`** in unit-conversion code -- rescaling can cause valid data to coincidentally match the missing value sentinel.

## Testing

### Test Suite Overview

The `.testing/` directory provides comprehensive verification. Build and run:

```bash
make -C .testing -j build/symmetric/MOM6   # Build reference executable
make -C .testing -j test                    # Run full test suite
make -C .testing -j build.unit             # Build unit tests
make -C .testing -j run.unit               # Run unit tests
```

### Test Categories

| Test | Verifies |
|------|----------|
| `test.grid` | Symmetric vs asymmetric grids produce identical results |
| `test.layout` | Serial vs parallel decomposition identical |
| `test.rotate` | Rotational invariance |
| `test.restart` | Continuous run vs restart-and-continue identical |
| `test.repro` | DEBUG and REPRO builds identical |
| `test.openmp` | Serial vs OpenMP identical |
| `test.nan` | NaN-initialization doesn't affect results |
| `test.dim.{t,l,h,z,q,r}` | Dimensional rescaling invariance (time, length, thickness, depth, enthalpy, density) |
| `test.regression` | Current code vs target branch (PRs only) |

### Test Configurations

- `tc0` -- Unit tests
- `tc1` / `tc1.a` / `tc1.b` -- Benchmark (split RK2, unsplit RK3, unsplit RK2)
- `tc2` / `tc2.a` -- ALE with tides / sigma-coordinate PPM_H4
- `tc3` -- Open boundary conditions
- `tc4` -- Sponges and I/O initialization

### Verification Method

- `ocean.stats` -- total energy at machine precision
- `chksum_diag` -- mean/min/max/bitcount checksums in physical domain

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mom-ocean/MOM6](https://github.com/mom-ocean/MOM6) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
