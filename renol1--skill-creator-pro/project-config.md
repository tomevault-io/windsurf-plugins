---
trigger: always_on
description: Create new skills, modify and improve existing skills, and measure skill quality. ALWAYS use this skill when users want to create a skill from scratch, edit or optimize an existing skill, extract a skill from a conversation, test a skill, or optimize a skill's description for better triggering accuracy. Also trigger when user says "turn this into a skill", "make a skill", "skill from this", "improve my skill", "fix my skill's triggering", "help me build a skill", "teach Claude to do X", "how do 
---


# Skill Creator Pro

Create, test, and iteratively improve agent skills. Works in claude.ai, Claude Code, and Cowork. Cross-platform: installs on Cursor, Codex, Gemini CLI, Windsurf, and 10+ tools.

For a working mini-example, see `references/example-skill.md` (35 lines). For a medium-complexity example with lookup tables and references, see `references/example-medium-skill.md` (120 lines). For design principles, see `references/gold-standard.md`.

## Configuration

Adjust these defaults to match your workflow:
- **default_platform**: claude.ai | claude-code | cowork (default: claude.ai)
- **test_rigor**: vibe | standard | rigorous (default: standard)
- **output_language**: en | sv | de | fr (default: en)

## The Core Loop

1. **Understand** — What should the skill do? When should it trigger?
2. **Draft** — Write the SKILL.md file and bundled resources
3. **Test** — Run realistic prompts, compare output to expectations
4. **Evaluate** — Review results with the user, grade quality
5. **Improve** — Revise based on feedback, retest
6. **Optimize** — Tune the description for reliable triggering
7. **Package** — Deliver the `.skill` file
8. **Deploy** — Install cross-platform, track performance

Jump in wherever the user is. If the user says "just vibe with me", skip the formal eval process — set `test_rigor` to `vibe`.

---

## Routing

| User says | Workflow |
|-----------|----------|
| "Make a skill for X" | **Create** → Step 1 |
| "Turn this into a skill" | **Extract** → Step 1b |
| "My skill doesn't trigger" | **Optimize** → Step 6 |
| "Improve my skill" / shows SKILL.md | **Improve** → Step 5 |
| "Test my skill" | **Test** → Step 3 |

### Platform Detection

Detect your environment and adapt:
- **Claude Code / Cowork**: Subagents available → spawn parallel test runs, use eval viewer, run description optimizer.
- **claude.ai**: No subagents → test conversationally, review inline, skip quantitative benchmarks.

---

## Step 1: Understand the Intent

1. **What should this skill enable Claude to do?** (the capability)
2. **When should it trigger?** (phrases, contexts, file types, keywords)
3. **What's the expected output?** (file format, structure, tone)
4. **What should NOT trigger it?** (adjacent tasks sharing keywords)
5. **Are test cases useful?** Verifiable outputs (file transforms, code gen) → yes. Subjective outputs (writing, art) → usually no. Suggest default, let user decide.

Proactively ask about edge cases, dependencies, and what "good" looks like. Check available MCPs for research.

### Edge Case Discovery

Probe systematically based on skill type:

| Skill type | Edge cases to ask about |
|------------|------------------------|
| **Data transform** (schema→code, CSV→report) | Missing fields, nulls, unexpected types, empty input, huge input, encoding |
| **Document generation** (reports, specs) | No data for a section, conflicting inputs, multilingual, very long/short content |
| **Code generation** (clients, tests, configs) | Reserved words, special characters, circular deps, platform differences |
| **Workflow automation** (deploy, build) | Partial failure, missing credentials, rate limits, idempotency |
| **Content/creative** (writing, design) | Ambiguous tone, conflicting instructions, sensitive topics, brand constraints |

Most skills span multiple types — check all matching rows. Ask 2-3 edge case questions before drafting.

### Step 1b: Extract from Conversation

When the user says "turn this into a skill":

1. Review the conversation — tools used, sequence, corrections made
2. Identify what Claude didn't know that the conversation taught it (= the skill's value)
3. Note corrections (= "Common Mistakes" section)
4. Note input/output formats
5. Confirm understanding before drafting

The extraction question: **"If a fresh Claude session got this same request, what would it get wrong?"** That's what the skill needs to teach.

---

## Step 2: Write the SKILL.md File

### Skill Structure

```
skill-name/
├── SKILL.md          (required — main instructions)
├── references/       (optional — templates, docs, loaded on demand)
├── scripts/          (optional — executable code for deterministic tasks)
└── assets/           (optional — fonts, icons, files used in output)
```

### Progressive Disclosure

| Level | What | When loaded | Size target |
|-------|------|-------------|-------------|
| **Metadata** | name + description | Always in context | ~100 words |
| **SKILL.md file body** | Main instructions | When skill triggers | <500 lines |
| **Bundled resources** | References, scripts, assets | On demand | Unlimited |

### Skill Size Guide

| Complexity | Lines | When |
|------------|-------|------|
| Simple (one task) | 30–60 | Single output, few edge cases. See `references/example-skill.md` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Renol1/skill-creator-pro](https://github.com/Renol1/skill-creator-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
