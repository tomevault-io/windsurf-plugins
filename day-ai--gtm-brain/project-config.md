---
trigger: always_on
description: This repo is a GTM operating system. A human Owner/Admin and a set of agents work from shared planning documents and execute against a live **Day AI** workspace through the Day AI MCP server.
---

# GTM Brain — Operating Guide

This repo is a GTM operating system. A human Owner/Admin and a set of agents work from shared planning documents and execute against a live **Day AI** workspace through the Day AI MCP server.

There are two layers, and they have a strict relationship:

- **The planning layer** (`planning/`, `workspace/PEOPLE.md`) is the source of truth for *what the business is trying to do and who is doing it*.
- **The implementation layer** makes the Day AI workspace *reflect* the plan — the right people, at the right roles, each with an agent configured to do their real work.

Plan first. Implement from the plan. When reality and the plan diverge, the job is to either change the workspace or update the plan — never to let the gap sit silently.

**Initiatives are the unit of work between the two.** An **initiative** (`initiatives/<slug>.md`) is a bounded, owned, time-boxed effort with a verifiable definition of success — it pulls from the plan and drives implementation work until its success criteria actually verify against the workspace. Initiatives sit *above* the fine-grained outcomes in `planning/OUTCOMES.md`: an outcome is atomic ("draft a follow-up after a call"); an initiative is the larger effort an outcome serves ("get the team running on Day AI by Q3"), realized through many outcomes, invites, agents, and skills. **`/start`** is the entrypoint that takes stock of every initiative and kicks off what each one needs. See [`initiatives/README.md`](initiatives/README.md) for the file format, status lifecycle (`NEW → IN_PROGRESS → SUCCEEDED`, plus `PAUSED`/`CANCELLED`), and the two initiatives that ship with the repo: `map-your-gtm` (leads pre-signup) and `bootstrap-day-ai` (leads once a workspace connects).

---

## The three operator states

Every run starts by knowing which of three states the operator is in. `/start` detects it (via `manage_workspace_members` → `list_configuration`) and every other skill inherits the answer.

1. **Connected Owner/Admin.** The full harness. All workspace reads and writes go through the Day AI MCP; identity is implicit in the OAuth token — you never pass `workspaceId`, `userId`, or the current `assistantId`. You only pass `targetAssistantId` when acting on a *different* agent than your own. Cross-agent and member-management actions require the Admin or Owner role.
2. **Connected Member.** The planning side works; inviting people, editing teammates' agents, and creating skills for others are blocked. If a tool returns *"requires Admin or Owner,"* stop and tell the operator — don't design around it.
3. **Prospect (no workspace yet).** The pre-signup mode. The harness maps the GTM through `/discover` and the `map-your-gtm` initiative, and builds the deployable payload in `rollouts/preflight/`. Skills that need the workspace degrade the way `/sync-pages` does: state plainly what's unavailable and why, do the repo-side work, never pretend a write happened. Verification uses the pre-signup vocabulary — confirm from recorded decisions with named owners, not from a doc existing.

In every state, the harness knows **who is operating it**: `/start` reads git config as a hint (and `gh auth status` when available), **confirms the identity with the operator**, and records the confirmed operator in `workspace/PEOPLE.md` with a trust note. Git identity is never trusted unconfirmed — a prospect's machine routinely carries someone else's. The harness acts with that person's hands.

---

## The Day AI MCP tools

This harness is built on the workspace-management tools plus the read-only graph tools the Day AI MCP exposes.

### Management tools (the core of this repo)

| Tool | Purpose | Key modes |
|------|---------|-----------|
| `mcp__day-ai__assistant_settings` | Inspect & edit agents — your own, or (Admin/Owner) any agent in the workspace | `read`, `update`, `list` |
| `mcp__day-ai__manage_skills` | Full skill lifecycle on an agent or in the workspace library, reading a skill's run history, and pushing library skills to teammates' agents | `list`, `get`, `create`, `update`, `delete`, `reset_prompt`, `get_history`, `deploy`/`undeploy` (Admin/Owner) |
| `mcp__day-ai__manage_workspace_members` | Members, roles, invites, domain auto-invite, suggested invites | `list_configuration`, `invite_member`, `resend_invite`, `revoke_invite`, `update_invite_role`, `enable_auto_invite`, `disable_auto_invite`, `list_suggested_invites`, `navigate_to_billing` |
| `mcp__day-ai__manage_workspace_instructions` | The single workspace-wide instruction every agent inherits — the home for rules that apply across the board | `list_configuration`, `update` |

**Always start a member/invite task with `list_configuration`** — it returns the current members, roles, claimed domains, auto-invite config, and *what the caller is allowed to do*. Read `currentUser` before acting.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [day-ai/gtm-brain](https://github.com/day-ai/gtm-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
