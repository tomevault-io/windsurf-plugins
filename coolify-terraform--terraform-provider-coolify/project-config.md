---
trigger: always_on
description: Read these skills when working in this repo:
---

# AGENTS.md

## Skills

Read these skills when working in this repo:

- `~/.grok/skills/coolify-test-instance/SKILL.md` — Local Coolify setup, API quirks, SSH validation, acc test troubleshooting. **Read first** when setting up a test instance or debugging real-API failures.
- `~/.grok/skills/terraform-provider-coolify-contrib/SKILL.md` — Contributing conventions for the Coolify upstream project, repo namespace notes, CI gotchas.
- `~/.grok/skills/terraform-provider/SKILL.md` — General Terraform provider patterns (resource implementation, testing, CI, releases).
- `~/.grok/skills/ci-workflow-hygiene/SKILL.md` — CI workflow rules: concurrency groups, timeouts, action versions, security scanners.

## Project

Terraform provider for [Coolify](https://coolify.io/), the open-source self-hosted PaaS.
Built with Go 1.26, Terraform Plugin Framework v1.19, and GoReleaser for releases.
Builds use `GOFIPS140=latest` for FIPS 140-3 compliant cryptography (required for
government/enterprise adoption; set in `.goreleaser.yml` and `release.yml` smoke test).
33 resources, 44 data sources, 980+ tests (unit + acceptance), 9 CI jobs.
17 ACME Corp scenario examples (all with `terraform test` integration tests; acme-private-repo uses plan-only).

## Source of Truth: Coolify Source Code (NOT OpenAPI spec)

**Never guess Coolify API behavior. You have access to the real source code.**

The Coolify project is open source at `github.com/coollabsio/coolify`. When
you encounter ANY question about how the API works, the answer is in the PHP
source code. Do not hypothesize, do not assume, do not test by trial and error.
Read the code.

### When to read the source

- A field isn't returned on GET? Read the controller's response builder.
- An update returns 422? Read the controller's `$allowedFields` and validation rules.
- A field has a surprising default? Read the migration and model `$attributes`.
- A flatten function causes "inconsistent result"? Read the model's accessors
  to see if the API normalizes the value (base64 encode/decode, path prefixing, etc.).
- The OpenAPI spec says one thing but the API does another? The spec is wrong.
  The source code is the truth.

### How to read the source

```bash
# Clone once per session (or reuse /tmp/coolify if already there)
git clone --depth 1 https://github.com/coollabsio/coolify.git /tmp/coolify
```

| Question | Where to look |
|----------|--------------|
| What fields exist on a resource? | `app/Models/Application.php` (`$fillable`) |
| What type is a field? What's the default? | `database/migrations/` (column definitions) |
| Is a field encrypted/sensitive? | Model `$casts` (look for `'encrypted'`) |
| What does the API accept on create/update? | `app/Http/Controllers/Api/ApplicationsController.php` (`$allowedFields`, `$request->validate()`) |
| What validation rules apply? | Controller + `bootstrap/helpers/api.php` (`sharedDataApplications()`) |
| What regex patterns are enforced? | `app/Support/ValidationPatterns.php` |
| What does the API return after create? | Controller method (look for `return response()->json(...)`) |
| Does updating a field trigger a side effect? | Controller method (look for `queue_application_deployment`, `StartService`, etc.) |
| What are the enum values? | `app/Enums/` directory |
| Where are settings stored? | `app/Models/ApplicationSetting.php` (separate table from Application) |

### The contract extraction pipeline

We have automated extraction that reads the source and produces a machine-readable
contract JSON. Use it as the first check, then go deeper into the PHP when needed:

1. `testdata/contracts/coolify-v4.json` -- extracted contract (check here first)
2. `/tmp/coolify/app/Models/*.php` -- the real model when the contract isn't enough
3. `/tmp/coolify/app/Http/Controllers/Api/*.php` -- the real controller for validation logic

The OpenAPI spec at `testdata/specs/` is generated FROM the contract. Do NOT
treat it as authoritative. It had 22 wrong nullability annotations, 3 type
mismatches, and zero validation rules when we compared it against the source.

## Commands

- **Run all checks before pushing**: `make ci` + targeted acceptance tests
- **Note**: `make ci` does NOT include acceptance tests. It DOES include `python-test`, so ensure Python 3.9+ is available locally. For real-API changes, run `make acc-preflight` first, then `make testacc-pkg PKG=./internal/service/<changed-package>/` for serialized package-scoped acceptance coverage, or `make testacc` for the full suite.
- **If `make ci` fails on `check-tfplugindocs`**: install it with `cd tools && GOBIN=$(cd .. && pwd)/bin go install github.com/hashicorp/terraform-plugin-docs/cmd/tfplugindocs` then re-run with `PATH="$(pwd)/bin:$PATH" make ci`. Do NOT skip `make ci` and run individual targets instead; that misses `docs-check` and causes CI failures when schema descriptions or templates change.
- Use `make testacc` for the full suite when changing shared code (client, provider, flex, validate)
- Build: `make build`
- Test (all, with race detector): `make test`
- Test (single package): `make test-pkg PKG=./internal/service/project/`
- Acceptance tests (needs running Coolify): `make testacc`
- Lint: `make lint`
- Format: `make fmt`
- Generate docs: `make docs`
- Validate HCL examples: `make validate`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coolify-terraform/terraform-provider-coolify](https://github.com/coolify-terraform/terraform-provider-coolify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
