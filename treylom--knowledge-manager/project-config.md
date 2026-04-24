---
trigger: always_on
description: > This file is the vendor-neutral version of `CLAUDE.md`. If you are an AI coding assistant other than Claude Code (e.g. Codex, Cursor, Gemini, Antigravity, OpenCode, an autonomous agent framework, or a future model), read this file **before** you modify anything in this repository.
---

# AGENTS.md — Generic AI agent contract

> This file is the vendor-neutral version of `CLAUDE.md`. If you are an AI coding assistant other than Claude Code (e.g. Codex, Cursor, Gemini, Antigravity, OpenCode, an autonomous agent framework, or a future model), read this file **before** you modify anything in this repository.

---

## Repository Identity

- **Name**: knowledge-manager
- **Type**: Claude Code / Antigravity skill + agent + command distribution repo
- **Primary purpose**: Ingest content (web, PDF, YouTube, social, chat logs) and file it as structured notes in the user's Obsidian vault (or Notion, or local markdown).
- **Deployment model**: Users `git clone` this repo, run a setup wizard, and the wizard rewrites placeholder tokens in every skill file with the user's personal paths.
- **Key invariant**: The committed source tree is **portable**. It contains `{{PLACEHOLDER}}` tokens, never one user's hardcoded paths.

---

## Read First

Read these files **in this order** before making any edit:

1. [`CLAUDE.md`](CLAUDE.md) — the same contract written for Claude Code, with a worked example of source-vs-substituted file forms.
2. [`README.md`](README.md) — end-user documentation; see the `## Installation & Configuration` section for how the placeholder system is presented to users.
3. [`docs/vault-path-configuration.md`](docs/vault-path-configuration.md) — full technical reference: config schema, substitution algorithm, skip-worktree mechanism, troubleshooting.

If you are unable to read those files for any reason, stop and ask the user to paste their contents rather than guessing.

---

## Placeholder Contract (CRITICAL)

The following 5 tokens appear verbatim in committed source files and **must be preserved** whenever you edit a skill, agent, or command file:

| Token | Semantic meaning |
|---|---|
| `{{VAULT_PATH}}` | Absolute path to the user's Obsidian vault directory |
| `{{VAULT_NAME}}` | Basename of `{{VAULT_PATH}}` (the vault folder name, used by Obsidian CLI `--vault` argument) |
| `{{OBSIDIAN_CLI}}` | Absolute path to the Obsidian CLI executable (may be empty string if not installed) |
| `{{ZETTELKASTEN_ROOT}}` | Vault-relative path to the Zettelkasten root folder (default: `Zettelkasten`) |
| `{{RESEARCH_ROOT}}` | Vault-relative path to the research/MOC root folder (default: `Research`) |

### Do

- Write placeholders in their exact `{{DOUBLE_BRACE_UPPERCASE}}` form.
- Compose placeholders for nested paths: `{{VAULT_PATH}}/{{ZETTELKASTEN_ROOT}}/AI/note.md`.
- Use `{{VAULT_NAME}}` when the Obsidian CLI needs a vault name rather than a path.
- Leave the one intentional exception in `skills/km-storage-abstraction.md` alone (it teaches users not to prefix paths with the vault name, using the generic string `YourVaultName`).

### Do not

- Do **not** hardcode any personal absolute path (any path containing a specific username, user home, or real vault name). Forbidden patterns include but are not limited to:
  - `/home/<anyname>/...`
  - `/Users/<anyname>/...`
  - `C:\Users\<anyname>\...`
  - `/mnt/c/Users/<anyname>/...`
- Do **not** rename, shorten, or invent new placeholder tokens. The set of 5 is fixed; adding new ones requires a design doc change and coordinated edits in `scripts/configure-vault-paths.sh`.
- Do **not** remove the `{{...}}` braces "because the local file already has the substituted value." That substituted value is a skip-worktree artifact; the committed HEAD still has the placeholder.
- Do **not** commit `km-config.json` (the user's personal config). It is gitignored.

---

## Editing Workflow

For any change that touches `skills/`, `agents/`, `commands/`, or `.claude/`:

1. **Identify the target file(s).** If the file has a mirror under `.claude/`, both must be edited together (see File Synchronization Invariant below).

2. **Read the committed form, not the working-tree form**, if you suspect skip-worktree may be active:
   ```bash
   git ls-files -v <file> | head -1   # leading 'S' = skip-worktree on
   git show HEAD:<file>               # pristine committed content
   ```

3. **Make your edit in placeholder form.** If you need to reference the vault root, write `{{VAULT_PATH}}`. If you need a file under Zettelkasten, write `{{VAULT_PATH}}/{{ZETTELKASTEN_ROOT}}/...`.

4. **Mirror the change** to the `.claude/` twin immediately, in the same commit.

5. **Run the verification commands** in the next section before `git commit`.

6. **Write a commit message** that explains the intent; if you replaced a hardcoded path, call out which placeholder you used.

---

## File Synchronization Invariant

Every file under `skills/`, `agents/`, or `commands/` has a byte-identical mirror under the matching `.claude/` path:

```
skills/km-workflow.md              ≡  .claude/skills/km-workflow.md
agents/knowledge-manager.md        ≡  .claude/agents/knowledge-manager.md
commands/knowledge-manager-at.md   ≡  .claude/commands/knowledge-manager-at.md
```

Check invariant:

```bash
for f in skills/*.md agents/*.md commands/*.md; do

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [treylom/knowledge-manager](https://github.com/treylom/knowledge-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
