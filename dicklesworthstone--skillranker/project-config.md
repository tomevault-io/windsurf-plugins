---
trigger: always_on
description: enables it. `--offline` means zero network calls and conflicts with
---

# AGENTS.md — SkillRanker

Guidelines for AI coding agents working in this repository.

## Rule 0 — Direct Instructions

Follow Jeffrey's direct instructions. These rules encode standing preferences;
they do not overrule the user. Finish authorized work and report concrete results,
with the checks that support them.

## No Deletion Or Destructive Git

Do not delete files or directories without explicit written permission, including
files you created yourself. Do not run `git reset --hard`, `git clean -fd`,
`rm -rf`, force pushes, or equivalent destructive operations without explicit
authorization for the exact operation and its consequences.

Inspect before changing. Preserve work you did not create. Never stash, revert,
overwrite, or blanket-stage another agent's changes. Stage explicit owned paths.
Do not amend published commits.

Work on `main`; create another branch only when the user requests it. Keep the
legacy compatibility branch synchronized when publishing instructions require it.
Public source URLs and documentation use `main`.

## Project Mission And Reading Order

SkillRanker is a standalone Rust CLI, `sr`, that recommends the most useful skills
for the **next step of a specific live agent session**:

**TypeSafe.ai's Jev is the system's essential ranking engine. A TypeSafe API key
is required for the product's ranking workflow.** Keep this dependency prominent
in product descriptions, installation, and onboarding. Context capture, Quill retrieval,
caching, local scoring, and feedback support Jev; they are not a replacement
inference system. Do not imply that `sr` has a key-free ranking backend.

```text
exact session + visible roster + user constraints
  -> bounded, redacted state -> local explicit resolution or Jev ranking
  -> eligibility + abstention -> JSON / table / Claude hook / inline TUI
  -> local observations + independent judgments -> evaluated policy changes
```

Keep this workflow focused. SkillRanker does not execute skills, grant tool
permissions, override user requirements, or stop an agent from working when the
recommendation service fails.

Read these before substantive work:

1. [The comprehensive plan](COMPREHENSIVE_PLAN_TO_DESIGN_SKILLRANKER.md), including
   core invariants, trust policy, dependency choices, and acceptance gates.
2. [README.md](README.md), the product and command contract.
3. Actual source, dependency versions, fixtures, and relevant live task state.

The reviewed plan supersedes its earlier sketches: there is no `ms` bridge,
ignored-suggestion penalty, transparent `ureq` fallback, or HTTP-only deadline.
Resolve contradictions at the affected boundary and update the relevant docs.
Current source and executed checks establish behavior; prose alone is not proof.
Keep README in the finished-product voice Jeffrey requested. Do not add a
design-stage banner or turn installation into a progress report. Track actual
implementation, unsupported commands, phase gates, and revision-bound evidence
in the capability contract, engineering docs, changelog, and live Beads. Product
prose is not a verification receipt.

## Maintainer Credentials — Local Environment And Vault

**Live Jev evaluations require `TYPESAFE_API_KEY`.** The maintainer credential is
available in these locations; retrieve it without printing it:

| Location | Lookup |
|---|---|
| This checkout on `threadripperje` | `/data/projects/skillranker/.env` (also `/dp/skillranker/.env`), variable `TYPESAFE_API_KEY` |
| HashiCorp Vault on `threadripperje` | KV v2 mount `secret`, path `skillranker`, field `TYPESAFE_API_KEY` |
| HashiCorp Vault on `ts1` (`thinkstation1`) | The same mount, path, and field in that machine's local Vault |

The local `.env` is Git-ignored, untracked, and owner-readable/writable only
(`0600`). Export its values before launching `sr` or the agent process whose hooks
need the key; do not assume `.env` is loaded automatically:

```bash
cd /data/projects/skillranker
set +x
set -a
. ./.env
set +a
```

For recovery or use on either maintainer machine, run the following **on that
machine** (connect with `ssh ts1` first for `ts1`). Existing Vault authentication
is required. Each machine has a copy of its local Vault's public CA certificate
at `~/.config/vault/skillranker-ca.pem`:

```bash
set +x
TYPESAFE_API_KEY="$(
  VAULT_ADDR=https://127.0.0.1:8200 \
  VAULT_CACERT="$HOME/.config/vault/skillranker-ca.pem" \
  VAULT_SKIP_VERIFY=false \
  vault kv get -mount=secret -field=TYPESAFE_API_KEY skillranker
)" && export TYPESAFE_API_KEY
```

Keep shell tracing off while handling credentials. Never print the key, paste it
into a tool argument, commit it, or include it in logs, fixtures, or documentation.
Keep `.env.example` credential-free. When rotating the key, update the local
`.env` and both Vault copies, preserving unrelated fields. A loaded credential
does not authorize sending session content: network opt-in still applies.

These are private maintainer stores. Outside users must sign up at the
[TypeSafe console](https://console.typesafe.ai) and obtain **their own API key**;
the README must direct them there.

## Architecture Doctrine

- Start with one Rust package, binary `sr`, and a library exposing pure context

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/skillranker](https://github.com/Dicklesworthstone/skillranker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
