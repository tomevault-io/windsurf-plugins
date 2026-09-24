---
trigger: always_on
description: This repository is the source of truth for the **Agentic Maintenance Harness** (AMH) and its
---

# AMH meta-repository — maintenance guide

This repository is the source of truth for the **Agentic Maintenance Harness** (AMH) and its
reference instance. Its product is shell and Markdown. Adopted harness version: **AMH 14.1.0**
(`harness/VERSION`).

This file states the harness and this repository as they are now. Rules are rewritten in place
and deleted when they stop binding; nothing here records what a rule used to be. A rule that
still binds stays, whatever its age — relocating a live rule is repeal, not tidying.
Supersession history, adoption narratives, and per-version records of what the owner sanctioned
go to the live ledger volume, with a pointer line in the `docs/STATE.md` changelog; moving
anything out of this file takes the rule-review protocol, and a bulk relocation is the owner's
call. No byte cap governs this file: the defect is kind, not size, and a cap over live
legislation would only invite shaving rules to make room for kept narrative. What stands in is
a reader, not a check — the `RULE_FILES` advisory is a local, WARN-only, uncommitted-diff-only
courtesy that says the protocol applies; the reviewer is the enforcement.

## Sources and generated artifacts

- `harness/src/*.md` is hand-edited harness prose; `harness/dist/AMH.md` is its generated
  bundle.
- `harness/templates/scripts/` contains the shipped, repository-agnostic scripts; matching
  files in `scripts/` are this repository's byte-identical local copies.
- `harness/templates/seed/` contains scaffolds copied once for adopters; `docs/STATE.md`,
  `docs/LEDGER.md`, and the other files in `docs/` are owned by this repository.
- `harness/templates/configs/` contains adopter configuration templates; `amh.conf` records
  this instance's current configuration.

Code and guard fixtures settle what the harness currently does; when descriptive prose
conflicts with them, correct the prose — but never a committed ledger row, which is immutable:
write a new row and append one `Superseded by` or `Corrected by` pointer to the old one. The
` [cited]` marker is the one exception, being metadata rather than content: syncing it in place —
adding it or dropping it — is what the citation rung demands. They do not settle what the harness *should* do:
changing a binding value or rule is legislation, not documentation repair, and must follow
the rule-review protocol.

## Universal session sequence

1. Run `scripts/session-start.sh` if the host has not already run it.
2. Read `docs/STATE.md`, including the Owner queue; verify observable queue claims before
   acting on or repeating them. The file is tree-relative by rule, but a legacy sentence about
   the world — merged, tagged, released, CI, branch protection — may predate that: check such a
   claim against a live source before acting on or repeating it, whichever section it sits in.
3. Select the relevant procedure under `docs/RUNBOOK.md` → **Change-type playbooks**, and
   read everything it names before editing.
4. Work sequentially in a small, shippable unit with binary acceptance.
5. Follow `docs/RUNBOOK.md` → **Acceptance ladder** and review the command's actual output.
6. Update `docs/STATE.md` with what stays true of the checked-out tree; never cache
   world-controlled status (merged, tagged, released, PR/CI, deployments, remote branches, forge
   settings) as current truth — point at the live probe, route it to the Owner queue, or keep it as
   an observation scoped in the sentence to when it was seen. Over the compression trigger, follow
   **Working-memory compression** before editing; that section carries both rules in full. Improve the runbook in the same change if its procedure proved insufficient.
7. Commit with an honest verification disclosure, then push the permitted
   `BRANCH_PREFIX/<codename>` session branch.

The procedures named in `docs/RUNBOOK.md` are binding. Follow **Session discipline** every
session; use **Change-type playbooks** for the task; apply **Rule-review protocol** when the
diff changes binding rules or guard semantics. `RULE_FILES` is a tripwire, not a complete
definition of that scope.

## Hard boundaries

- Never inspect or disclose credential values or private personal identifiers, including
  fragments, lengths, hashes, environment dumps, credential files, or container/service
  inspect output. Automated identity checks may inspect commit metadata but must not render
  unapproved addresses. See `scripts/command-guard.sh` — and read the **what this guard does
  NOT catch** block in its header before treating a green check as safety: interpreters outside
  its enumerated reader list (`python3 -c "open('.env')"` above all), wrappers it does not strip,
  constructed commands and heredocs reach past it. That block is the authority on which is
  which; do not infer coverage in either direction from this line. This rule binds you whether
  or not a script can see the shape you chose. If exposure occurs, follow `docs/RUNBOOK.md` →
  **Incident: leaked credential**.
- Never force-push or push to the branch named by `DEFAULT_BRANCH`. Push only the configured
  session branch; the owner merges.
- Never rewrite pushed history. The only exception is the owner-directed, owner-executed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [faded-penguin021/AMH](https://github.com/faded-penguin021/AMH) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
