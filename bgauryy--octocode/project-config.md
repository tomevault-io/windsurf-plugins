---
trigger: always_on
description: Agent guide for `@octocodeai/skills`. Read this before any edit to this package.
---

# AGENTS.md — `packages/octocode-skills`

Agent guide for `@octocodeai/skills`. Read this before any edit to this package.
Root `AGENTS.md` applies everywhere; this file narrows scope to this package only.

---

## What this package is

A zero-runtime-dep CLI + library that:
1. **Bundles** all skills from `../../skills/` at build time into `skills/` (package root).
2. **Lists** bundled skills with install status and env readiness.
3. **Installs** skills to a canonical home (`~/.octocode/skills/<name>/`) then symlinks into platform dirs.
4. **Checks** all installation locations and env param configuration.
5. **Informs** — every command tells the user what is missing and how to fix it.

**Hard invariants:**
- Zero npm runtime dependencies — Node.js builtins only (`node:fs`, `node:path`, `node:os`, `node:url`).
- `src/cli.ts` must NOT start with `#!/usr/bin/env node` — esbuild adds it via `banner`; a source shebang creates a double-shebang that crashes Node ESM.
- All TypeScript is compiled with `exactOptionalPropertyTypes: true` — never assign `undefined` to an optional property; use conditional spread `...(x !== undefined ? { x } : {})`.
- `out/` is always generated — never edit it by hand; always rebuild after changes.

---

## Directory map

```
packages/octocode-skills/
├── src/
│   ├── cli.ts              ← CLI entry point — arg parsing, command dispatch, help text
│   ├── index.ts            ← Library exports (programmatic API)
│   ├── registry.ts         ← Read bundled skills list; parse SKILL.md frontmatter
│   ├── installer.ts        ← Copy/symlink a skill to home, platform dirs, workspace, custom path
│   ├── checker.ts          ← Probe install locations: installed | linked | broken | missing
│   ├── env-params.ts       ← Static env param registry per skill; runtime set/missing check
│   ├── home.ts             ← getOctocodeHome(), getSkillsHome() — inlined, zero deps
│   ├── platforms.ts        ← Platform → dir mapping; parsePlatforms()
│   ├── commands/
│   │   ├── list.ts         ← `octocode-skills list`    — skills + install + env status
│   │   ├── install.ts      ← `octocode-skills install` — install with override/keep + env warning
│   │   ├── check.ts        ← `octocode-skills check`   — verify install + env per skill
│   │   └── info.ts         ← `octocode-skills info`    — SKILL.md + env params detail
│   └── utils/
│       ├── colors.ts       ← dim / bold / green / yellow / red / cyan — single-file, no dep
│       └── spinner.ts      ← TTY spinner on stderr; silent when !isTTY or CI=true
├── skills/                 ← GENERATED at build — copy of ../../skills/ (no scripts/ dirs)
├── out/                    ← GENERATED — esbuild bundles (cli.js, index.js)
├── build.mjs               ← Build script: clean → sync skills → esbuild CLI + index
├── package.json
├── tsconfig.json
└── README.md
```

---

## Data flow

```
../../skills/<name>/SKILL.md   ← source of truth for skill content
        ↓  build.mjs (sync at build time, excludes scripts/)
skills/<name>/SKILL.md         ← bundled copy inside this package
        ↓  src/registry.ts
listSkills() / getSkill()      ← SkillInfo { name, folder, description, dir }
        ↓  src/env-params.ts
getSkillEnvStatus()            ← SkillEnvStatus { readiness, params[] }
        ↓  src/checker.ts
checkSkill()                   ← SkillCheckResult { home, platforms[], workspace }
        ↓  src/commands/*.ts
list / install / check / info  ← human or JSON output to stdout
```

All four commands read from these three modules. Changes to skill content flow through
registry; changes to env requirements flow through env-params; installation state flows
through checker.

---

## Task: Adding a new skill

**1. Create the skill folder** in `../../skills/<new-name>/` with at least a `SKILL.md`
   that has YAML frontmatter:
   ```yaml
   ---
   name: <new-name>
   description: "One sentence — shown in list and info."
   ---
   ```

**2. Register env params** in `src/env-params.ts` → `SKILL_ENV_PARAMS`:
   ```ts
   // If the skill needs no env params — do nothing. Skills absent from the map are treated as "ok".

   // If it needs web search (like brainstorming):
   'new-skill-name': WEB_SEARCH_PARAMS,

   // If it needs GitHub token (like research):
   'new-skill-name': GITHUB_TOKEN_PARAMS,

   // If it needs something unique:
   'new-skill-name': [
     {
       key: 'MY_KEY',
       description: 'What it is for',
       required: 'recommended',   // 'required' | 'recommended' | 'optional'
       link: 'https://where-to-get.it/',
       // group: 'my-group',  ← add only if AT LEAST ONE of multiple keys is enough
     },
   ],
   ```

**3. Rebuild and verify:**
   ```bash
   cd packages/octocode-skills
   node build.mjs
   node out/cli.js list
   node out/cli.js info <new-name>
   node out/cli.js list --json | python3 -c "import sys,json; s=json.load(sys.stdin)['skills']; [print(x['name'], x['env']['readiness']) for x in s]"
   ```

**4. Update README.md** — add a skill entry under `## Bundled skills` following the
   existing pattern: name, one-liner, when-to-use, env params table.

**No other files need to change.** `registry.ts` auto-discovers all SKILL.md folders;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bgauryy/octocode](https://github.com/bgauryy/octocode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
