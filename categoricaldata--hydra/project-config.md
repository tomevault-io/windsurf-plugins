---
trigger: always_on
description: This document orients an LLM assistant (or human reader) to the Hydra project.
---

# LLM quickstart guide for Hydra

This document orients an LLM assistant (or human reader) to the Hydra project.
It provides just enough context to begin working, then links to detailed documentation.
Prefer consulting linked docs over relying on summaries here.

## What is Hydra?

Hydra is a functional programming language based on the LambdaGraph data model.
It explores an isomorphism between typed lambda calculus and labeled hypergraphs:
**programs are graphs, and graphs are programs.**

Hydra is self-hosting: the kernel is defined in Haskell-based DSLs and code-generated
into nine host languages spanning six implementation families:
Haskell, Java, Python, Scala, TypeScript, and Lisp (Clojure, Common Lisp, Emacs Lisp, Scheme —
sharing one coder).
All nine pass the hydra-kernel test suite as targets; Haskell, Java, Python, Scala, TypeScript, and the
JVM/native Lisp dialects also self-host (Emacs Lisp is still maturing as a host — see README
implementation status).

The Java, Python, and Scala coder DSL sources (`packages/hydra-{java,python,scala}/`) are
authored in Java, Python, and Scala respectively (host-native), and are now the **sole** source of truth:
the Haskell DSL copies under `packages/hydra-{java,python,scala}/src/main/haskell/` have been
**deleted** (#346 for Java/Python; the Scala conversion landed in commit b01f1d34c6).
The `hydra-jvm` package (`packages/hydra-jvm/`) is also authored in Java (host-native) and shares
the same generation pipeline.
The native sources are authoritative for the generated output: the main sync regenerates
`dist/json/hydra-{jvm,java,python,scala}` from them in Phase 5 (`bin/generate-hydra-java-from-java.sh`,
`bin/generate-hydra-python-from-python.sh`, `bin/generate-hydra-scala-from-scala.sh`), and the native
drivers also synthesize the `hydra.dsl.{java,python,scala}.*` wrapper modules (previously written by the
Haskell DSL pass). As of 0.16 the Java and Python drivers run against the **published host by default**
(`net.fortytwo.hydra:hydra-java` from Maven / `hydra-python` from PyPI, version from `hydra.json`
`hostVersion`), with a `--local-host` bootstrap shim for backward-incompatible kernel changes — the #370
"consume" path. The Scala driver currently runs only against the local host (no published-host probe
yet). See [Consuming published hosts](docs/build-system.md#consuming-published-hosts) and
[Migration shims](docs/recipes/migration-shims.md). Note: the Haskell generator still loads the
`hydra.{jvm,java,python,scala}.*` JSON into its inference universe (so cross-package references like
`hydra-rdf` → `hydra.jvm.serde` resolve), but no longer *generates* those packages.

The **Haskell host** compiles the kernel from the co-generated `dist/haskell/hydra-kernel` (always
generated as a target); secondary coders may be consumed from Hackage once published. `dist/haskell/hydra-kernel`
is *both* a generated target and the kernel the host compiles from (host-vs-target split for the exec
drivers, shared kernel source for generated coder output; see #500). `sync-haskell.sh` / `bin/sync.sh`
default to `--published-host`; `--local-host` builds the whole host from source, and
`hostOverrides[pkg]="local"` forces one package local while the rest come from the registry. See
[build-system.md § Consuming published hosts](docs/build-system.md#consuming-published-hosts) for the
full model (probe gate, the per-host mechanics).

A ninth target, Go (`hydra-go`, `heads/go/`), is a "head bud" — the kernel can be generated
to Go via `bin/sync-go.sh`, but the Go coder still has emission bugs and the head's
hand-written runtime is mostly placeholder. Go is not yet a complete implementation
and does not (yet) host the test suite.

Key use cases: graph construction (TinkerPop, RDF, SHACL, GQL), data integration
(coders for Protobuf, Avro, JSON, YAML, GraphQL, PDL, CSV/TSV, RDF), and computational
graphs with deep support for polymorphism.

## Where code lives

- **`packages/`** holds each package's DSL-based module definitions, plus source-language helpers used to write them.
- **`heads/`** holds per-host runtimes that run those modules after translation to a target language.
- **`overlay/`** holds hand-written, language-specific source that is *copied onto* the generated
  distribution packages to make them complete. The governing equation:
  **`dist/<lang>/<pkg>/` = transform(`packages/<pkg>/`) + copy(`overlay/<lang>/<pkg>/`)** — a
  distribution package is generated modules plus copied overlay files. Two invariants: (1) **only the
  copy step reads `overlay/`** — nothing else may reference it; (2) **heads depend on `dist/`**, never on
  `overlay/` or on `heads/` for shipped runtime (generation drivers and a head's own test runners are not
  shipped and stay in `heads/`). Structure: `overlay/<lang>/<package>/src/...`. All ten hosts
  have `overlay/<lang>/hydra-kernel/` populated (Haskell also has the `overlay/haskell/hydra`
  umbrella). Downstream-package overlays exist only where host-specific integrations live:
  `overlay/java/hydra-pg`, `overlay/python/hydra-pg`, `overlay/java/hydra-rdf`. Extending
  downstream-package overlay coverage to other hosts is deferred (#434). Authored, not
  generated; never compiled in place. (#418, #434)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CategoricalData/hydra](https://github.com/CategoricalData/hydra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
