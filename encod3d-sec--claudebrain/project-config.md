---
trigger: always_on
description: | Operation | Action |
---

# Pentesting & Bug Bounty Wiki: Schema

---

## Quick reference

| Operation | Action |
|---|---|
| Query | `qmd_query "..."` via `wiki-search` MCP -> read results -> synthesise |
| Ingest skip check | Read frontmatter only; skip page if ingest slug already in `sources:` |
| Re-index / wiki status | `wiki` skill |
| Git clone | Always WSL: `wsl -d kali-linux -u kali -- git clone <url> /home/kali/<name>` |
| Run tooling against a target | Kali VM over SSH: `bash /root/vm.sh '<cmd>'` (VPN route + tools + chromium live there) -> `docs/virtual-machine.md` |

---

## Skills and tools

| Task                                          | Use                                                                                    |
| --------------------------------------------- | -------------------------------------------------------------------------------------- |
| Multi-step planning                           | `superpowers:brainstorming` then `superpowers:writing-plans`                           |
| Execute a plan                                | `superpowers:subagent-driven-development`                                              |
| Debug unexpected behavior                     | `superpowers:systematic-debugging`                                                     |
| About to claim done                           | `superpowers:verification-before-completion`                                           |
| Write/edit vault `.md`                        | `obsidian:obsidian-markdown`                                                           |
| Fetch URL for ingest                          | `WebFetch` tool                                                                        |
| Read vault file                               | `Read` tool with machine path (see below)                                              |
| Search vault                                  | `qmd_query` (semantic) or `qmd_search` (keyword) via `wiki-search` MCP                 |
| Maintain wiki index (re-index, status)        | `wiki` skill                                                                           |
| Load engagement playbook / FIND schema        | Read `targets/TARGETS.md`                                                              |
| Audit CLAUDE.md (full review)                 | `claude-md-management:claude-md-improver`                                              |
| Update CLAUDE.md (targeted session learnings) | `claude-md-management:revise-claude-md`                                                |
| Session end / pause work                      | `gsd:pause-work` (optional plugin) or the manual pause-work steps                                                                       |
| Parallel independent tasks                    | `superpowers:dispatching-parallel-agents`                                              |
| About to attack a web endpoint                | `hunt-<type>` skill (see auto-triggers below)                                            |
| Driving a web target through Burp (proxy-history triage, Repeater/Intruder/Collaborator) | `hunt-burp` skill (Burp MCP; setup [[burp-mcp]])              |
| Starting recon on any target                  | wiki-recon skill                                                                       |
| Validating / moving finding to Completed      | triage then evidence skills                                                            |
| Vuln/CVE research on a target (binary/repo/app/firmware) | `research` skill (scaffolds `raw/research/<project>/`)                       |

Vault-local skills (read file directly from `skills/`): `code-review/`, `obsidian/`, `wiki/`, `research/` (CVE-discovery loop), `disclosure/` (finding -> CVE). Workflow + hunt skills live under `skills/hunt/`: all `hunt-*` plus `wiki-arsenal` (fast PARALLEL wiki lookup engine over techniques/payloads/tools/cheatsheets; `arsenal` delegates to it), `triage`, `evidence`, `coverage`, `ingest`, `next-move`, `wiki-recon`, `nday`, `research-ingest`, `ctf-box`, `ctf-category`, `screenshot` (visual PoC capture via `scripts/shot.py`/`pocshot.sh`, chromium on Kali -> `targets/<eng>/poc/`), `screenshot-burp` (Burp Repeater request/response PoC via `scripts/burpshot.sh`), and `learn` (post-engagement knowledge harvest: diff a completed engagement against the wiki, promote the delta via the leak-gated stage->promote pipeline). The `claude-md-improver/` local copy is an offline fallback (auto-invocation disabled); prefer the `claude-md-management:claude-md-improver` plugin. For MCP setup, hooks, and plugin troubleshooting: read `skills/skills-setup.md`.

Search rule: never read `wiki/index.md` to find pages - always search first. MCP tool names: `mcp__wiki-search__qmd_query` (semantic), `mcp__wiki-search__qmd_search` (keyword).

`session/memory.md` holds long-term editorial patterns. Load it when making editorial or tagging decisions.

---

## Hunt Skill Auto-Triggers


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Encod3d-Sec/ClaudeBrain](https://github.com/Encod3d-Sec/ClaudeBrain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
