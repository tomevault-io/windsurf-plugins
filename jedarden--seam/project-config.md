---
trigger: always_on
description: Repo-level rules. These sit closer to the work than `~/CLAUDE.md`, so they win
---

# SEAM — agent rules

Repo-level rules. These sit closer to the work than `~/CLAUDE.md`, so they win
where they differ. Nothing here relaxes the hard prohibitions there.

## Secrets travel by reference, never by value

**This is the rule this repo exists to embody.** SEAM's whole purpose is that a
calling agent passes a *route reference* and never sees the credential — the
mediator injects it server-side from OpenBao. Agents working *on* SEAM are held
to the same standard the product enforces.

So: **never write a credential value.** Not into a file, commit, bead, doc, log,
or chat. Write the retrieval path instead:

- an OpenBao path — `secret/<cluster>/<app>/<key>`
- or the command that fetches it — `gh auth token`, `git credential fill`

If you need to demonstrate that a credential works, **record the result of the
check, not the credential**: "verified: has `repo` scope, ADMIN on
declarative-config" is the deliverable. The token is not.

If a bead asks you to *provision* a credential, its deliverable is **the path
where the value now lives**, plus whatever policy grants access to it. A bead
that does not name a storage target is under-specified — say so and ask, rather
than inventing a home for the value.

### Why this is stated so bluntly

On 2026-08-09 a worker on `bf-2hwgv` ("Provision GitHub token with
declarative-config PR capability") did the verification correctly and then
pasted the live `gho_` GitHub OAuth token into
`docs/notes/github-token-declarative-config-pr.md` — twice, under "Current
Authentication Status" and "Option A". Nothing but Forgejo's pre-receive hook
stood between that and a public GitHub mirror. The token had to be rotated.

The failure was not carelessness about secrets in the abstract; the note was
otherwise careful and accurate. It was that "document the token" and "document
how to obtain the token" were treated as the same instruction. They are not.

A `PreToolUse` hook (`~/.claude/hooks/org-rule-guard.py`) now denies writes
containing a high-signal credential value in any file type. It **fails open** by
design — a missed violation is recoverable, a wedged fleet is not — so the rule
binds you whether or not the hook catches it. Genuine test fixtures may carry a
`gitleaks:allow` comment on the line; obvious placeholders (all-one-character
bodies, `example`, `REPLACE`) pass unblocked.

## Where things live

The SEAM **binary** lives here. Its **configuration** — route fragments, OpenBao
policies, Kubernetes manifests — belongs in `jedarden/declarative-config` under
`k8s/rs-manager/{seam,seam-retirement-evaluator}/`. That split is load-bearing:
fragments and the evaluator's own manifests are GitOps-managed there, so every
change to them is reviewable and revertible as an ordinary commit to `main`.

The retirement evaluator (`tools/seam-retirement-evaluator/`) is
**detection-only** (since 2026-09-05, bead `seam-d1120e75`): it holds no
git-host credential, opens no PR, and cannot write anywhere. Its entire output
is one structured log record and one Prometheus counter per deprecation
candidate, carrying the fragment-shaped `x-seam-deprecated` block it proposes.

**Handoff for a detected retirement:** the finding is a proposal, not a
change. A human lands it as an ordinary commit to `declarative-config` `main`
— adding the proposed `x-seam-deprecated` block to the route fragment named in
the record — and reverts the same way if a caller appears. SEAM hot-reloads
the fragment, so no deployment and no review gate are involved; reversibility
is the gate. The step-by-step procedure (reading the record, the pre-land
lint gate, fragment-root placement, hot-reload observation, revert) is
[docs/retirement-handoff-runbook.md](docs/retirement-handoff-runbook.md). Do
not re-add a write path (git-host client, forge token, PR opener, git exec)
to the evaluator: its module's write-contract test fails the build if one
appears.

`declarative-config/infra/` in this repo is a retirement pointer only. Do not
restore manifests there. New infrastructure configuration goes directly to the
authoritative `declarative-config` repository paths named by that pointer.

The same pointer rule covers CI. The live **seam-ci WorkflowTemplate** lives
only in `jedarden/declarative-config` at
`k8s/iad-ci/argo-workflows/seam-ci-workflowtemplate.yml` — edit CI behavior
there and let the `argo-workflows-ns-iad-ci` ArgoCD app sync it. The in-repo
file `declarative-config/k8s/iad-ci/argo-workflows/seam-ci.yaml` is a pointer
only (bead `seam-5515cac0`, 2026-09-25): it held a stale early snapshot that
ArgoCD never applied, and editing a copy like that changes no CI behavior
while looking like it does — the failure class behind the 2026-09 red-gate
incidents. If a doc or a grep names the in-repo path, follow its pointer to
the authoritative file before changing anything. The rule is enforced by
`internal/pointerguard`, which fails the build if a manifest reappears under
either path or a document instructs a reader to run or apply files from them;
frozen point-in-time records that quote such instructions carry a
`seam-pointerguard: historical` header marker instead.

## Beads

This workspace migrated from bead-forge to **bead-rs on 2026-08-14**. Use the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jedarden/SEAM](https://github.com/jedarden/SEAM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
