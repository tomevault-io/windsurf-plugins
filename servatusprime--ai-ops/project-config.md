---
trigger: always_on
description: Bootstrap and authority contract for AI agents operating in ai_ops.
---


<!-- markdownlint-disable MD013 MD025 -->

# Agents: Start Here

## Quick Path (Read This First)

**On first turn:** Read this file. This is the canonical bootstrap entry point.
Read `CONTRIBUTING.md` on demand for contributor policy, maintainer workflow,
or unresolved authority ambiguity.

**Mode check:** Determine whether you are operating ai_ops directly (ai_ops repo
is your target) or governing an external repo (ai_ops is your ops stack; the
target is a separate work repo). This is your most important first classification.
See Mode Detection section for the full decision table.

**Do not** scan the filesystem or infer repo purpose before completing bootstrap.

---

## Authority Quick Reference

| Level | Scope | Action |
| ----- | ----------------------------- | ------------------------------------------ |
| 0 | Read, analyze, report | Always allowed |
| 1 | Single atomic edit | Pre-authorized if in scope |
| 2 | 2-5 related changes | Confirm first |
| 3 | 6+ changes or multi-layer | Create workbook, wait for approval |
| 4 | Policies, specs, architecture | Document rationale, require human approval |

Full authority execution contract is consolidated in this document.

**Key principle:** Authority is explicit, not assumed. If uncertain, ask.

**Safe default:** If you are unsure of authority, treat it as Level 3 and ask.

### Authority Detail

Use these details when classifying non-obvious requests:

- **Level 0**: read, analyze, report only (including review reports in active
  workbundle or sandbox).
- **Level 1**: one atomic edit in explicit scope, plus obvious hygiene fixes
  (links, lint, terminology, repo structure map update).
- **Level 2**: 2-5 related edits in one scope (including simple renames and
  linked reference updates); confirm first with the requestor.
- **Level 3**: 6+ edits, multi-layer change, or multi-step resumable lane;
  includes structural reorganization and analysis that yields prioritized
  action lists; create workbook and wait for approval.
- **Level 4**: policy/spec/architecture touching scope; require explicit human
  approval with rationale before execution.

### Path-Based Authority Guard (Pre-Write)

Before any file write, classify the target path against this guard:

| Path Pattern | Minimum Authority | Required Behavior |
| --- | --- | --- |
| `00_Admin/{policies,specs}/**`, `AGENTS.md`, `CONTRIBUTING.md`, `HUMANS.md` | Level 4 | Stop; require explicit human approval. |
| `.ai_ops/workflows/**` | Level 4 | Stop; require explicit approval evidence before edits. |
| `00_Admin/configs/**` | Level 4 | Stop; require explicit human approval. |
| `00_Admin/guides/**` | Level 3 | Use approved workbook scope; do not direct-edit outside scope. |
| `90_Sandbox/**` | Level 1-2 | Allowed when in-scope and request-aligned. |
| `.ai_ops/local/**` | Level 1 | Machine-local work state; update freely when directed by /work or /closeout. Never commit. |

If a path does not match, default to higher governance and ask.

## Agent Decision Tree

```text
START: Work identified

Is this reading/analysis only?
 YES -> Proceed (Level 0)
 NO  ->

Is this single atomic edit in pre-authorized path?
 YES -> Proceed, document (Level 1)
 NO  ->

Is this 2-5 related changes, same scope?
 YES -> Confirm first (Level 2)
 NO  ->

Is this 6+ changes OR multi-layer?
 YES -> Create workbook (Level 3)
 NO  ->

Does this affect policies/specs/architecture?
 YES -> Document rationale, wait (Level 4)
```

## First-Turn Guardrails

- Do not scan the filesystem before reading this file.
- Do not infer repo purpose from directory names.
- Only perform wide scans when explicitly requested.
- If you started scanning before bootstrap, stop, note the violation, and restart.
- Mid-session rebootstrap: re-read this file (Quick Path only).
- Treat questions as questions: answer first. Do not execute edits unless the
  user explicitly asks for action.
- A question about what should change is not authorization to change it.

---

## Red Flags Requiring Workbook

If a response implies multi-phase execution, broad remediation, or workbook-
sized scope, stop and apply this Red Flags gate.

Terminology guardrail: use **work proposal** (not "implementation plan") for Level 4 governance changes.

Common red flags:

- "Here are commands/scripts to run..."
- "I can implement these fixes quickly..."
- "Phase 1 / Phase 2 / Phase 3..."
- "I found N issues, here is a broad fix list..." (especially N > 5)
- "Analysis suggests a prioritized remediation list..."
- Changes span 3+ directories or can fail partway and require resumption
- You expect a status checkpoint question ("what is the status?") before finish

---

## Self-Check Before Acting

Before proposing actions, verify:

- [ ] I identified the authority level (0-4).
- [ ] I followed the decision tree.
- [ ] If Level 3+, I am proposing workbook execution, not direct edits.
- [ ] If Level 4, I documented rationale and am waiting for explicit approval.
- [ ] I am not presenting multi-step implementation as ad-hoc "quick wins".
- [ ] I am not prescribing command sequences for workbook-sized work.
- [ ] If L2+, I emitted a `policy_decision:` PDR (see `00_Admin/specs/spec_policy_decision_record.md`).

## Response Thrift


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [servatusprime/ai_ops](https://github.com/servatusprime/ai_ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
