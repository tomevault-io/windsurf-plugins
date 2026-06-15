---
trigger: always_on
description: **WORKFLOW SKILL** — Iteratively improve skill frontmatter compliance using the Ralph loop pattern. WHEN: \"run sensei\", \"sensei help\", \"improve skill\", \"fix frontmatter\", \"skill compliance\", \"frontmatter audit\", \"score skill\", \"check skill tokens\". INVOKES: token counting tools, test runners, git commands. FOR SINGLE OPERATIONS: use token CLI directly for counts/checks.
---


# Sensei

> "A true master teaches not by telling, but by refining."

Automates skill frontmatter improvement using the [Ralph loop pattern](references/loop.md) - iteratively improving skills until they reach Medium-High compliance with passing tests.

## Help

When user says "sensei help" or asks how to use sensei:

```
╔══════════════════════════════════════════════════════════════════╗
║  SENSEI - Skill Frontmatter Compliance Improver                  ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  USAGE:                                                          ║
║    Run sensei on <skill-name>              # Single skill        ║
║    Run sensei on <skill-name> --fast       # Skip tests          ║
║    Run sensei on <skill1>, <skill2>        # Multiple skills     ║
║    Run sensei on all Low-adherence skills  # Batch by score      ║
║    Run sensei on all skills                # All skills          ║
║                                                                  ║
║  WHAT IT DOES:                                                   ║
║    1. READ    - Load skill's SKILL.md and count tokens           ║
║    2. SCORE   - Check compliance (Low/Medium/Medium-High/High)   ║
║    3. SCAFFOLD- Create tests from template if missing            ║
║    4. IMPROVE - Add WHEN: triggers (cross-model optimized)       ║
║    5. TEST    - Run tests, fix if needed                         ║
║    6. TOKENS  - Check token budget                               ║
║    7. SUMMARY - Show before/after comparison                     ║
║    8. PROMPT  - Ask: Commit, Create Issue, or Skip?              ║
║    9. REPEAT  - Until Medium-High score achieved                 ║
║                                                                  ║
║  TARGET SCORE: Medium-High                                       ║
║    ✓ Description > 150 chars, ≤ 60 words                         ║
║    ✓ Has "WHEN:" trigger phrases (preferred)                     ║
║    ✓ No "DO NOT USE FOR:" (risky in multi-skill envs)             ║
║    ✓ Has "INVOKES:" for tool relationships (optional)            ║
║    ✓ SKILL.md < 500 tokens (soft limit)                          ║
║                                                                  ║
║  MCP INTEGRATION (when INVOKES present):                         ║
║    ✓ Has "MCP Tools Used" table                                  ║
║    ✓ Has Prerequisites section                                   ║
║    ✓ Has CLI fallback pattern                                    ║
║    ✓ No skill-tool name collision                                ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

## Configuration

Sensei uses these defaults (override by specifying in your prompt):

| Setting | Default | Description |
|---------|---------|-------------|
| Skills directory | `skills/` or `.github/skills/` | Where SKILL.md files live |
| Tests directory | `tests/` | Where test files live |
| Token soft limit | 500 | Target for SKILL.md |
| Token hard limit | 5000 | Maximum for SKILL.md |
| Target score | Medium-High | Minimum compliance level |
| Max iterations | 5 | Per-skill loop limit |

Auto-detect skills directory by checking (in order):
1. `skills/` in project root
2. `.github/skills/` 
3. User-specified path

## Invocation Modes

### Single Skill
```
Run sensei on my-skill-name
```

### Multiple Skills
```
Run sensei on skill-a, skill-b, skill-c
```

### By Adherence Level
```
Run sensei on all Low-adherence skills
Run sensei on all Medium-adherence skills
```

### All Skills
```
Run sensei on all skills
```

### Fast Mode (Skip Tests)
```
Run sensei on my-skill --fast
```

### GEPA Mode (Deep Optimization)
```
Run sensei on my-skill --gepa
Run sensei on my-skill --gepa --fast
Run sensei on all skills --gepa
```

When `--gepa` is used, Step 5 (IMPROVE) is replaced with GEPA evolutionary optimization.
Instead of template-based improvements, GEPA uses the existing test harness as a fitness
function and an LLM to propose and evaluate many candidate improvements automatically.

**GEPA score-only mode** (no LLM calls, just evaluate current quality):
```
Run sensei score my-skill
Run sensei score all skills
```

## The Ralph Loop

For each skill, execute this loop until score >= Medium-High:

### Step 1: READ
Load the skill's current state:
```
{skills-dir}/{skill-name}/SKILL.md
{tests-dir}/{skill-name}/ (if exists)
```

Run token count:
```bash
npm run tokens -- count {skills-dir}/{skill-name}/SKILL.md
```

### Step 2: SCORE
Assess compliance by checking the frontmatter for:
- Description length (>= 150 chars, ≤ 60 words; *length floor waived when target repo's `.sensei.json` or AGENTS.md prefers short trigger phrases*)
- "WHEN:" trigger phrases (preferred) or "USE FOR:"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spboyer/sensei](https://github.com/spboyer/sensei) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
