---
trigger: always_on
description: We use the following numbering schemes on Cartesian or curved structured meshes.
---

# [Conventions](@id conventions)

## Spatial dimensions and directions

We use the following numbering schemes on Cartesian or curved structured meshes.
- The `orientation`s are numbered as
  `1 => x, 2 => y, 3 => z`.
  For example, numerical fluxes such as
  `flux_central(u_ll, u_rr, orientation, equations::AbstractEquations)`
  use the `orientation` in this way.
- The `direction`s are numbered as
  `1 => -x, 2 => +x, 3 => -y, 4 => +y, 5 => -z, 6 => +z`.
  For example, the `boundary_conditions` are ordered in this way
  when a `Tuple` of boundary conditions per direction is passed
  to the constructor of a `SemidiscretizationHyperbolic`.
- For structured and unstructured curved meshes the concept of direction is
  generalized via the variable `normal_direction`. This variable points in the
  normal direction at a given, curved surface. For the computation of boundary fluxes
  the `normal_direction` is normalized to be a `normal_vector` used, for example, in
  [`FluxRotated`](@ref).


## Cells vs. elements vs. nodes

To uniquely distinguish between different components of the discretization, we use the
following naming conventions:
* The computational domain is discretized by a `mesh`, which is made up of
  individual `cell`s. In general, neither the `mesh` nor the `cell`s should be
  aware of any solver-specific knowledge, i.e., they should not store anything
  that goes beyond the geometrical information and the connectivity.
* The numerical `solver`s do not directly store their information inside the `mesh`,
  but use own data structures. Specifically, for each `cell` on which
  a solver wants to operate, the solver creates an `element` to store
  solver-specific data.
* For discretization schemes such as the discontinuous Galerkin or the finite
  element method, inside each `element` multiple `nodes` may be defined, which
  hold nodal information. The nodes are again a solver-specific component, just
  like the elements.
* We often identify `element`s, `node`s, etc. with their (local or global)
  integer index. Convenience iterators such as `eachelement`, `eachnode`
  use these indices.


## Keywords in elixirs

Trixi.jl is distributed with several examples in the form of elixirs, small
Julia scripts containing everything to set up and run a simulation. Working
interactively from the Julia REPL with these scripts can be quite convenient
while for exploratory research and development of Trixi.jl. For example, you
can use the convenience function
[`trixi_include`](@ref)
to `include` an elixir with some modified arguments. To enable this, it is
helpful to use a consistent naming scheme in elixirs, since
[`trixi_include`](@ref)
can only perform simple replacements. Some standard variables names are

- `polydeg` for the polynomial degree of a solver
- `surface_flux` for the numerical flux at surfaces
- `volume_flux` for the numerical flux used in flux differencing volume terms

Moreover, [`convergence_test`](@ref) requires that the spatial resolution is
set via the keywords
- `initial_refinement_level`
  (an integer, e.g., for the [`TreeMesh`](@ref) and the [`P4estMesh`](@ref)) or
- `cells_per_dimension`
  (a tuple of integers, one per spatial dimension, e.g., for the [`StructuredMesh`](@ref)
  and the [`DGMultiMesh`](@ref)).


## Variable names

- Use descriptive names (using `snake_case` for variables/functions and `CamelCase` for types)
- Use a suffix `_` as in `name_` for local variables that would otherwise hide existing symbols.
- Use a prefix `_` as in `_name` to indicate internal methods/data that are "fragile" in the
  sense that there's no guarantee that they might get changed without notice. These are also not
  documented with a docstring (but maybe with comments using `#`).


## Structure of the `solver` directory

If some functionality is shared by multiple combinations of meshes/solvers,
it is defined in the directory of the most basic mesh and solver type.
An example for this is the `rhs!` function, which lays out the sequence of functions
that compose the overall right-hand-side function provided to the ODE integrator.
Since this general "recipe" can be unified for different meshes of a certain dimension,
a shared implementation is used to minimize code duplication.

The most basic (in the sense that it is most tested and developed) solver type in Trixi.jl is
[`DGSEM`](@ref) due to historic reasons and background of the main contributors.
We consider the [`TreeMesh`](@ref) to be the most basic mesh type since it is Cartesian
and was the first mesh in Trixi.jl.
Thus, shared implementations for more advanced meshes such as the [`P4estMesh`](@ref) can be found in
the `src/solvers/dgsem_tree/` directory, while only necessary specifics are actually placed in
`src/solvers/dgsem_p4est/`.


## Array types and wrapping

To allow adaptive mesh refinement efficiently when using time integrators from
[OrdinaryDiffEq](https://github.com/SciML/OrdinaryDiffEq.jl) and its sub-packages,
Trixi.jl allows to represent numerical solutions in two different ways. Some discussion
can be found [online](https://github.com/SciML/OrdinaryDiffEq.jl/pull/1275) and
in form of comments describing `Trixi.wrap_array` and `Trixi.wrap_array_native`
in the source code of Trixi.jl.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trixi-framework/Trixi.jl](https://github.com/trixi-framework/Trixi.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
