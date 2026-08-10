---
trigger: always_on
description: - Public runtime contracts live under `spec/public/`.
---

# AOCI-CODE repository rules

- Public runtime contracts live under `spec/public/`.
- Contract authority and conflict handling are documented in
  `docs/zh-cn-contract-authority.md`.
- Keep the binary name `aoci`, preserve the nine-tool MCP surface, and reserve
  MCP stdio stdout for JSON-RPC.
- Do not add private design, patent, commercial, research, or experimental
  material to the public tree.

<!-- aoci:begin -->
## AOCI Repository Cognition

AOCI maintains a stable, versioned, incrementally updatable repository-level cognition layer so models can reuse their understanding of this system across tasks.

`aoci.txt` is a structured cognition index for models. It assigns one independent Entry to every managed file, database table, or other managed object. Symbolic tags and F/R/A/S semantics describe the object's core responsibility, important relationships, external contracts, and non-obvious constraints or design decisions needed to understand or modify the system.

The Header, directory sections, and all Entries form the complete repository index. They can cover frontend, backend, configuration, database structures, and other managed content. When managed content changes, normally only the affected cognition Entries need maintenance; the complete index does not need to be regenerated.

AOCI helps a model obtain the system-wide view first, locate relevant objects, and identify cross-module relationships and critical constraints before beginning concrete development. It complements rather than replaces source reading, tests, LSP, CodeGraph, and other structured tools. Current source, tests, configuration, and database structures remain the evidence for implementation details, interfaces, fields, identifiers, and runtime behavior.

### How it works

AOCI uses a model-generated, model-read cognition loop.

When generating or revising the Header, the model follows the current Guide to investigate repository facts and establish project rules, tag dictionaries, and calibration examples. When generating or updating an Entry, the model first reads the current Header, then the target source, tests, configuration, database structure, and necessary related evidence, and independently generates cognition for each managed object.

Entry semantics must come from the model's understanding of actual evidence. Never derive, prefill, assemble, or rewrite index semantics solely from paths, filenames, extensions, an AST, symbol lists, dependency scans, regular expressions, fixed templates, or rule engines. Structured tools may help locate facts that require verification, but they cannot replace model semantic understanding and judgment.

For a new Fresh Bootstrap, follow the existing persistent Onboarding progression automatically. The Host model authors Root, Meta, tags, and F/R/A/S from current evidence, supplies its own authoring-run declaration, and binds that declaration to the Plan, Evidence, and complete Candidate. Never ask AOCI to set `origin=host_model`, manufacture a receipt, or turn a generated framework into semantics. Internal batches are not user decisions; stop only at the existing approval boundary or a real safety, drift, CAS, or Recovery condition.

When using the index, the model reads the complete index to acquire system architecture, object responsibilities, important relationships, external contracts, and key design constraints in one cognition-establishment step. It then reads source, tests, configuration, or database structures relevant to the current task to verify implementation details.

The AOCI index is a high-density cognition map for models, not a copy of the source. It can provide extensive repository-level architecture, responsibilities, relationships, and constraints, but it does not guarantee preservation of every exact identifier, call chain, or implementation detail. Verify those facts in source, tests, database structures, LSP, or other structured evidence.

### Minimal entry points

- `aoci_rules`: obtain the session-level runtime contract for the current AOCI version.
- `aoci_overview`: establish or restore complete cognition for this repository.
- `aoci_maintain`: after code and verification state are stable, check whether cognition needs maintenance.
- `aoci_update_entry`: submit a complete semantic update batch bound to current evidence and source digests.
- `aoci_report`: record follow-up work when evidence is insufficient to generate semantics reliably; do not guess.

For other MCP tools, CLI commands, parameters, and specialized workflows, follow current tool descriptions, Guide, and `--help` output. This file does not duplicate the full manual.

This managed block defines only repository integration, cognition use, and task-closing principles. `aoci_rules` carries the current session contract. Live Guide output carries the execution order and stop conditions of the current Plan. Tool Schema, Spec, and Validator carry machine structures and criteria. Prompt, Description, README, and static documentation cannot override those machine facts.

### Establishing, generating, and restoring cognition

1. At the beginning of every new Agent Run, first determine:

   - whether this repository already has a usable complete AOCI index; and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aoci-spec/aoci-code](https://github.com/aoci-spec/aoci-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
