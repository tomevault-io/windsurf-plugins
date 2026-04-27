---
trigger: always_on
description: - Use the `docs/` directory as the source of truth for internal project contracts and implementation-planning documents.
---

### Instructions

- Use the `docs/` directory as the source of truth for internal project contracts and implementation-planning documents.
- All repository-wide rules must be defined in this `AGENTS.md`.
- List files in `docs/` before starting each task, and keep `docs/` up-to-date.
- After completing each task, update the relevant `AGENTS.md` and `docs/` files in the same change when policies, structure, or contracts changed.
- Write all content in English, including code, comments, commit messages, PR titles, PR descriptions, issue titles, and issue bodies.
- Run `bash scripts/validate-skill-md.sh` before finishing any task that modifies skill bundle files.
- Run `bash skills/design-farmer/tests/run-all.sh` before finishing any task that modifies phase files, tests, or cross-phase contracts.
- Commit when each logical unit of work is complete; do NOT use the `--no-verify` flag.
- Run `git commit` only after `git add`; keep each commit atomic and independently revertible.
- After addressing pull request review comments and pushing updates, mark the corresponding review threads as resolved.
- When no explicit scope is specified and you are currently working within a pull request scope, interpret instructions within the current pull request scope.
- Do not guess; search the web instead.
- When accessing `github.com`, use the GitHub CLI (`gh`) instead of browser-based workflows when possible.
- Rules using MUST/NEVER are mandatory. Rules using prefer/whenever possible are guidance.

### Repository Structure Map

- `docs/`: Source of truth for internal project contracts and implementation-planning documents.
  - `docs/project-template.md`: Required structure for every new project document.
  - `docs/project-<id>.md`: Per-project contract document (created before implementation begins).
  - `docs/README.md`: Explains the role of internal project-contract docs and how they differ from user-facing docs.
- `skills/`: Skill bundles distributed to end-user AI tools.
  - `skills/design-farmer/SKILL.md`: Router — frontmatter, voice, phase index, cross-phase contracts.
  - `skills/design-farmer/phases/`: Phase instruction files (`phase-*.md`, `operational-notes.md`).
  - `skills/design-farmer/docs/`: Companion docs (`PHASE-INDEX.md`, `QUALITY-GATES.md`, `MAINTENANCE.md`, `EXAMPLES-GALLERY.md`).
  - `skills/design-farmer/examples/`: Reference examples (`DESIGN.md` — Nova UI greenfield reference).
  - `skills/design-farmer/bin/`: Executable utilities (`version-check`).
  - `skills/design-farmer/tests/`: Test suites (`run-all.sh`, `test-semantic-consistency.sh`, `test-version-check.sh`).
- `scripts/`: Repository-level validation and CI scripts.
  - `scripts/validate-skill-md.sh`: Structural validation (phase files, router references, contracts).
  - `scripts/release.sh`: Atomic release automation (version bump, file sync, tag creation).
  - `scripts/release-sync-manifests.mjs`: Schema-aware synchronization of `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` from `package.json`; importable as a library and invokable as a CLI from `release.sh`.
  - `scripts/__tests__/release-sync-manifests.test.mjs`: `node --test` unit tests for the manifest sync module (baseline, unknown-field preservation, schema-forbidden stripping).
  - `scripts/test-install-smoke.sh`: Install/uninstall smoke tests across tools and shells.
- `.github/`: GitHub configuration.
  - `.github/workflows/skill-quality.yml`: CI pipeline (structural validation, plugin manifest validation, install/uninstall smoke tests).
  - `.github/pull_request_template.md`: PR template with validation evidence checklist.
  - `.github/dependabot.yml`: Dependabot configuration that tracks GitHub Actions pins on a weekly schedule.
- `.claude-plugin/`: Claude Code Marketplace plugin metadata.
  - `.claude-plugin/plugin.json`: Marketplace plugin manifest (name, version, skills path).
  - `.claude-plugin/marketplace.json`: Marketplace listing metadata (owner, plugins array, tags, category).
- `package.json`: Single source of truth for version and release metadata (`private: true`, no npm publish).
- `INSTALLATION.md`: Canonical install lifecycle guide, including Marketplace UI and CLI flows, curl installer, manual setup, troubleshooting, and optional removal.
- `install.sh`: Automated installer (detects tools, supports selective target flags, downloads skill bundle atomically).
- `uninstall.sh`: Automated uninstaller (detects/selects tools and removes only `skills/design-farmer` targets safely).
- `docs/marketplace-release-procedure.md`: Step-by-step marketplace release workflow.
- `AGENTS.md`: This file — repository-wide rules.
- `CONTRIBUTING.md`: Contributor workflow (branch naming, commit convention, PR requirements).
- `README.md`: Project overview, installation, and documentation links.
- `README.*.md`: Localized overview and installation entrypoint documents.

### Documentation Policy

- New feature or subsystem creation requires a `docs/project-<id>.md` before implementation begins.
- Every structural change to file paths or phase boundaries must update the corresponding `docs/` file in the same change.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ohprettyhak/design-farmer](https://github.com/ohprettyhak/design-farmer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
