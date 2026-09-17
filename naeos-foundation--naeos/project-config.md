---
trigger: always_on
description: <!-- Copyright 2024-2026 NAEOS Foundation -->
---

<!-- Copyright 2024-2026 NAEOS Foundation -->
<!-- SPDX-License-Identifier: Apache-2.0 -->

NAEOS Repository Legal & License Audit Instructions

Mission

Perform a comprehensive, read-only legal and licensing audit of the NAEOS repository.

Repository:
https://github.com/NAEOS-foundation/naeos

Primary licensing assumption:

- NAEOS Core is intended to use Apache License 2.0.
- Do NOT modify any repository files during this audit.
- Do NOT change LICENSE, NOTICE, package manifests, source files, or contributor documentation.
- Produce findings and recommendations first.
- Any remediation must require explicit approval after the audit.

The goal is to determine whether the current repository is legally and operationally ready for:

1. Apache License 2.0 open-source distribution
2. External contributors
3. Commercial adoption
4. Enterprise customers
5. Future NAEOS Cloud / Enterprise products
6. Founder and investor due diligence
7. Future trademark and IP protection

---

1. Audit Principles

Follow these principles:

- Treat repository evidence as the primary source.
- Never assume ownership of code without evidence.
- Never assume a dependency license is compatible without checking it.
- Distinguish copyright, patent rights, trademark rights, and contractual rights.
- Distinguish open-source code from proprietary/commercial components.
- Flag uncertainty explicitly.
- Never provide a legal conclusion where repository evidence is insufficient.
- Separate technical findings from legal recommendations.
- Do not silently fix problems.
- Do not rewrite existing licensing language during the audit.

Use the following severity levels:

CRITICAL
HIGH
MEDIUM
LOW
INFO

---

2. Repository Inventory

First create a complete repository inventory.

Inspect:

- root files
- source directories
- packages
- modules
- libraries
- CLI
- runtime
- kernel
- policy
- governance
- specification
- infrastructure
- prompts
- tools
- examples
- documentation
- tests
- generated files
- configuration files
- scripts
- CI/CD workflows
- Dockerfiles
- deployment manifests
- lockfiles
- package manifests
- vendored code
- bundled third-party assets

Do not assume directories are correctly categorized.

Create a table:

Path| Component| Type| First-party/Third-party/Unknown| License Evidence| Risk

---

3. LICENSE Audit

Inspect:

- LICENSE
- LICENSE.txt
- LICENSE.md
- NOTICE
- COPYING
- README files
- package metadata
- repository metadata
- source headers
- documentation headers
- generated artifacts

Determine:

1. Is Apache License 2.0 clearly present?
2. Is the license text complete and unmodified?
3. Are there conflicting licenses?
4. Are there files without clear licensing information?
5. Are there directories containing different licensing terms?
6. Are examples licensed separately?
7. Are documentation and source code treated consistently?
8. Are generated files covered?
9. Are scripts covered?
10. Are prompt files covered?
11. Are configuration files covered?

Report every conflicting or ambiguous license.

---

4. SPDX and Copyright Headers

Inspect source files for:

- SPDX-License-Identifier
- copyright notices
- author attribution
- license headers
- generated-file notices

Identify inconsistent patterns.

Report:

- missing headers
- incorrect SPDX identifiers
- contradictory license declarations
- obsolete copyright notices
- suspicious ownership claims
- files claiming a license different from the repository license

Do NOT automatically add headers.

---

5. Third-Party Dependency Audit

Identify every third-party dependency.

For each dependency determine:

- package name
- version
- ecosystem
- direct/transitive
- license
- copyright obligations
- attribution requirements
- NOTICE requirements
- patent implications
- compatibility with Apache-2.0
- whether source redistribution is required
- whether modifications trigger additional obligations

Inspect:

- package.json
- package-lock.json
- pnpm-lock.yaml
- yarn.lock
- requirements.txt
- pyproject.toml
- poetry.lock
- Cargo.toml
- Cargo.lock
- go.mod
- go.sum
- Gemfile
- Gemfile.lock
- Maven/Gradle files
- Docker dependencies
- GitHub Actions
- vendored dependencies
- downloaded binaries
- embedded libraries

Do not rely solely on package metadata.

When possible, verify licenses from the upstream project.

Flag especially:

- GPL
- AGPL
- LGPL
- SSPL
- source-available licenses
- non-commercial licenses
- proprietary licenses
- custom licenses
- unknown licenses

Pay particular attention to copyleft dependencies that may affect distribution.

---

6. License Compatibility Matrix

Create a compatibility matrix:

Dependency| License| Apache-2.0 Compatible?| Modification Risk| Distribution Risk| Action

Use:

SAFE
REVIEW
BLOCKER

Do not classify a license as SAFE merely because it is commonly used.

Explain the reasoning.

---

7. Trademark Audit

Search the repository for:

- NAEOS
- NAEOS Foundation
- NAEOS logo references
- product names
- domain names
- organization names
- third-party trademarks

Determine whether the repository accidentally implies:

- trademark permission
- endorsement
- affiliation
- ownership
- certification

Apache-2.0 does NOT grant trademark rights.

Recommend explicit trademark language where appropriate.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NAEOS-foundation/naeos](https://github.com/NAEOS-foundation/naeos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
