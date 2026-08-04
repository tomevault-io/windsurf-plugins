---
trigger: always_on
description: generates the content-root attribute. An in-test-assembly host fails without a solution.
---

# CLAUDE.md — working on this repo

This is an AI hand-off guide. Read it before making changes. It's terse on purpose;
`README.md`, `CONTRIBUTING.md`, `RELEASING.md`, `eng/compat/README.md`, and
`eng/runtime/README.md` have the long form.

## What this is

A curated set of **ABP Framework** agent skills, distributed as **15 versioned
plugins / 80 skills** to four AI coding tools (Claude Code, Codex CLI, Cursor,
VS Code/Copilot). One set of Markdown skill content, published through four
per-tool marketplace manifests. Scope: ABP Framework development (building and
extending modules through the framework's extension points), not tutorials.

## Repo layout (key paths)

```text
plugins/<plugin>/
  plugin.json                 # Claude Code / Cursor manifest
  .codex-plugin/plugin.json   # Codex manifest (shared fields MUST match plugin.json)
  skills/<skill>/SKILL.md      # the skill (frontmatter + body), shared by all 4 tools
.claude-plugin/ .agents/ .cursor-plugin/ .github/plugin/   # 4 marketplace.json (entries must be identical)
tests/<plugin>/<skill>/eval.yaml   # routing/shape eval (keyword assertions, NOT executable)
eng/compat/                        # C# compile-smoke: 1 isolated project per C# skill
  Skills/<PascalCase>.cs       # references the ABP APIs that skill teaches
  generate-projects.py         # PACKAGES map -> regenerates csproj + AbpSkillsCompat.slnx
  Directory.Packages.props     # central pkg mgmt, <AbpVersion>10.5.0</AbpVersion>
  coverage-exemptions.yaml     # skills with no C# surface (TS/CLI/process) + reason
eng/compat-ts/                     # TypeScript compile-smoke, Angular 21 (@abp/ng.core)
eng/compat-ts-extensible/          # SEPARATE TS workspace, Angular 20 (extend-angular-module-ui; see gotchas)
eng/runtime/                       # real ABP app behavior tests (xUnit); CPM via eng/runtime/Directory.Packages.props
  webapp/                      # minimal Web SDK SUT for AbpWebApplicationFactoryIntegratedTest
eng/                           # validators, eval harness, A/B experiment, baselines, unit tests
```

## Quality gates — run the full sweep before claiming "done"

```bash
python3 eng/validate.py                 # layout/manifest/frontmatter/menu-budget
python3 eng/validate_evals.py           # eval schema + 100% skill coverage + unique names
python3 eng/verify_compat_coverage.py   # every skill: C# smoke OR exemption
python3 eng/check_version_annotations.py # "ABP 10.6+" symbols registered + #if ABP_NEXT
python3 eng/verify_baselines.py         # AbpVersion consistent across props/eng/runtime/ts/README/workflows
python3 -m unittest discover -s eng -p 'test_*.py'
dotnet build eng/compat/AbpSkillsCompat.slnx                                   # stable 10.5.0
dotnet build eng/compat/AbpSkillsCompat.slnx -p:AbpVersion=10.6.0-rc.1 -p:AbpNext=true  # next ABP
dotnet test  eng/runtime/AbpRuntimeTests.csproj                                # runtime behavior
(cd eng/compat-ts && npm ci && npm ls @abp/ng.core @angular/core && npm run build)
(cd eng/compat-ts-extensible && npm ci && npm run build)   # no `npm ls` here (see gotchas)
npx --yes markdownlint-cli2@0.23.0
actionlint    # needs shellcheck on PATH; also in .github/workflows/actionlint.yml
```

All must be green. `git diff --check` too. CI runs the same set in
`.github/workflows/validate.yml` (PR/push) and `release.yml` (tag).

## Conventions

- **SKILL.md frontmatter**: `name` (== folder name, globally unique — routing is
  name-based), `description` (≤1024 chars, **no angle brackets `<>`**, must contain
  `USE FOR:` and `DO NOT USE FOR:` markers with sibling-skill routing), `license: MIT`.
- **Framework code in examples**: match ABP idiom; new framework methods a customer
  might override default to `protected virtual`.
- **No hedging or unverifiable claims in shipped content.** Every claim must be
  source-verifiable (see next section); the maintainers are responsible for verifying
  all content. Follow the organization's commit policy and don't add false attribution.
- **Central Package Management**: `eng/compat/` and `eng/runtime/` each have a
  `Directory.Packages.props`. Do NOT hard-code `Version="..."` on `PackageReference`;
  add a `PackageVersion` to the props (ABP packages use `$(AbpVersion)`).
- **Marketplace/plugin descriptions**: if you change a plugin's description, update
  all 6 places (plugin.json, .codex-plugin/plugin.json, 4 marketplace.json).

## Verifying content against ABP source (mandatory)

Every ABP API/behavior claim needs source evidence. Read the real files:

- Check out the ABP source at the baselines recorded in `eng/abp-baselines.yaml`:
  **rel-10.5** is the compile baseline and the `blob/rel-10.5/...` doc links in skills;
  **rel-10.6** is the content-review baseline (use `git show <commit>:<path>` for the
  exact commit). ABP Studio (Studio CLI / templates) is in the `abp-studio` repo.
- Grep for the type/method, confirm it exists with the signature you're writing, note
  `file:line`. Independent reviewers (see next) have caught real bugs from prose that
looked right but wasn't (e.g. `SetUseCurrentAccessToken` vs a non-existent property,
`DeleteDirectAsync` provider differences, `KnownNetworks` deprecated in net10).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abpframework/abp-skills](https://github.com/abpframework/abp-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
