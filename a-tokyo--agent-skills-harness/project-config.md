---
trigger: always_on
description: Agent Skills Harness is a factory and testing ground for building production-grade agent skills. It provides a structured pipeline for creating skills that are benchmarked against gold standards, autonomously improved via autoresearch loops, and verified through multi-agent consensus.
---

# AGENTS.md

## Project Overview

Agent Skills Harness is a factory and testing ground for building production-grade agent skills. It provides a structured pipeline for creating skills that are benchmarked against gold standards, autonomously improved via autoresearch loops, and verified through multi-agent consensus.

The core output is the `create-skill-autoresearch` factory skill, which orchestrates the entire skill creation lifecycle.

## Repository Structure

```
.agents/skills/                 # Skills the harness USES (factory + vendored companions)
  create-skill-autoresearch/    # The factory skill (main deliverable; original)
  autoresearch/                 # Autonomous experimentation loop (vendored)
  production-grade/             # Engineering posture principles (vendored)
  premortem/                    # Risk analysis before execution (vendored)
  handoff/                      # Context preservation across sessions (vendored)
  documentation-writer/         # Diataxis documentation generation (vendored)
  llm-council/                  # Multi-agent planning with consensus (vendored)
  writing-great-skills/         # Skill-writing craft (vendored)
  tribunal/                     # Phase 5 verification when delegated (vendored)
  design-taste-frontend/        # Anti-slop frontend/UI skill (vendored)

builds/                         # Skills the harness PRODUCES (gitignored; one folder per build)
self-test/                      # The factory's own regression test + worked example
site/                           # Docs + landing site (Nextra → Vercel)
skills-lock.json                # Provenance + version pins for vendored skills

docs/
  reference/io-contract.md      # What goes in / what comes out (start here)
  reference/workspace-layout.md # Full file-by-file build layout
  reference/                    # rubric-format, metric-protocol, ...
  thoughts/                     # Research notes and design decisions
  study/                        # Case study materials (gitignored; maintainer-local)
  resources/                    # External reference implementations (git submodules)
  usage-guide.md                # How to use the factory
  architecture.md               # Design overview
```

## Skills

| Skill | Purpose |
|-------|---------|
| `create-skill-autoresearch` | Factory for creating production-grade skills via autoresearch |
| `autoresearch` | Autonomous iterative experimentation loop with METRIC protocol |
| `production-grade` | Principle-engineering posture for production-grade code |
| `premortem` | Identify failure modes before they occur |
| `handoff` | Compact conversation into handoff document for another agent |
| `documentation-writer` | Diataxis-guided documentation generation |
| `llm-council` | Multi-agent planning with anonymized judging |
| `writing-great-skills` | Skill-writing craft: leading words, information hierarchy, description quality |
| `tribunal` | Doer → verifier-panel → consensus verification; the factory delegates Phase 5 to it when present |
| `design-taste-frontend` | Anti-slop frontend/UI design for landing pages and redesigns |

## Setup

```bash
git clone <repo-url>
cd agent-skills-harness
npm install
git submodule update --init --recursive   # docs/resources/ reference implementations
```

No build step required. Skills are markdown-based and used directly by any AI coding agent that supports the SKILL.md format. The vendored skills are committed, so a fresh clone works offline.

To refresh a vendored skill, scope it: `npx skills update <skill-name>`. `npm run skills:refresh-all` does every one of them from each upstream's current HEAD and rewrites the lock entries — review that diff before committing, since an upstream edit changes how a companion behaves. `skills-lock.json` records provenance and a content hash; it is not a pin the CLI restores you to.

`npm test` runs the regression gate (`self-test/evaluation/gate.sh`): the structural checks, failing on any failure not on its advisory list — which is currently empty on purpose, so the suite is expected to be fully green. It needs no credentials and no network.

## Development Conventions

- Skills live in `.agents/skills/<skill-name>/SKILL.md`
- Skills follow the official skill-authoring rules (Anthropic best-practices; YAML frontmatter, < 500 lines, progressive disclosure, references one level deep) — see `.agents/skills/create-skill-autoresearch/references/skill-authoring-best-practices.md`
- A factory run produces `builds/<skill-name>/` with three zones: `input/` (human materials), `work/` (generated artifacts), `output/<skill-name>/` (the finished skill). `builds/` is gitignored.
- `create-skill-autoresearch` is developed here (source of truth) and published as a byte-identical release to `a-tokyo/agent-skills` — never edit the published copy directly. See "Releasing create-skill-autoresearch" below.
- Research notes go in `docs/thoughts/` with numbered prefixes (00-, 01-, etc.)
- Design decisions are logged in `docs/thoughts/07-design-questions.md`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [a-tokyo/agent-skills-harness](https://github.com/a-tokyo/agent-skills-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
