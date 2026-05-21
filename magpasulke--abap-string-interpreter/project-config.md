---
trigger: always_on
description: ZASIS is an ABAP-based String Interpreter that allows configuring RuleSets to extract structured information from strings (e.g. barcodes, data matrix codes, scanned values). It uses regex-based rules (MATCH and REPLACE types) with configurable offsets and supports custom logic extensibility.
---

# ZASIS - ABAP String Interpreter

## Project Overview

ZASIS is an ABAP-based String Interpreter that allows configuring RuleSets to extract structured information from strings (e.g. barcodes, data matrix codes, scanned values). It uses regex-based rules (MATCH and REPLACE types) with configurable offsets and supports custom logic extensibility.

**This is an ABAP project without a live SAP system connection.** All source files are stored in the **abapGit serialized file format** so the repository can be synced to an ABAP system via [abapGit](https://docs.abapgit.org/). The workspace contains `.clas.abap`, `.intf.abap`, `.ddls.asddls`, `.tabl.xml`, `.doma.xml`, `.dtel.xml`, `.bdef.asbdef`, `.srvd.srvdsrv`, and other abapGit-standard file types. There are many agent skills available (prefixed `abapgit-*`) that help create files in the correct abapGit format for each object type (CLAS, INTF, TABL, DDLS, DOMA, DTEL, BDEF, SRVD, SRVB, FUGR, etc.). **Always use the appropriate skill when creating new ABAP repository objects** to ensure correct file structure and XML metadata.

**Target System**: ABAP Cloud Trial 2022 SP01 (ABAP Platform 2022 / SAP BASIS 757 SP0004).

---

## Git Workflow

**NEVER commit or push directly to `main`.** Always follow this workflow:

1. Create a feature branch (e.g., `feat/my-feature`, `fix/my-bugfix`)
2. Commit changes to the feature branch, **always use the `conventional-commit` skill**
3. Push the feature branch to the remote
4. Create a Pull Request (PR) against `main`
5. Merge the PR (squash preferred)
6. Delete the feature branch (local + remote)

This applies to ALL changes — code, documentation, skills, configuration. No exceptions.

**PR merging is the user's responsibility by default.** Agents must:
- **Ask the user** before creating a PR (do not create PRs autonomously)
- **Never merge PRs** unless the user explicitly instructs the agent to merge
- Wait for the user's explicit confirmation before creating or merging

**Clean commit history on feature branches.** Agents must:
- **Never amend commits** — each change gets its own commit
- **Never force-push** (`--force`, `--force-with-lease`) — history must remain linear and traceable
- If a fix is needed after a commit, create a new commit with a clear message (e.g., `fix: resolve duplicate attribute in zasis_cx_exc`)
- The commit history should tell the story of what happened, including fixes

---

## Architecture & Package Structure

```
src/                          Root package (ZASIS)
├── app/                      Application / UI layer (placeholder for future UI components)
├── bo/                       Business Objects — core domain logic & RAP service
├── config/                   Configuration management & eventing
│   └── eventing/             Event producer configuration & maintenance
├── dm/                       Data Model — database tables, domains, data elements, CDS views
├── srv/                      HTTP Service — REST API handler (GET/POST for RuleSet operations)
└── utils/                    Shared utilities — constants, exceptions, domain value helpers
```

### dm (Data Model)

Database schema and type definitions:

- **Tables**: `zasis_rulesethd` (RuleSet header), `zasis_rulesetitm` (RuleSet items/rules), `zasis_ruleset_refs` (cache)
- **CDS Views**: `zasis_i_ruleset` (composite root), `zasis_i_rulesetheader`, `zasis_i_rulesetitem`
- **Domains/Data Elements**: Types for UUID, RuleSet ID, regex patterns, offsets, target fields, interpretation types (MATCH=1, REPLACE=2)
- **Table Types**: `zasis_tt_rulesetitm`, `zasis_tt_interpretationresult`, `zasis_tt_rulesetrefs`

### bo (Business Objects)

Core domain logic, RAP behavior, and consumption layer:

- **`zasis_cl_interpreter`** — Main execution engine; interprets strings against rulesets using regex MATCH/REPLACE rules
- **`zasis_cl_ruleset`** — Immutable ruleset container (header + items)
- **`zasis_cl_ruleset_factory`** — Factory with in-memory caching and auth checks
- **`zbp_asis_i_ruleset`** — RAP Behavior Implementation for managed entity with draft
- **Consumption CDS**: `zasis_c_ruleset`, `zasis_c_rulesetitem` (Fiori Elements annotations)
- **Service Definition**: `zasis_ui_ruleset.srvd` — OData V4 service exposing RuleSet and RuleSetItem
- **Interfaces**: `zasis_if_interpreter`, `zasis_if_ruleset`, `zasis_if_customlogic`

### srv (HTTP Service)

REST API for external consumers:

- **`zasis_cl_http_handler`** — Routes GET (retrieve RuleSet) and POST (execute RuleSet) requests
- Local request validator for path parsing and content-type checks

### config (Configuration & Eventing)

- Event configuration tables and maintenance function group (`zasis_conf_maint`)
- `zasis_if_event_producer` interface for event-driven extensibility

### utils (Utilities)

- **`zasis_constants`** — Static constants (rule types, HTTP methods, content types)
- **`zasis_cx_exc`** — Custom exception class with T100 messages (invalid route, unknown ruleset, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MagPasulke/abap-string-interpreter](https://github.com/MagPasulke/abap-string-interpreter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
