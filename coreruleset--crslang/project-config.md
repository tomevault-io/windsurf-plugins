---
trigger: always_on
description: **CRSLang** is a Go library and CLI tool that translates [OWASP CRS](https://coreruleset.org/) rules between their native [SecLang/ModSecurity](https://github.com/coreruleset/seclang_parser) configuration format (`.conf` files) and a new, language-independent YAML representation called **CRSLang**.
---

# CRSLang – Copilot Agent Onboarding Instructions

## Project Overview

**CRSLang** is a Go library and CLI tool that translates [OWASP CRS](https://coreruleset.org/) rules between their native [SecLang/ModSecurity](https://github.com/coreruleset/seclang_parser) configuration format (`.conf` files) and a new, language-independent YAML representation called **CRSLang**.

The project also compiles to WebAssembly (WASM) so the translation logic can run in a browser without any server-side component.

---

## Repository Layout

```
crslang/
├── main.go                    # CLI entry point (seclang ↔ CRSLang conversion)
├── mage.go / magefile.go      # Mage build targets (generate, build, test)
├── Makefile                   # Short-cut targets: build, test, wasm, clean
├── go.mod / go.sum            # Go module definition (module: github.com/coreruleset/crslang)
│
├── listener/                  # ANTLR listener that walks the parse tree and
│   │                          # produces internal types.* structs
│   └── extended_seclang_parser_listener.go
│       (+ sec_rule.go, sec_action.go, variables.go, actions.go, …)
│
├── translator/                # Orchestration layer
│   ├── seclang.go             # LoadSeclang / LoadSeclangFromString / PrintSeclang
│   ├── crslang.go             # ToCRSLang (seclang → CRSLang conversion)
│   ├── file.go                # writeToFile helper
│   └── translator_test.go     # Round-trip and unit tests
│
├── types/                     # Core CRSLang type definitions + YAML serialisation
│   ├── configuration.go       # ConfigurationList, DirectiveList, ExtractDefaultValues
│   ├── secrule.go             # SecRule (variables, operator, actions, chaining)
│   ├── secaction.go           # SecAction
│   ├── metadata.go            # SecRuleMetadata, CommentMetadata, etc.
│   ├── operators.go           # Operator type + all operator constants
│   ├── variables.go           # Variable type + all variable-name constants
│   ├── collections.go         # Collection type + all collection-name constants
│   ├── actions.go             # Action type + all action constants
│   ├── transformations.go     # Transformation type + all transformation constants
│   ├── condition_directives.go# RuleWithCondition (CRSLang rule representation)
│   └── *_test.go              # Per-file unit tests
│
├── wasm/
│   ├── main.go                # WASM build (js/wasm build tag)
│   │                          # Exports: seclangToCRSLang(), crslangToSeclang()
│   └── demo.html              # Browser demo page
│
├── base_test.go               # Lexer/parser smoke tests (TestSecLang, TestCRSLang, …)
├── listener_test.go           # Full listener round-trip tests (TestLoadSecLang, …)
│
└── testdata/                  # Test fixture .conf files
    ├── crs/                   # Subset of OWASP CRS rule files
    ├── plugins/               # Sample plugin configuration files
    └── test*.conf             # Individual unit-test fixtures
```

---

## Key Concepts

### Data Flow

```
.conf file(s)
    └─► ANTLR SecLang lexer/parser  (github.com/coreruleset/seclang_parser)
            └─► ExtendedSeclangParserListener  (listener/)
                    └─► types.ConfigurationList  (raw Seclang structs)
                            └─► translator.ToCRSLang()
                                    └─► types.ConfigurationList  (CRSLang structs)
                                            └─► YAML marshal  →  crslang.yaml
```

Reverse path:

```
crslang.yaml
    └─► types.LoadDirectivesWithConditionsFromFile()
            └─► types.ConfigurationList  (CRSLang structs)
                    └─► types.FromCRSLangToUnformattedDirectives()
                            └─► DirectiveList.ToSeclang()  →  .conf files
```

### Important Types

| Type | Package | Description |
|---|---|---|
| `ConfigurationList` | `types` | Top-level container; holds `Global` defaults + `[]DirectiveList` |
| `DirectiveList` | `types` | A group of directives (typically one `.conf` file); has an `Id` and optional `Marker` |
| `SecRule` | `types` | A parsed `SecRule` directive; Variables/Collections, Operator, Transformations, Actions, optional ChainedRule |
| `SecAction` | `types` | A parsed `SecAction` directive |
| `RuleWithCondition` | `types` | CRSLang representation of a rule with structured conditions |
| `Operator` | `types` | Operator name + value + optional negation flag |
| `SeclangActions` | `types` | Disruptive, Non-disruptive and Flow actions |

---

## Build & Run

### Prerequisites

- Go 1.22+ (`go.mod` declares `go 1.22.2`)
- No code-generation step is required; the ANTLR-generated parser is vendored via `github.com/coreruleset/seclang_parser`.

### Build the CLI

```bash
go build .
# Produces: ./crslang
```

### Run the CLI

```bash
# Convert a directory of CRS .conf files to CRSLang YAML
./crslang -o output_name path/to/rules/

# Convert CRSLang YAML back to SecLang .conf files
./crslang -s -o output_dir/ crslang.yaml
```

### Build the WASM module

```bash
make wasm

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coreruleset/crslang](https://github.com/coreruleset/crslang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
