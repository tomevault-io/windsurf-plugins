---
trigger: always_on
description: A Lean 4 project for formally verified CSG / geometric algorithms.
---

# CSG — verified geometric algorithms

A Lean 4 project for formally verified CSG / geometric algorithms.

## Trust model

Correctness rests on **two things only**: the Lean type checker, and human review
of a *small* specification. It does **not** rest on trusting any AI agent.

A human reviewer of the kernel reads only the **specification files**.

The reviewed surface splits into **three independent parts**. Only the first is
the web application and the kernel its calls; the other two are standalone/legacy
surfaces relegated to `CSG/Legacy/`:

- the **web-application spec** — the files at the **top level of `CSG/`**:
  `DataStructures.lean`, `Def.lean` (ray-based mesh semantics), the mesh-facing
  statement files (`MeshIntersect.lean`, `WellFormedCheck.lean`,
  `MeshIntersectWithPreconditionCheck.lean`, `WellFormedCheckMsg.lean`) and
  `Web.lean` — plus `Example.lean`, optional sanity theorems that give a
  reviewer extra assurance about the semantics but are not needed to specify
  the algorithms;
- the standalone **general-position utility** — `CSG/Legacy/GenPosDef.lean` and
  `CSG/Legacy/GenPosCheck.lean` (the `meshIntersect` theorems carry no
  general-position hypothesis);
- the separate, **legacy chain-level spec** — `CSG/Legacy/ChainDataStructures.lean`
  and `CSG/Legacy/ChainDef.lean` (Feito–Rivero origin-cone winding) with
  `CSG/Legacy/ChainNormalCheck.lean`, `CSG/Legacy/ChainIntersectionAlgorithm.lean`
  and `CSG/Legacy/ChainIntersectionExistence.lean`.

The web-application spec depends on **nothing** under `CSG/Legacy/` — no top-level
`CSG/` spec file nor any runtime implementation imports a Legacy module, and the
mesh runtime never calls the chain or general-position algorithms — so a reviewer
of the application never has to read a Legacy file, and each surface is reviewed
on its own. (Untrusted *proofs* in `CSG/Proof/` may still reuse Legacy machinery;
that never enlarges the reviewed surface or the runtime.)

## File roles

| File | Reviewed by humans? | Mathlib? | Role |
| --- | --- | --- | --- |
| `DataStructures.lean` | yes | no | The mesh data types in the algorithm interface (shared by all reviewed surfaces). |
| `Def.lean` | yes | yes | The ray-based mesh semantics used to state the web-application theorems. |
| web-application statement files | yes | yes | `MeshIntersect.lean`, `WellFormedCheck.lean`, `MeshIntersectWithPreconditionCheck.lean`, `WellFormedCheckMsg.lean` — theorem statements and algorithm signatures at the top level of `CSG/`. The *meaning* (type) of each depends **only** on `DataStructures.lean` and `Def.lean`. |
| `Example.lean` | yes (optional) | yes | Sanity theorems exercising the `Def.lean` semantics (∀/∃ agreement for `solid`, admissible-ray existence, a concrete well-formed cube whose solid is exactly the open unit cube). Pure assurance: nothing in the spec or runtime depends on it. |
| `CSG/Legacy/*.lean` | yes | yes | The **general-position utility** (`GenPosDef.lean`, `GenPosCheck.lean`) and the **legacy chain-level spec** (`ChainDataStructures.lean`, `ChainDef.lean`, `ChainNormalCheck.lean`, `ChainIntersectionAlgorithm.lean`, `ChainIntersectionExistence.lean`) — two *separate* reviewed surfaces the web application does **not** depend on. Namespaces `CSG.Legacy.*`. |
| `CSG/Impl/*.lean` | no | no | Executable definitions of the application (the algorithms). |
| `CSG/Impl/Legacy/*.lean` | no | no | Executable definitions used only by the Legacy surfaces / by proofs (never by the app runtime). Namespaces `CSG.Impl.Legacy.*`. |
| `CSG/Proof/*.lean` | no | yes | Formal proofs of the specifications (app and legacy alike). |
| `Web.lean` | yes | no | WASM entry point: a `String → String` wrapper over the exported algorithms, `@[export]`ed. |

**Repository layout.** The reviewed *application* surface is easy to find: its
specification files (data/semantics + the statement files above) sit directly at
the **top level of `CSG/`**. Everything else lives in subfolders — executable app
code in **`CSG/Impl/`** (namespaces `CSG.Impl.*`), proofs in **`CSG/Proof/`**
(namespaces `CSG.Proof.*`), informal proof-strategy notes in
**`CSG/ProofPlanning/`**, and the standalone/legacy surfaces (general-position +
chain-level, spec and impl) under **`CSG/Legacy/`** and **`CSG/Impl/Legacy/`**
(namespaces `CSG.Legacy.*` / `CSG.Impl.Legacy.*`). So the trust boundary is
visible both in the folder tree and in every fully-qualified name: the trusted
*application* spec is exactly the top level of `CSG/`; `CSG.Legacy.*` is a
separate reviewed surface the app does not depend on; `CSG.Impl.*`/`CSG.Proof.*`
are untrusted.

The specification files delegate their definitions to the `CSG/Impl/` files and
their proofs to the `CSG/Proof/` files. So a reviewer never needs to read a file
under `CSG/Impl/` or `CSG/Proof/`: they are machine-checked, not trusted. The one
exception is `Web.lean`: its `String → String` (de)serialization is trusted glue
with no correctness proof, so it is kept minimal and is reviewed.

## Adding to the specification (for specification agents)

When introducing a new user-facing definition or theorem, keep the trusted surface
minimal and self-contained:

1. **Put each piece in the right file** (see *File roles*):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [schildep/verified-3d-mesh-intersection](https://github.com/schildep/verified-3d-mesh-intersection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
