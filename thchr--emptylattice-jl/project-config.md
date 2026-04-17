---
trigger: always_on
description: Julia package for empty-lattice photonic band structure and first-order perturbation theory.
---

# Package: EmptyLattice.jl

Julia package for empty-lattice photonic band structure and first-order perturbation theory.

## Associated theory project

The theory underpinning this package lives at:
`/mnt/c/Users/tchr/OneDrive - Danmarks Tekniske Universitet/Files/Projects/perturbative-topological-analysis/`

- `latex/main.tex` — master theory note "Empty-lattice perturbation theory" (RevTeX, author: Thomas Christensen, DTU); **primary reference** for all `PerturbationTheory` implementation
- `latex/repeated-irreps.tex` — Phase 4 (M>1 multiplicity) theory (included into main.tex)
- `*.jl` — Julia scripts for phase diagrams, perturbation expressions, MPB comparisons (see CLAUDE.md in that project for the full table)
- `figs/` — figures referenced by main.tex

That directory has its own `CLAUDE.md` with full context about the LaTeX conventions, Julia scripts, and implementation phases. When working on theory (latex edits, phase diagrams, MPB comparisons), switch to that project.

## Package structure

```
EmptyLattice/
├── Project.toml                        # deps: Crystalline ≥0.6.23, Bravais, StaticArrays, LinearAlgebra
│                                       # weakdeps: Makie (extension)
├── ext/
│   └── EmptyLatticeMakieExt.jl         # Makie extension: plot_dielectric, plot_dielectric!
├── src/
│   ├── EmptyLattice.jl                 # Main module: spectrum, unique_spectrum, planewave_symeigs
│   ├── fincke-pohst-enumeration.jl     # Fincke–Pohst sphere enumeration (finds full orbit)
│   ├── symvec_distance.jl              # (utility, not part of public API)
│   └── PerturbationTheory/
│       ├── PerturbationTheory.jl       # Submodule declaration + exports
│       ├── polarizations.jl            # (1) TE/TM polarization frame utilities
│       ├── gamma_rep.jl                # (2) Γ representation matrices; b_vector_orbits
│       ├── coefficients.jl             # (3) symmetry_adapted_coefficients
│       ├── perturbation_results.jl     # (4) result types: ShiftTerm, IrrepShiftExpr, evaluate
│       └── frequency_shifts.jl         # (5) geometric_factor, frequency_shifts (high-level);
│                                       #     stubs for plot_dielectric/plot_dielectric!
└── test/
    ├── runtests.jl                     # Test runner (includes all test files below)
    └── perturbation_theory/
        ├── test_p2.jl                  # p2, Y-point, TM+TE
        ├── test_p4_X.jl                # p4, X-point, TM+TE
        ├── test_p4_M.jl                # p4, M-point, TM+TE
        ├── test_p6mm_Gamma.jl          # p6m, Γ-point, degeneracy_idx=1 and 2
        ├── test_nonsymmorphic_phases_2d.jl  # Phase tests: sg=7 (p2mg), sg=12 (p4gm)
        ├── test_p41_orbit_phases.jl    # b_vector_orbits phase convention test
        ├── test_pm3m_X.jl              # 3D: pm3m, X-point
        └── test_multiplicity.jl        # M>1: P̄1 (sg=2, 3D) X-point + p2mg (sg=7, 2D) S-point
```

## Core EmptyLattice API

- `spectrum(kv, Gs; maxN, Nfreq)` — all empty-lattice frequencies at kv
- `unique_spectrum(kv, Gs; maxN, Nfreq, atol)` → `(ωs, kvGsv)` — unique frequencies + orbits
  - `kvGsv[i]::Vector{SVector{D,Float64}}` — orbit vectors for the i-th unique frequency
- `planewave_symeigs(lg, Gs, polarization; Nfreq)` — symmetry eigenvalues at each frequency

## PerturbationTheory submodule API

**Exported symbols:** `gamma_matrix`, `gamma_matrices`, `b_vector_orbits`,
`symmetry_adapted_coefficients`, `multiplicity_adapted_coefficients`,
`geometric_factor`, `frequency_shifts`,
`OrbitRelations`, `ShiftTerm`, `IrrepShiftExpr`,
`AbstractShiftExpr`, `MultipletShiftTerm`, `DoubletShiftExpr`, `MultipletShiftExpr`,
`evaluate`, `plot_dielectric`, `plot_dielectric!`

**Makie extension** (`ext/EmptyLatticeMakieExt.jl`, loaded when a Makie backend is active):
- `plot_dielectric(orbits, Δεs [, Gs_or_Rs]; npoints, levels, colorbar, ...)` → `Figure`
- `plot_dielectric!(ax, orbits, Δεs [, Gs_or_Rs]; npoints, levels, ...)` → plot object
  - `orbits::Vector{OrbitRelations{2}}`, `Δεs::Vector{<:Real}` (one canonical Δε per orbit)
  - `Gs_or_Rs::Union{ReciprocalBasis{2}, DirectBasis{2}, Nothing} = nothing`
    - `nothing` → fractional coordinates on [-½,½]²
    - `DirectBasis{2}` / `ReciprocalBasis{2}` → field always computed on fractional grid;
      xs_c, ys_c coordinate matrices built via `r_cart = Rm * r_frac` and passed to
      `contourf!` as a curvilinear grid (Makie supports matrix x/y arguments)

### High-level workflow
```julia
Gs    = dualbasis(primitivize(directbasis(sgnum, Val(D)), centering(sgnum, D)))
lgirs = primitivize(lgirreps(sgnum, Val(D))["M"])   # irreps at k-point of interest

es = frequency_shifts(lgirs, Gs, degeneracy_idx; polarization=:TM)
# → Collection{IrrepShiftExpr{D}}

evaluate(es, Dict(SVector(1.0,0.0) => 0.3, SVector(1.0,1.0) => 0.2))
# → Dict{String, Float64}  (irrep label → Δω)
```

### Key types
- `OrbitRelations{D}` — orbit of a b-vector + phase coefficients
  - Convention: `coefs[i] * Δε[orbit[i]] = Δε̃` (real common RHS)
  - `coefs[1] = exp(iθ)` where `θ = −arg(α)/2`, `α` = constraint phase; = 1 for cosine orbits
  - Phase: `coefs[i] = exp(+2πi b_i · w)` for operation g=(W,w) mapping canonical→b_i

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thchr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
