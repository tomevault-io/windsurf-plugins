---
trigger: always_on
description: > **NOTE FOR HUMANS:** these docs use [Mermaid](https://mermaid.js.org/) diagrams. To view them rendered, install a Mermaid-aware Markdown viewer — e.g., the **"Markdown Preview Mermaid Support"** extension in VS Code, or read the files on GitHub (which renders Mermaid in `.md` files natively). Without one, the diagram code blocks will display as plain text.
---

# AGENTS.md — pyGSTi orientation for coding agents

> **NOTE FOR HUMANS:** these docs use [Mermaid](https://mermaid.js.org/) diagrams. To view them rendered, install a Mermaid-aware Markdown viewer — e.g., the **"Markdown Preview Mermaid Support"** extension in VS Code, or read the files on GitHub (which renders Mermaid in `.md` files natively). Without one, the diagram code blocks will display as plain text.

This folder is hierarchical orientation material for agents (and new group developers) working on **pyGSTi** itself. It covers architecture, key abstractions, and non-obvious gotchas. It does *not* teach pyGSTi as a library — for that, read the jupyter-book at [docs/markdown/](../docs/markdown/) in the source tree.

Read this file first, then jump into the subsystem doc(s) relevant to your task.

> **A note to agents on doc accuracy.** These docs are intentionally maintained at a "not overtly wrong" bar, not at a "perfectly accurate" one — they're a hint system to reduce trial-and-error, not a contract. **If, while doing real work, you find that what these docs say contradicts what the code actually does, treat the code as authoritative and flag the discrepancy to the user.** Be specific: name the file, the section, the claim, and the contradicting code path. Don't silently work around it, and don't edit the docs yourself unless the user asks you to.

## What pyGSTi is

pyGSTi is a Python framework for **modeling and analyzing collections of qudits** (commonly qubits, but the package supports higher-dimensional qudit spaces). Its most prominent capability is **gate set tomography (GST)** — fitting a [`Model`](../pygsti/models/model.py) of a noisy quantum device from circuit-outcome count data. Beyond that, it covers randomized benchmarking (RB), robust phase estimation (RPE), drift characterization, and standalone circuit simulation. Some protocols (notably the RB family) intentionally do *not* produce a full Model and instead estimate scalar performance metrics on structured random circuits. 

The "data" pyGSTi consumes can come from a real experiment or be sampled from a simulation — there is nothing experiment-specific about the data path. pyGSTi's **report generator**, which renders fit results and diagnostics into interactive HTML/PDF/notebook reports, is itself a major user-facing capability.

## The mental model

The central triad is **`Model` ↔ `Circuit` ↔ `DataSet`**:

- A [`Model`](../pygsti/models/model.py) describes a (possibly noisy) quantum device — a parameter vector with structure, typically equipped with a forward simulator.
- A [`Circuit`](../pygsti/circuits/circuit.py) is a program that can run on such a device — a sequence of `Label` objects.
- A [`DataSet`](../pygsti/data/dataset.py) records circuit outcomes (real or simulated).

A [`Protocol`](../pygsti/protocols/protocol.py) is the user-facing object for estimating a model from data: it consumes a `ProtocolData` (which pairs an `ExperimentDesign` with a `DataSet`) and produces a `ProtocolResults`. For GST, the result contains updated Models; for RB, a scalar error rate; for any protocol with reportable outputs, a `Report` can be generated downstream.

If you internalize that triad and the Protocol-on-top picture, the rest of the subsystem docs are about *how* each piece is implemented.

## Layout of `agent-docs/`

| Doc | Subpackages | Read when you need to… |
|---|---|---|
| [01-representation.md](01-representation.md) | `baseobjs`, `circuits`, `models`, `modelmembers`, `evotypes` | add/change a gate parameterization, state, POVM, instrument, or a parameterization mode; understand Model internals |
| [02-forward-simulation.md](02-forward-simulation.md) | `forwardsims`, `layouts` | change how circuit outcomes get computed; tune MPI configuration |
| [03-data-and-fitting.md](03-data-and-fitting.md) | `data`, `objectivefns`, `optimize`, `algorithms` | change how data is stored, or how a fit's objective / optimizer / loop behaves |
| [04-orchestration.md](04-orchestration.md) | `protocols`, `drivers` | add a new high-level workflow; understand the Protocol class API users actually call |
| [05-reporting-and-persistence.md](05-reporting-and-persistence.md) | `report`, `io`, `serialization` | touch reports, HTML/PDF/notebook output, or persistence formats |
| [06-modelpacks-and-processors.md](06-modelpacks-and-processors.md) | `modelpacks`, `processors` | add a named (and annotated) gate set, or a device description |
| [07-tools-library.md](07-tools-library.md) | `tools` | reach for a basis transform, channel conversion, MPI helper, or other utility |
| [08-domain-plugins.md](08-domain-plugins.md) | `extras`, `errorgenpropagation` | work on RB / RPE / drift / error-generator propagation |
| [known-debt.md](known-debt.md) | — | the area you're touching has a known smell or in-flight redesign |

Files are numbered for stable referencing only — not as a recommended reading order.

## Cross-cutting concerns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sandialabs/pyGSTi](https://github.com/sandialabs/pyGSTi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
