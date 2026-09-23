---
trigger: always_on
description: Paperthin is an agent-agnostic suite of plain-Markdown skills that keep an artifact **clean and true** — hygiene reflexes an agent reaches for on its own. This is the guide for authoring them — and itself a skill artifact, so `re0` it when it drifts. Every skill named below is defined in the shipped catalog, the [README](./README.md).
---

# Paperthin

Paperthin is an agent-agnostic suite of plain-Markdown skills that keep an artifact **clean and true** — hygiene reflexes an agent reaches for on its own. This is the guide for authoring them — and itself a skill artifact, so `re0` it when it drifts. Every skill named below is defined in the shipped catalog, the [README](./README.md).

## Philosophy

- **Trust the artifact, not the author.** A skill exists to make work read true to someone who wasn't there — *clear* (does it read?) and *correct* (is what it claims real?). The maker's in-session mind is the worst judge of either, so a skill either re-derives the work from what's true now or imports outside eyes to test it.
- **Generic examples, not instances.** In any durable doc — this guide, a `SKILL.md`, a release note — an example stays generic (`(#123, @handle)`, `land/pr-<n>`); a real PR number, handle, SHA, or range ages into residue a later reader misreads for current fact.
- **Single source of truth, and self-contained skills.** One fact lives in one place; everything else references it by name. But a skill ships and runs installed alone (`npx skills add` can pull one without its siblings or this guide), so it states the runtime rules it needs inline rather than pointing at AGENTS.md or another skill. The two live at different layers: SSOT keeps a *fact* from drifting; self-containment lets a *skill* travel. The one-home rule bends in exactly two places: cross-skill naming, for declared pairs and orchestrators (see [Conventions](#conventions)); and the skill roster, which — with no build step to generate it — is hand-written onto several surfaces that can't reference each other ([re0-upgrade](./skills/breadth/re0-upgrade/SKILL.md)'s catalog, [`scripts/catalog.cjs`](./scripts/catalog.cjs), [plugin.json](./.claude-plugin/plugin.json), the READMEs). CI guards those copies in sync — `validate-skills.sh` and [`check-catalog-sync.cjs`](./scripts/check-catalog-sync.cjs) — rather than one home feeding them; never dedupe them.
- **Restraint.** Change only what genuinely improves — a pass that finds nothing to improve changes nothing. The enemy is slop (noise, duplication, padding), not addition.
- **Recursive.** Every skill is general enough to use *while building the skills*, so we maintain the skills with the skills, and each pass is audited and refined by the tools it ships. Treat that loop as the point.

## Layout

```
skills/<perspective>/<name>/SKILL.md
```

Topic domains belong in separate plugins, so within a plugin the only durable cut is **perspective**. One skill = one directory with a `SKILL.md`.

That cut is two orthogonal axes, **cardinality × time**:

```text
skills/
├── breadth/   reconcile one truth across files and platforms
├── coil/      carry learning between build cycles
├── depth/     refine or verify the thing in hand
└── mesh/      converge independent views into consensus
```

The axes' quadrants and each skill's home are the README's facts — its [map](./README.md#the-map) and [index](./README.md#the-index); this file defines only the cut. Within a perspective the listing runs in a logical order (a `depth/` skill's work-lifecycle, say). One deliberate pin: `re0` leads `depth/` as the founding skill that opened the suite, so `reorder` keeps it first rather than sorting it into the cleanup group.

**File by trigger-scope, not by what a skill invokes.** A skill lives where the work it *triggers or emits* ranges, even when it orchestrates skills from other folders — `sip` gates one finished deliverable, so it's `depth/`, though its check may run cross-file `ssotize`.

Reach for `breadth/` to *establish* order (legacy refactor, knowledge-base build, fresh scaffolding); once a fact is cleanly SSOT'd, *maintain* it with `re0` rather than re-consolidating. Keep drafts and retired skills out of the README and `plugin.json`.

**Name it for the reflex it fires** — a plain real word (`shower`, `sip`) or a tight compression of a real term (`re0`, `ssotize`); a stranger should half-guess what it does from the name alone, so no opaque coinage. A self-evident metaphor-noun is allowed only as a deliberate exception, when its own intuition carries it — `autobahn` is the standing example. The `re0-` prefix is for clean-version lifecycle commands: upgrade an install, release a package, memorize a cycle, restart a build, run the loop, or clean a commit message. Never model-brand a name; the mechanism must outlive any one model.

## SKILL.md format

```
---
name: <kebab-name>            # matches the directory and the invocation
description: "<trigger-rich one-liner>"
# disable-model-invocation: true   ← user-invoked skills only
---

<one line: what the skill does>

## Goal
## Workflow      — numbered steps
## Rules         — constraints
## Verification  — checks before finishing; report what changed
```

## Invocation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LilMGenius/paperthin](https://github.com/LilMGenius/paperthin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
