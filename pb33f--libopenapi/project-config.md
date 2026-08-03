---
trigger: always_on
description: `github.com/pb33f/libopenapi` is a Go library for parsing, indexing, mutating, bundling, diffing, overlaying, rendering, and mocking OpenAPI/OAS-adjacent documents. It is the engine behind vacuum, wiretap, openapi-changes, printing press, and the pb33f platform. When code, tests, and external docs disagree, code is canonical.
---

# AGENTS.md

`github.com/pb33f/libopenapi` is a Go library for parsing, indexing, mutating, bundling, diffing, overlaying, rendering, and mocking OpenAPI/OAS-adjacent documents. It is the engine behind vacuum, wiretap, openapi-changes, printing press, and the pb33f platform. When code, tests, and external docs disagree, code is canonical.

## Context

This repo is a library, not an app. The root package exposes the public entry points (`NewDocument`, `CompareDocuments`, overlay/arazzo helpers), while most real behavior lives in subsystem packages. The highest-risk edits are in `index/`: it owns reference extraction, cross-document lookup, circular analysis, schema-id resolution, and performance-sensitive caches.

## Packages

| Package | Purpose |
|---|---|
| `.` | Public API surface: document creation, model building, render/reload, compare, cache clearing, overlay/arazzo entry points |
| `index/` | Core indexing engine: `SpecIndex`, `Rolodex`, lookup, resolver, `$id` registry, origin search, local/remote file systems |
| `datamodel/low/` | YAML-backed low-level models and generic builders; source of truth for comments, line/column, refs, hashing |
| `datamodel/high/` | Mutable high-level facades plus node builders/rendering back to YAML/JSON |
| `datamodel/` | Spec parsing/version detection, schemas, and `DocumentConfiguration` |
| `bundler/` | Inline and composed bundling, ref rewrite/composition, origin tracking |
| `what-changed/` | Unified diff engine for OAS2 and OAS3+, plus breaking-change rule config/report helpers |
| `overlay/` | OpenAPI Overlay application engine |
| `arazzo/` | Arazzo parsing, resolution, validation, and workflow execution engine |
| `renderer/` | Schema/mock sample generation |
| `orderedmap/` | Stable insertion-ordered map wrapper used throughout models/rendering |
| `json/` | YAML-node to ordered JSON conversion |
| `tests/` | Cross-package integration and benchmark coverage, especially sibling-ref behavior |
| `test_specs/` | Realistic fixtures and regression specs used across packages |

## Key Paths

| Path | Purpose |
|---|---|
| `document.go` | Root orchestration layer; keep it thin |
| `index/doc.go` | Best summary of `index` subsystem boundaries and invariants |
| `index/index_model.go` | `SpecIndex`, config, caches, release lifecycle |
| `index/spec_index_build.go` | Index construction/build sequencing |
| `index/rolodex.go` | Cross-document lookup ownership and lifecycle |
| `index/extract_refs.go` | Reference discovery entry point |
| `index/find_component_entry.go` | Component lookup entry path |
| `index/search_index.go` | Reference search flow, cache usage, schema-id lookup |
| `index/resolver_entry.go` | Circular detection and destructive resolution entry point |
| `datamodel/document_config.go` | Canonical config surface for documents/index/bundler behavior |
| `datamodel/spec_info.go` | Spec parsing, version detection, JSON conversion, `$self` handling |
| `datamodel/low/v3/create_document.go` | V3 document/index/rolodex assembly |
| `datamodel/low/model_builder.go` | Reflection-driven low-model population |
| `datamodel/high/node_builder.go` | High-model re-rendering/mutation path |
| `bundler/bundler.go` | Public bundling entry points/config |
| `bundler/bundler_composer.go` | Composed bundling and component lifting |
| `what-changed/model/document.go` | Unified change model and compare flow |
| `what-changed/model/breaking_rules.go` | Default/custom breaking-change policy |
| `.github/workflows/build.yaml` | CI shape: Linux + Windows `go test ./...`, coverage upload |

## Commands

| Command | Purpose |
|---|---|
| `go test ./...` | Canonical full test suite |
| `go test -coverprofile=coverage.out ./...` | CI-style coverage run |
| `go test ./index ./bundler ./what-changed/...` | Fast pass over the most coupled subsystems |
| `go test ./tests -run SiblingRefs` | Target sibling-ref integration surface |
| `go test ./index -run TestSpecIndex` | Target index-heavy regressions |
| `go test ./bundler -run TestBundle` | Target bundler regressions |
| `go test ./what-changed/... -run Test` | Target diff/breaking-rule regressions |
| `go test -bench . ./index ./datamodel/low/... ./what-changed/...` | Run benchmarks in hot paths |
| `GOCACHE=/tmp/go-build go test ./...` | Useful in restricted sandboxes where default Go build cache is not writable |

## Testing Caveats

- `go.mod` is the authoritative toolchain target: Go `1.25.0`. CI still shows `1.23`; prefer `go.mod` when they conflict.
- Some tests use `httptest.NewServer` and require local loopback socket binding.
- Some bundler/index tests clone or fetch pinned external specs (notably DigitalOcean and `raw.githubusercontent.com` fixtures).
- In network-restricted or socket-restricted sandboxes, prefer targeted offline package tests and report environment-caused failures explicitly instead of treating them as code regressions.

## Rules & Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pb33f/libopenapi](https://github.com/pb33f/libopenapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
