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
| Run a full bb/pt/ctf engagement autonomously  | `bb-workflow` / `pt-workflow` / `ctf-workflow` skill (driver: `scripts/campaign.py`; the single source of truth for the execution loop) |
| Check the workflow driver is set up on this machine | `campaign-health` skill (`scripts/campaign-doctor.py`)                            |
| About to attack a web endpoint                | `hunt-<type>` skill (see auto-triggers below)                                            |
| Driving a web target through Burp (proxy-history triage, Repeater/Intruder/Collaborator) | `hunt-burp` skill (Burp MCP; setup [[burp-mcp]])              |
| Starting recon on any target                  | wiki-recon skill                                                                       |
| Manual login / MFA the agent can't do headlessly (Smart-ID, Mobile-ID, captcha) + drive & observe via CDP | `chrome-devtools-browser` skill (visible chromium on the VM via `scripts/browser-visible.sh` + chrome-devtools MCP) |
| Manual login / MFA the agent can't do headlessly (Smart-ID, Mobile-ID, captcha) + drive & observe via CDP | `chrome-devtools-browser` skill (visible chromium on the VM via `scripts/browser-visible.sh` + chrome-devtools MCP) |
| Validating / moving finding to Completed      | triage then evidence skills                                                            |
| Vuln/CVE research on a target (binary/repo/app/firmware) | `research` skill (scaffolds `raw/research/<project>/`)                       |
| Hand a fiddly, fully-specified exploit-compile/escalation run to a sub-agent | `delegate` skill (autonomous sub-agent exploit-run; false-root/hostname guardrail mandatory) |
| Drive msfconsole (recon, exploit search/run, reverse shells, post-ex) | `metasploit` skill (msfconsole framework-driver; cheatsheet [[metasploit]]) |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Encod3d-Sec/TORCH](https://github.com/Encod3d-Sec/TORCH) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
