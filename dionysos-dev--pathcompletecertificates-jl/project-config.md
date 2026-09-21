---
trigger: always_on
description: Operating guide for AI coding agents (Claude Code) working in
---

# CLAUDE.md

Operating guide for AI coding agents (Claude Code) working in
**PathCompleteCertificates.jl**. Read this before making changes.

---

## 1. What this package is

Certificates for switched systems built on **path-complete graphs**.

A path-complete certificate is always the same three things:

1. a **labelled graph**, whose labels are the modes of the switched system;
2. a function `V_α` drawn from a **template** at each node;
3. one **inequality along each edge**.

The graph is **path-complete** when every switching sequence of the system is readable as a
path in it. That is the soundness condition — without it the inequalities certify nothing.

**What makes this package different from the tools that exist**
([SwitchOnSafety.jl](https://github.com/blegat/SwitchOnSafety.jl), the MATLAB
[JSR Toolbox](https://www.mathworks.com/matlabcentral/fileexchange/33202-the-jsr-toolbox)):
they treat the path-complete graph as an internal device for obtaining a
joint-spectral-radius bound. Here the graph is *the object of study* — something you build,
compare against another, order, and refine iteratively. Keep that framing when adding
features: a change that makes the graph less manipulable is working against the package.

---

## 2. The one architectural contract — read this before touching `src/`

Only the **edge inequality** changes between problems:

| Problem | Edge inequality on `(α, β, i)` |
| :-- | :-- |
| Stability | `V_α(x) ≥ γ⁻¹ V_β(A_i x)` |
| Optimal control | `V_α(x) ≥ c(x) + V_β(f_i(x))` |
| Safety | the invariance condition |

The graph, the templates and the aggregation are identical. So the package has **two
independent axes**, not one type hierarchy:

- **template** (`src/templates/`) — what the node functions are;
- **problem** (`src/problems/`) — what the edge inequality says.

### The interfaces

A template is passed as an **instance**, never as a type. `QuadraticTemplate()` carries
nothing, but a template is not in general determined by its type: `PolyhedralTemplate` carries
one fixed matrix per node, and `::Type{T}` has nowhere to put it.

**A template supplies primitives; a problem chooses which to apply, and with what arguments.**
Neither axis names the other.

Every one of these is **public**. They are what a user implements, so none of them is
`_`-prefixed — a hidden extension point is a contradiction.

```julia
# --- Template axis (src/templates/). One file per template, answering all of it.
add_function_variables!(model, template, dim, node)  # -> the node function V_α
add_domination!(model, template, V_src, V_dst, map; scale = 1, margin = 0)
                                                     # scale·V_src(x) − V_dst(map·x) ≥ margin‖x‖ᵈ
add_nonnegativity!(model, template, V)               # V(x) ≥ 0
add_normalization!(model, template, V)               # excludes V ≡ 0
rate_exponent(template)                              # degree d: V(cx) = cᵈ V(x)
solution_value(template, V)                          # variables -> a callable node function
node_value(template, problem, V, x)                  # evaluate V_α; generic in the problem
check_dynamics(template, A)                          # is this template applicable at all?

# --- Problem axis (src/problems/). One file per problem, composing the above.
add_edge_constraint!(model, problem, template, V_src, V_dst, dynamics; rate = 1)

# --- Aggregation (src/aggregation.jl). Dispatches on the graph, so neither axis owns it.
#   complete → min over nodes;  co-complete → max;  otherwise min-of-max over the observer.
#   `common` is the literature's word (Philippe et al.) -- do not rename it to `aggregate`.
```

**Each problem defines its own certificate, in its own file.** `AbstractCertificate` fixes the
shared interface — `functions`, `status`, `is_feasible`, `problem`, `template`, `graph`, and
callability, where `certificate(x)` is the common function. What a problem actually certifies
is a **typed field** on its own type: `StabilityCertificate.rate`, `SafetyCertificate.margin`,
`OptimalControlCertificate.gains`. Not entries in a shared bag — a field is documented,
inferable and discoverable, and a new problem adds a type rather than inventing keys.

The six shared fields live once in `CertificateData`, which each certificate holds as `data`;
the accessors read it, so a certificate implements nothing to get them.

`add_domination!` is the load-bearing one. Quantifying an edge inequality over all `x` needs a
lifting into a cone, and that lifting is template-specific — which is why it cannot be written
once against a callable `V(x)`. But it is **problem-agnostic**: stability is this with
`scale = γᵈ`, safety is this on the homogeneous lift with `margin = ε`. So:

```julia
# The whole of stability's edge condition, for every template that exists or will:
add_edge_constraint!(model, ::StabilityProblem, template, V_src, V_dst, A, rate) =
    add_domination!(model, template, V_src, V_dst, A; scale = rate)
```

**A new template is one file in `src/templates/`. A new problem is one file in
`src/problems/`. Neither requires editing the other directory.** That is the contract, and
`grep -c '^function add_edge_constraint!' src/problems/*.jl` returning 1 everywhere is how you
check it still holds.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dionysos-dev/PathCompleteCertificates.jl](https://github.com/dionysos-dev/PathCompleteCertificates.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
