---
trigger: always_on
description: A comprehensive mathematical reasoning skill for AI assistants — handles arithmetic to research-level problems with rigorous step-by-step reasoning, systematic verification, and transparent uncertainty handling
---


# Math.skill

## Skill Name

**Math.skill** — A comprehensive mathematical reasoning skill for AI assistants.

## Skill Purpose

Enable AI assistants to handle mathematical tasks across all difficulty levels — from basic arithmetic to research-level problems — with rigorous, step-by-step reasoning, systematic verification, and transparent uncertainty handling.

This skill enforces a disciplined mathematical workflow: every problem is parsed, modeled, solved with justifications, verified through multiple independent checks, and only then delivered as a final answer. The verification engine (see **Verification Engine** section) is the core differentiator — no answer is output without passing at least two verification methods.

## Scope of Application

This skill covers the following mathematical domains:

- **Foundations**: Arithmetic, number sense, order of operations, unit conversions
- **Algebra**: Algebraic expressions, polynomial operations, factoring, completing the square, rational expressions, radical expressions, exponents and logarithms
- **Equations**: Linear, quadratic, polynomial, rational, radical, exponential, logarithmic, absolute value equations
- **Inequalities**: Linear, quadratic, rational, absolute value, exponential, logarithmic inequalities; systems of inequalities
- **Functions**: Domain/range, composition, inverse, monotonicity, parity, periodicity, graphing, transformations, piecewise functions
- **Geometry**: Plane geometry, solid geometry, coordinate geometry, vector geometry, geometric transformations
- **Trigonometry**: Trigonometric functions, identities, equations, triangle solving, inverse trigonometric functions
- **Sequences and Series**: Arithmetic, geometric, recursive sequences; series convergence, summation formulas
- **Combinatorics**: Permutations, combinations, inclusion-exclusion, pigeonhole principle, generating functions
- **Probability and Statistics**: Classical probability, conditional probability, Bayes' theorem, distributions, expectation, variance, hypothesis testing, confidence intervals
- **Limits**: Limits of sequences and functions, one-sided limits, limits at infinity, epsilon-delta definitions, L'Hôpital's rule
- **Differentiation**: Derivative rules, implicit differentiation, logarithmic differentiation, higher-order derivatives, applications (tangents, rates, optimization, curve sketching)
- **Integration**: Indefinite and definite integrals, substitution, integration by parts, partial fractions, trigonometric integrals, improper integrals, applications (area, volume, arc length, work)
- **Multivariable Calculus**: Partial derivatives, gradients, directional derivatives, double/triple integrals, line integrals, surface integrals, divergence, curl, Green's/Stokes'/divergence theorems
- **Linear Algebra**: Matrices, determinants, vector spaces, linear transformations, eigenvalues/eigenvectors, diagonalization, inner product spaces, quadratic forms
- **Ordinary Differential Equations**: First-order, second-order linear, systems, Laplace transforms, series solutions, qualitative analysis
- **Complex Analysis**: Complex numbers, analytic functions, contour integration, residue theorem, conformal mapping
- **Real Analysis**: Completeness, sequences and series of functions, continuity, differentiation, Riemann integration, measure theory basics
- **Abstract Algebra**: Groups, rings, fields, homomorphisms, isomorphisms, quotient structures, Galois theory basics
- **Topology**: Metric spaces, topological spaces, continuity, compactness, connectedness, fundamental group basics
- **Number Theory**: Divisibility, congruences, prime numbers, Diophantine equations, modular arithmetic, cryptography basics
- **Discrete Mathematics**: Graph theory, recurrence relations, Boolean algebra, automata theory basics
- **Optimization**: Linear programming, nonlinear optimization, constrained optimization, Lagrange multipliers, convex optimization
- **Mathematical Modeling**: Model formulation, parameter estimation, model validation, sensitivity analysis
- **Proofs**: Direct proof, induction, contradiction, contrapositive, construction, exhaustion, epsilon-delta, combinatorial proofs
- **Counterexamples**: Systematic search for counterexamples to disprove conjectures or verify solution uniqueness
- **Solution Checking**: Verifying existing solutions, identifying errors, providing corrections
- **Problem Generation**: Creating well-posed problems with verified solutions at specified difficulty levels
- **Research-Level Problems**: Engaging with open or partially solved problems, clearly distinguishing known results from conjectures

## Out of Scope

This skill should NOT be invoked for:

- **Pure opinion questions**: "Is math beautiful?" — not a mathematical task
- **Non-mathematical creative writing**: Generating poems, stories, or essays not centered on mathematical reasoning
- **Factual lookup without reasoning**: "What is the capital of France?" — no mathematical reasoning required (use general knowledge or web search directly)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Wholiver/Math.Skill](https://github.com/Wholiver/Math.Skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
