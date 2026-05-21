---
trigger: always_on
description: SPDX-FileCopyrightText: 2025 ash_neo4j contributors <https://github.com/diffo-dev/ash_neo4j/graphs.contributors>
---

<!--
SPDX-FileCopyrightText: 2025 ash_neo4j contributors <https://github.com/diffo-dev/ash_neo4j/graphs.contributors>

SPDX-License-Identifier: MIT
-->

# AGENTS.md — AshNeo4j

AI agent guidance for the AshNeo4j source repository.

## What this project is

AshNeo4j is an `Ash.DataLayer` that stores Ash resources as nodes in a Neo4j graph database.
It is a library published on hex.pm and maintained at `diffo-dev/ash_neo4j`. Its primary consumer
is the Diffo project; upstream bugs found while working in Diffo belong here.

## Before making changes

1. Read `usage-rules.md` — the canonical rules for using AshNeo4j, including naming conventions,
   relationship semantics, aggregate kinds, and the test sandbox.
2. Understand the label system (see **Label system** below) — the label concept is
   a frequent source of bugs and the most important thing to get right.
3. Run `mix test` before and after your change to confirm nothing regressed.

## Fixing bugs

Before writing any fix, review existing test coverage for the affected behaviour. If the bug
has no test, write the failing test first — this confirms the reproduction and guards the fix
against regression. Only then implement the fix and verify the test passes.

## Project structure

```
lib/
  data_layer.ex                — Ash.DataLayer behaviour: CRUD, aggregates, calculations,
                                 transaction, enrichments (OPTIONAL MATCH → source attributes)
  cypher.ex                    — Cypher string helpers: node/2, relationship/3, expression/5,
                                 parameterized_node/3, render/1, run/1
  cypher/query.ex              — Typed clause structs (Match, Where, Return, …) and builder
                                 functions for every query shape used by the data layer
  query_helper.ex              — Translates Ash.Query (filter, sort, offset, limit) into
                                 a Cypher.Query; entry point is query_nodes/1
  resource/info.ex             — All DSL introspection: label/1, module_label/1, domain_label/1,
                                 domain_fragment_label/1, all_labels/1, label_pair/1,
                                 mapping/1, relate/1, translations/1, and relationship helpers
  resource_mapping.ex          — %ResourceMapping{} struct (module, label, module_label,
                                 domain_fragment_label, all_labels, label_pair,
                                 properties, edges, guards, skip)
  edge_descriptor.ex           — %EdgeDescriptor{} struct (relationship, label, direction,
                                 destination_label)
  neo4j_helper.ex              — Low-level node/edge operations via Bolty
  data_layer/cast.ex           — Casts Neo4j return values to Ash types
  data_layer/dump.ex           — Dumps Ash values to Neo4j-compatible primitives
  data_layer/type_classifier.ex — Classifies types as :ash_json (embedded/struct/map) or scalar
  sandbox.ex                   — AshNeo4j.Sandbox: per-test transaction isolation
  util.ex                      — short_name/1, to_camel_case/1, reverse/1
  persisters/
    persist_labels.ex          — Computes and persists domain_label, module_label, label,
                                 domain_fragment_label, all_labels, label_pair
    persist_translations.ex    — Builds attribute → property name keyword list; excludes
                                 belongs_to source attributes and skip attributes
    persist_relate.ex          — Merges explicit relate DSL with default auto-generated edges
    persist_relationship_attributes.ex — Maps source attributes to relationship names
    persist_mapping.ex         — Bakes __ash_neo4j_mapping__/0 onto each resource module
  verifiers/
    verify_labels_pascal_case.ex
    verify_relate.ex
    verify_guard.ex
    verify_properties_camel_case.ex
    verify_enrichable.ex
    verify_attribute_type.ex

test/
  support/resource/            — Test resources (Post, Comment, Author, Specification, …)
  support/srm.ex               — Test domain (Srm)
  blog_test.exs                — CRUD, filter, relationship tests
  aggregate_test.exs           — All aggregate kinds including filtered and expr aggregates
  calculation_test.exs         — Expression calculations
  data_layer/                  — Unit tests for Cast, Dump, TypeClassifier, Info
```

## Label system

Every node has several distinct label concepts. Getting them confused is the most common
source of bugs:

| Name | Persisted as | Example | When used |
|---|---|---|---|
| `domain_label` | `:domain_label` | `:Servo` | Written on CREATE; also part of MATCH via `label_pair` |
| `module_label` | `:module_label` | `:ShelfInstance` | Written on CREATE; also part of MATCH via `label_pair` |
| `label` | `:label` | `:Instance` | May differ from `module_label` when a resource fragment declares a base type; written on CREATE only |
| `domain_fragment_label` | `:domain_fragment_label` | `:Telco` | Written on CREATE only — from a domain fragment using `AshNeo4j.DataLayer.Domain`; `nil` when none declared |
| `all_labels` | `:all_labels` | `[:Servo, :ShelfInstance, :Instance, :Telco]` | Full CREATE label list — `[domain_label, module_label, label, domain_fragment_label]` deduped |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [diffo-dev/ash_neo4j](https://github.com/diffo-dev/ash_neo4j) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
