---
trigger: always_on
description: This file is the **schema**: it tells you (the LLM) how to operate this workspace. Everything else is either a raw source, a wiki page, or a meta file. Read this on every session start. Active workstreams + open decisions live in `ROADMAP.md`, not here.
---

# Cybersecurity Research Wiki — Schema

This file is the **schema**: it tells you (the LLM) how to operate this workspace. Everything else is either a raw source, a wiki page, or a meta file. Read this on every session start. Active workstreams + open decisions live in `ROADMAP.md`, not here.

## Purpose

Local knowledge hub for **cybersecurity research, training, and offensive/defensive operations** — scoped to:

1. **Offensive security** (pentesters, red teamers, bug-bounty hunters) who need ingestable references for engagements, certification prep, and tool selection. Running case: a working pentester building toward CRTO / OSCP / eCPTX while documenting tradecraft per-target-class (web, Windows AD, cloud, mobile, IoT/OT).
2. **Defensive security** (SOC analysts, incident responders, threat hunters, blue teamers) who need detection-engineering references, low-cost tooling stacks, and adversary-emulation playbooks to build labs against.
3. **Career + education** (students, career switchers, parents of kids, recruiters) who need orientation on the cybersecurity job-family map, certification paths, and age-appropriate safety material (cyberbullying, child internet safety, school-attack survival).

The wiki is a librarian that **manages, curates, and applies** that knowledge:

- **Manage** — inventory raw sources (PDFs, slide decks, video transcripts, repo snapshots, blog posts); track what's been read, extracted, and applied
- **Curate** — pull relevant fragments out of raw sources; structure them as interlinked wiki pages on attack techniques, defense controls, tools, frameworks, certifications, and threat actors
- **Apply** — route findings to a real workflow:
  - **claude.ai / Claude Desktop** — context for engagement reports, exam study, threat-model write-ups, training material
  - **Direct hands-on use** — paste a brief into a pentest report, a SOC runbook, a CTF write-up, or a kid-safety conversation

This is a laptop-only workspace. No remote servers, no team distribution. Everything lives on this MacBook.

## Architecture — three layers

1. **Raw sources** — immutable. You read them, never modify them. Live locally in `raw-sources/` (gitignored — PDFs, slide decks, video transcripts, repo snapshots).
   - PDFs (e-books, vendor whitepapers, conference talks — much of the seed corpus is from [Joas A Santos](wiki/entities/people/joas-a-santos.md), a prolific Brazilian cybersecurity educator)
   - Slide decks from conference talks (RoadSec, TDC, Red Team Village, etc.)
   - GitHub repos (cloned snapshots of relevant FOSS tools — Caldera, Metasploit, Cobalt-Strike-detections, Wazuh rule packs, etc.)
   - Video transcripts saved as `.md`
   - **Drop pattern**: drop new sources into `research to be indexed/` (transient drop zone). Ingest pipeline reads + synthesizes, then move to `raw-sources/`.

2. **The wiki** — LLM-written, human-read. Lives in `wiki/`. Structured pages on certifications, tools, frameworks, threat actors, platforms, people, vendors, programming languages, and concepts.

3. **The schema** — this file.

Staging/output lives outside the wiki:
- `briefs/` — one-off deliverables (gitignored): engagement-prep briefings, SOC-runbook drafts, certification cram sheets, kid-safety conversation scripts
- `research to be indexed/` — transient drop zone for new raw sources (gitignored)
- `LESSONS.md` — meta-lessons about *how we work* (distinct from `wiki/log.md`)
- `hot.md` — ephemeral session-state cache (gitignored)
- `ROADMAP.md` — active workstreams + open decisions (tracked)

## Folder layout

```
Cybersecurity-wiki/                  # repo root
  CLAUDE.md                         # this file — the schema
  LESSONS.md                        # meta-lessons (how we work)
  ROADMAP.md                        # active workstreams + decisions + done log
  hot.md                            # session-state cache (gitignored)
  .env.example                      # env-var template (commit this)
  .env                              # actual keys (gitignored — never commit)
  claude_desktop_config.json.example  # Claude Desktop MCP config template (commit this)
  research to be indexed/           # transient drop zone (gitignored)
  raw-sources/                      # archived raw source corpus (gitignored)
  briefs/                           # staging for distribution → claude.ai or hands-on use (gitignored)
  wiki/                             # canonical wiki
    index.md                        # content-oriented catalog of all wiki pages
    log.md                          # append-only chronological operations log
    sources/                        # one page per ingested source
    entities/
      certifications/               # OSCP, CRTO, eCPPT, CEH, CompTIA Security+/PenTest+, etc.
      tools/                        # Cobalt Strike, Metasploit, Burp Suite, Caldera, Maltego, Wazuh, etc.
      frameworks/                   # MITRE ATT&CK, Cyber Kill Chain, OWASP WSTG, Zero Trust, NIST CSF
      threat-actors/                # APT28, APT29, named groups + TTP profiles

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cemini23/Cybersecurity-wiki](https://github.com/cemini23/Cybersecurity-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
