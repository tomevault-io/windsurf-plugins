---
trigger: always_on
description: This guide explains how to create, extend, and maintain agent skills for Catalyst by Zoho. Read this before submitting a PR.
---

# Contributing to catalyst-skills

This guide explains how to create, extend, and maintain agent skills for Catalyst by Zoho. Read this before submitting a PR.

---

## Repository layout

```
skills/
  SKILL.md                          ← routing layer (do not add technical content here)
  {catalyst-service}/
    SKILL.md                        ← thin entry point: frontmatter + workflow + triggers + references table
    references/
      {topic}.md                    ← detailed technical content loaded on demand
```

The routing `catalyst-by-zoho/SKILL.md` routes queries to service-level skills. Service skills route to reference files. **Never inline detailed technical content in a SKILL.md** — it gets loaded on every match and bloats the context window.

---

## How Skills Load (3-Tier Model)

Every agent interaction with skills goes through three loading stages:

| Tier | What loads | When |
|------|-----------|------|
| **1 — Metadata** | YAML frontmatter only (`name`, `description`, `metadata`) | Always — for every skill in the registry, on every request |
| **2 — Body** | Full SKILL.md markdown (How It Works, Triggers, Security Checklist, References table) | Only when the skill's `description` matches the user query |
| **3 — References** | Individual `references/*.md` files | Only when a step in How It Works explicitly loads one |

**Critical implication:** The `description` field is the ONLY content available at routing time. Every product name, symptom phrase, error term, and common entry point that should trigger this skill MUST appear in the description. The `## Triggers` section in the body is the fallback for agents that read the full SKILL.md, but it does not help with initial routing.

**Do not put trigger context only in the body.** If a user asks about `busboy` and `busboy` is not in the description, the skill won't trigger — even if it's in `## Triggers`.

---

## SKILL.md format

Every `skills/{service}/SKILL.md` must follow this structure exactly:

```markdown
---
name: catalyst-{service}
description: "One or two sentences. Include quoted trigger phrases like 'deploy my app', 'create a table', 'upload a file'."
metadata:
  version: "2.0.0"
---

## How It Works

1. Numbered step — what the agent checks or decides first.
2. Numbered step — which reference file to load and why.
3. Numbered step — key rule or gotcha to apply.
(3–5 steps total)

## Triggers

Use this skill for: "trigger phrase one", "trigger phrase two", `code-term`, or "another phrase".

## References

| Reference | Load when the query is about… |
|-----------|-------------------------------|
| `references/{file}.md` | What this file covers |
```

### Rules

- **`description`** — 1–3 sentences. Must contain the service name AND maximize coverage: every major product name, common symptom phrase, SDK method name, and error term that should trigger this skill. Agents use this field for routing — it is the only content loaded at selection time. Aim for 250–500 chars. Do NOT rely on `## Triggers` in the body for routing coverage.
- **`metadata.version`** — semver. Bump the patch version (`2.0.x`) for content fixes, minor version (`2.x.0`) for new reference files, major version (`x.0.0`) for breaking structural changes.
- **`## How It Works`** — 3–5 numbered steps. Describes the agent's decision flow, not the service documentation. Focus on: what to check first, which file to load, what gotcha to apply.
- **`## Triggers`** — exhaustive list of phrases that should route here. Use backticks for code terms, quotes for natural-language phrases.
- **`## References`** — table only. One row per reference file. Load files lazily — only when the relevant step in How It Works is reached.
- **500-line limit** — keep `SKILL.md` under 500 lines total. If you're going over, move content to a reference file.

---

## Reference file format

Reference files in `references/` contain the actual technical content. No YAML frontmatter. Use plain Markdown.

### Required sections (in order)

```markdown
# {Topic Title}

Brief one-line intro.

## {Main Content Section}

...SDK examples, API signatures, config options...

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `ErrorName` | Why it happens | How to fix it |
```

- **`## Common Errors`** — every reference file must end with this section. Use a table for structured error/fix pairs. This is the standardized heading — do not use "Troubleshooting", "Gotchas", or "Known Issues".
- Code examples must be complete and runnable — no `// TODO` or `...` placeholders.
- Include both Node.js and Python examples where the SDK supports both platforms.

---

## Naming conventions

| Thing | Convention | Example |
|-------|-----------|---------|
| Skill directory | `catalyst-{service}` | `catalyst-datastore` |
| SKILL.md `name` field | matches directory | `catalyst-datastore` |
| Reference files | `{topic}-basics.md` or `{topic}-advanced.md` | `datastore-basics.md` |
| New service additions | lowercase, hyphenated | `catalyst-circuits` |

---

## Adding a new service skill

1. Create `skills/catalyst-{service}/SKILL.md` following the format above.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [catalystbyzoho/agent-skills](https://github.com/catalystbyzoho/agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
