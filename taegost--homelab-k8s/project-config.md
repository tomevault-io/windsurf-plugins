---
trigger: always_on
description: This file instructs Claude Code on how to operate within this repository.
---

# CLAUDE.md — homelab-k8s

This file instructs Claude Code on how to operate within this repository.
It is the authoritative source for conventions, patterns, and standards.

See `~/.claude/CLAUDE.md` for global operator preferences (communication rules,
research discipline, response format). This file contains homelab-k8s-specific
conventions only.

---

## Bypass Rules (NEVER SELF-AUTHORIZE)

The pre-commit and pre-push hooks have bypass env vars for the human operator:

- `HOMELAB_ALLOW_LATEST=1` — allows `:latest` image tags
- `HOMELAB_ALLOW_MAIN=1` — allows direct pushes to main

**Claude must NEVER set these.** If a bypass is needed (e.g., Chainguard only
publishes `:latest`, or a break-fix must land on main), flag it to the user and
let them decide. The bypasses exist for the human operator, not for Claude.

**Claude must NEVER use `git commit --amend`.** It rewrites the commit hash,
causing merge conflicts when the user has local changes or tries to push.
Always create a normal commit instead.

---

## Planning & Review Sessions (MANDATORY CHECKPOINTS)

These rules apply during any planning session (ce-plan, ce-doc-review,
ce-brainstorm, or any multi-step decision walkthrough). They exist because
Claude defaults to optimizing for completion speed over fidelity to user
decisions, and has a track record of silently reclassifying user directives
as "deferred" rather than acting on them.

### Checkpoint 1: Every user directive must land

When the user gives any directive about the document or the work — whether
phrased as "address this," "fix this," "include X," "we need to," "make sure,"
"apply this," or any other imperative — that directive must produce one of
two outcomes before the session ends:

- **Applied:** An edit was made that fulfills the directive.
- **User-deferred:** The user explicitly chose Defer or Skip on a walkthrough
  question that presented the directive.

Silently dropping, reclassifying to a later phase, or converting a directive
to a "noted for next planning phase" bullet is a violation. If the user
wants it deferred, they will say so explicitly.

### Checkpoint 2: Design changes cascade immediately

When a walkthrough decision changes document structure (e.g., redesigning a
component, changing a data flow, altering scope), every downstream section
that now contradicts the new design must be surfaced as an explicit finding
BEFORE advancing to the next walkthrough question. Do not batch broken
cross-references to the completion report. The walkthrough exists so the
user decides at each step — hiding cascading breakage removes that choice.

### Checkpoint 3: Pre-completion directive scan

Before presenting any completion report or terminal question, scan every
user directive from the current session. Verify each one was either applied
(with a document edit) or explicitly deferred/skipped by the user via a
walkthrough question answer. If any directive was not resolved, surface it
and ask before presenting completion.

### Checkpoint 4: Nothing ignored, everything documented

Every concern the user raises, every question they ask, and every directive
they give during a planning session must be reflected somewhere in the final
state: either as a document edit, a Deferred / Open Questions entry, a
residual concern with explicit owner, or an explicit user Skip. If the user
said it, it exists in the output. Silence is not an acceptable response to
a user-raised concern.

---

## Read Before Acting

Before implementing any change, read the relevant documentation first:

- **Known problems, best practices, or patterns** → `docs/solutions/` — documented solutions organized by category with searchable YAML frontmatter (`module`, `tags`, `problem_type`)
- **New app deployment or Postgres migration** → `docs/postgres-runbooks.md`
- **New app deployment with MariaDB database** → `docs/mariadb-runbooks.md`
- **New app deployment with MongoDB database** → `docs/mongodb-runbooks.md`
- **Secrets workflow** → `docs/sealed-secrets.md`
- **Cluster recovery or node loss** → `docs/disaster-recovery.md`
- **DNS or networking issues** → `docs/troubleshooting.md`
- **Storage utilisation or trim jobs** → `docs/storage.md`
- **External service routing** → `apps/traefik/external/README.md`
- **n8n HA migration (S3, queue mode, custom nodes)** → `docs/n8n-ha-migration.md`
- **ArgoCD HA migration** → `docs/argocd-ha-migration.md`

Do not assume context is current. Read the actual files.

---

## Pre-Commit Verification (MANDATORY)

A git pre-commit hook at `.githooks/pre-commit` runs the full validation
suite automatically. It fires on every commit that touches `.yaml` or `.yml`
files. The hook blocks the commit if any check fails.

**One-time setup (per clone):**
```bash
ln -sf ../../.githooks/pre-commit .git/hooks/pre-commit
```

The hook runs these checks (scripts at `.claude/skills/homelab-validate/scripts/`):

| Check | Script | What it catches |
|---|---|---|
| Sync waves | `sync-wave-check.sh` | Missing/misplaced wave annotations |
| YAML validity | `yaml-validity.sh` | Invalid YAML syntax |
| Plaintext secrets | `plaintext-secret-guard.sh` | Accidentally staged `secret-*.yaml` files |
| IngressRoute | `ingressroute-check.sh` | Wrong namespace, missing middleware, cert issues |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Taegost/homelab-k8s](https://github.com/Taegost/homelab-k8s) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
