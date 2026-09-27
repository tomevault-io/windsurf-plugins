---
trigger: always_on
description: npm test                 # all tests (node --test)
---

# AGENTS.md — HuaweiCloud DevKit

## Commands

```bash
npm test                 # all tests (node --test)
npm run lint             # ESLint + markdownlint
npm run lint:js          # ESLint only
npm run lint:md          # markdownlint only
npm run format           # Prettier format all files
npm run format:check     # Prettier check (no write)
npm run validate         # structural validation + README beta badge sync check
npm run badge:sync       # rewrite README beta badge to the next stable version
node --test test/structure.test.mjs   # single test file
node ./scripts/validate-package.mjs   # validation alone
```

No build step, no typecheck. One runtime dependency (undici for proxy support).

## Architecture

This is an **agent guidance + safety package**, not a service encyclopedia. Six compact meta-skills route agent intent to the right capability path (Skills / KooCLI / API / SDK / MCP / Terraform).

```
plugins/huaweicloud-core/
  skills/           ← 6 meta-skills + service skills
  src/              ← Node.js MCP server (stdio/remote JSON-RPC, 39 tools in tools.mjs)
  safety/           ← shared policy.json + risk rules
  hooks/            ← PreToolUse hook (Node huaweicloud-safety.mjs, wired via hooks.json; .py variant kept for compatibility)
  .codex-plugin/    ← Codex plugin manifest
  .claude-plugin/   ← Claude Code plugin manifest
  .cursor-plugin/   ← Cursor plugin manifest
  .workbuddy-plugin/← WorkBuddy plugin manifest
  .hermes-plugin/   ← Hermes plugin manifest
  .mcp.json         ← MCP server config for agents
  openclaw.plugin.json ← OpenClaw plugin manifest
```

Safety is 3-layer: **skills teach → hooks block → MCP/CLI wrappers enforce**.

Also in the repo:

- `bin/setup.cjs` — interactive installer (`huaweicloud-devkit`); dispatches to each agent's plugin dir.
- `integrations/` — per-agent adapter configs (opencode, dsh, hermes, workbuddy, atomcode), separate from the plugin.
- `src/tools.mjs` — 39 MCP tool definitions (hcloud CLI, hooks, catalog, auth, sandbox, voucher, update). `src/mcp-server-remote.mjs` — remote (HTTP) transport alongside stdio.
- `src/setup-cli.mjs` — KooCLI install/doctor logic; honors `HCLOUD_BIN`.
- `scripts/*.mjs` — validation, version sync, packaging, release helpers.
- `.superpowers/` + `docs/superpowers/` — planning/spec workflow used for larger changes.

## Skill Naming: Meta vs Service

- **Meta-skills** (`huaweicloud-*`, 6 required): horizontal capability skills such as routing, discovery, CLI/auth, API/SDK, safety, troubleshooting. Agent always starts here.
- **Service skills** (`huawei-*`): vertical domain knowledge for specific Huawei Cloud services (ecs, obs, vpc, iam, dew, etc.). Loaded via `huaweicloud_retrieve_skill` after routing by the core meta-skill.

Required meta-skills (tethered to `test/structure.test.mjs`):
`huaweicloud-api-and-sdk`, `huaweicloud-capability-discovery`, `huaweicloud-cli-and-auth`, `huaweicloud-core`, `huaweicloud-safety`, `huaweicloud-troubleshooting`

## File Naming: Design Docs vs Implementation

`docs/` holds planning/design artifacts (`*-design.md`, `architecture.md`, `safety-model.md`, ...). These are historical and may lag reality. The **actual implementation** is in `plugins/huaweicloud-core/` — `huaweicloud-*` for meta-skills and `huawei-*` for service skills. Trust the filesystem, not the docs.

## Creating or Editing Skills

- Every `SKILL.md` must start with `---\nname: huaweicloud-<name>` or `---\nname: huawei-<name>` YAML frontmatter (validated by both `npm run validate` and `structure.test.mjs`)
- No `TODO` or `[TODO]` markers in committed files (also validated)
- The 6 meta-skills must always exist. Service skills can be added freely. `test/structure.test.mjs` enforces a minimum of 6 skills and `scripts/validate-package.mjs` a minimum of 5; the installed set is not an exact count.
- Update `test/structure.test.mjs` if introducing new testable invariants (e.g., new required sections in SKILL.md)
- Add `node --test` tests if introducing new measurable invariants

### Skill Design Principles

**Parameters are discovered via `--help`, not hardcoded.** Every service skill must instruct the agent:

> Always run `hcloud <Service> <Operation> --help` before constructing commands to discover exact parameter names and requirements.

The skill provides the correct **service name and operation names** (which agents cannot reliably discover). Parameters come from `--help` (which is self-documenting and never stale).

**Three-class parameter value rule.** When a command in a SKILL.md or reference file contains a concrete value (not a `<placeholder>`), classify it before committing:

| Class           | Definition                                | Action                           |
| --------------- | ----------------------------------------- | -------------------------------- |
| **HELPFUL**     | `--help` cannot reveal this knowledge     | **Keep** the concrete value      |
| **UNNECESSARY** | `--help` already documents this correctly | **Replace** with `<placeholder>` |
| **WRONG**       | Contradicts what `--help` says            | **Fix immediately**              |

```
HELPFUL examples (keep):
  --publicip.associate_instance_type=PORT   # ECS→PORT mapping is non-obvious

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huaweicloud/huaweicloud-devkit](https://github.com/huaweicloud/huaweicloud-devkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
