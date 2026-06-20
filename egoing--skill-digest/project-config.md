---
trigger: always_on
description: |
---


# Skill Digest

Analyze any agent skill and present it in a standardized digest format.

## When to Use

- User installs a new skill and wants to understand it quickly
- User wants to understand how a skill works before using it
- User asks to compare, quiz, or practice with a skill

**When NOT to use:**
- User already knows the skill and just wants to run it
- Quality evaluation (use rate-skill)
- Security auditing (use skill-auditor)

---

## Input Resolution

Resolve the argument to a SKILL.md before analysis. Support all of the following:

| Input form | Example | Resolution |
|------------|---------|------------|
| Local skill name | `is`, `skill-digest` | Look up `{skills-dir}/{name}/SKILL.md` where `{skills-dir}` is the host's skills directory (e.g. `~/.claude/skills/` on Claude Code) |
| Local path | `<skills-dir>/is/SKILL.md` | Read directly |
| GitHub full URL | `https://github.com/user/repo/blob/main/skills/foo/SKILL.md` | Convert to raw URL and fetch |
| GitHub short form | `user/repo` or `user/repo/path/to/skill` | Fetch from `https://raw.githubusercontent.com/user/repo/main/{path}/SKILL.md` |
| GitHub org/collection | `vercel-labs/skills` | Fetch index, find all SKILL.md files |

**GitHub URL conversion rule:**
- `https://github.com/A/B/blob/BRANCH/PATH` → `https://raw.githubusercontent.com/A/B/BRANCH/PATH`
- Short form `user/repo` with no path → treat as collection (see Multi-Skill Resolution below)
- Short form `user/repo/path` → treat as single skill at that path

**Resolution priority:** local file → local skills dir → GitHub fetch

---

## Multi-Skill Resolution

If the resolved target contains **more than one SKILL.md** (e.g., a GitHub repo or directory with multiple skills):

1. List all discovered skills with their one-line descriptions
2. Immediately ask:

```
Found N skills in [source]:
  1. skill-foo — [one-line from frontmatter description]
  2. skill-bar — [one-line from frontmatter description]
  3. skill-baz — [one-line from frontmatter description]

Analyze:
[A] All — collection overview digest
[B] Pick one — enter number
```

- If user picks **[A]**: output Collection Digest format (see below)
- If user picks **[B]**: output Single Skill Digest format for that skill

---

## Localization

Detect the user's language from the conversation. If not English:

- **Keep the `◆` prefix** on every `##` section — it is a language-independent visual marker, never translate or drop it.
- **Translate the heading text only** (e.g., `## ◆ Steps` → `## ◆ 단계`, `## ◆ Watch Out` → `## ◆ 주의사항`).
- **Translate the `Key:` sentence** into the user's language while keeping the `**Key:**` label (or its localized equivalent) bold.
- **Write all body text** in the user's language.
- **Keep code blocks, skill names, and CLI examples** in their original form — do not translate identifiers, command names, or flag names.
- **Keep the compact menu labels** (`[1] Quiz`, `[2] Deep Dive`, etc.) in the same language as the rest of the output.

**Section heading translation reference (Korean):**

| English | Korean |
|---------|--------|
| Key | 핵심 |
| When | 언제 쓰나요 |
| Steps | 단계 |
| Watch Out | 주의사항 |
| Tips | 알아두면 좋은 것 |
| Try It | 사용 예시 |
| Related | 관련 스킬 |
| Skills at a Glance | 스킬 한눈에 보기 |
| Common Paths | 자주 쓰는 조합 |
| Quick start | 빠른 시작 |
| Learn more | 더 알아보기 |
| More details | 세부 정보 |
| What next? | 다음 단계? |

For other languages, derive equivalent translations. Do not use the English headings when the user's language is detected as non-English.

## README Integration

Before outputting the digest, check if a `README.md` exists alongside `SKILL.md` (same directory):

- If found: read it and **merge any additional context** not covered in SKILL.md — e.g., installation steps, known limitations, changelog, or extended examples
- Surface README-only information in the appropriate digest sections (extra TIPs, extra WARNs, or a new `## Setup` section if installation steps are present)
- If README contradicts SKILL.md, prefer SKILL.md for behavioral rules; prefer README for setup/environment facts

---

## How It Works

1. **Resolve input** — determine source (local / GitHub), fetch SKILL.md
2. **Check for README** — read README.md from same directory if present
3. **Check for multiple skills** — if collection, ask All vs Pick (see Multi-Skill Resolution)
4. **Read skill files** — SKILL.md and all auxiliary files (reference/, rules/, scripts/, examples/)
5. **Analyze** — in this exact order, do not skip:
   a. Distill the single most load-bearing sentence about the skill → this becomes the `Key:` line
   b. Classify type (see Classifying Type)
   c. Classify complexity (see Classifying Complexity)
   d. Identify 1–3 `When` rescue scenarios (pain → what the skill does)
   e. Extract primary workflow as numbered sequence (3–7 steps, no sub-bullets)
   f. Identify warnings (⚠) — **only those explicitly documented** in SKILL.md / README. Search for phrases like "주의", "warning", "do not", "must not", "절대로 ~ 금지", "irreversible", "will silently". Quote or paraphrase the exact documented warning. **Do not infer warnings from the skill's architecture** — if docs don't warn, assume the skill already handles it. Fewer true warnings > more invented ones.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [egoing/skill-digest](https://github.com/egoing/skill-digest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
