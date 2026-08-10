---
trigger: always_on
description: npm test                 # all tests (node --test)
---

# AGENTS.md — HuaweiCloud Devkit

## Commands

```bash
npm test                 # all tests (node --test)
npm run validate         # structural validation
node --test test/structure.test.mjs   # single test file
node ./scripts/validate-package.mjs   # validation alone
```

No build step, no linter, no typecheck. Zero runtime npm dependencies.

## Architecture

This is an **agent guidance + safety package**, not a service encyclopedia. Six compact meta-skills route agent intent to the right capability path (Skills / KooCLI / API / SDK / MCP / Terraform).

```
plugins/huaweicloud-core/
  skills/           ← 6 meta-skills + service skills
  src/              ← Node.js MCP server (stdio JSON-RPC, 12 tools)
  safety/           ← shared policy.json
  hooks/            ← Python PreToolUse hook
  .codex-plugin/    ← Codex plugin manifest
  .claude-plugin/   ← Claude Code plugin manifest
  .cursor-plugin/   ← Cursor plugin manifest
  .mcp.json         ← MCP server config for agents
```

Safety is 3-layer: **skills teach → hooks block → MCP/CLI wrappers enforce**.

## Skill Naming: Meta vs Service

- **Meta-skills** (`huaweicloud-*`, 6 required): horizontal capability skills such as routing, discovery, CLI/auth, API/SDK, safety, troubleshooting. Agent always starts here.
- **Service skills** (`huawei-*`): vertical domain knowledge for specific Huawei Cloud services (ecs, obs, vpc, iam, dew, etc.). Loaded via `huaweicloud_retrieve_skill` after routing by the core meta-skill.

Required meta-skills (tethered to `test/structure.test.mjs`):
`huaweicloud-api-and-sdk`, `huaweicloud-capability-discovery`, `huaweicloud-cli-and-auth`, `huaweicloud-core`, `huaweicloud-safety`, `huaweicloud-troubleshooting`

## File Naming: Design Docs vs Implementation

Design docs in `docs/` use `huawei-*` and plan 20+ service skills. The **actual implementation** uses `huaweicloud-*` for meta-skills and `huawei-*` for service skills. Design docs are planning artifacts; trust the filesystem.

## Creating or Editing Skills

- Every `SKILL.md` must start with `---\nname: huaweicloud-<name>` or `---\nname: huawei-<name>` YAML frontmatter (validated by both `npm run validate` and `structure.test.mjs`)
- No `TODO` or `[TODO]` markers in committed files (also validated)
- The 6 meta-skills must always exist. Service skills can be added freely — `test/structure.test.mjs` enforces a minimum of 6, not an exact count.
- Update `test/structure.test.mjs` if introducing new testable invariants (e.g., new required sections in SKILL.md)
- Add `node --test` tests if introducing new measurable invariants

### Skill Design Principles

**Parameters are discovered via `--help`, not hardcoded.** Every service skill must instruct the agent:
> Always run `hcloud <Service> <Operation> --help` before constructing commands to discover exact parameter names and requirements.

The skill provides the correct **service name and operation names** (which agents cannot reliably discover). Parameters come from `--help` (which is self-documenting and never stale).

**Three-class parameter value rule.** When a command in a SKILL.md or reference file contains a concrete value (not a `<placeholder>`), classify it before committing:

| Class | Definition | Action |
|-------|-----------|--------|
| **HELPFUL** | `--help` cannot reveal this knowledge | **Keep** the concrete value |
| **UNNECESSARY** | `--help` already documents this correctly | **Replace** with `<placeholder>` |
| **WRONG** | Contradicts what `--help` says | **Fix immediately** |

```
HELPFUL examples (keep):
  --publicip.associate_instance_type=PORT   # ECS→PORT mapping is non-obvious
  --x_cff_request_version=v0                # v0=raw, v1=APIG-wrapped semantics
  --delete_publicip=true                    # default=false leaks EIP; teach override
  --code_type=inline                        # zip unreliable on KooCLI
  --loadbalancer_provider=elb               # elb=public, lvs=internal-only

UNNECESSARY examples (replace with placeholder):
  --publicip.type=5_bgp       → <type>       # --help lists valid types
  --bandwidth.size=5          → <size>       # user-determined
  --security_group_rule.direction=ingress → <direction>  # --help lists ingress/egress
  --server.root_volume.volumetype=SSD → <type>          # --help lists SSD/SAS/…
  --cli-region=cn-north-4     → <region>     # example region
```

**Duplication rule:** If the same UNNECESSARY value repeats across 3+ files, fix all occurrences together. Single-occurrence values in reference files are acceptable tradeoffs for teaching clarity.

**Reference files vs SKILL.md:** SKILL.md is the routing layer (~80 lines) — prefer placeholders or omit inline values entirely. `references/*.md` are teaching files — complete working commands are expected, but UNNECESSARY values should still use placeholders unless the value itself is the teaching point.

**Only document non-obvious traps.** If `--help` already explains a parameter correctly, don't repeat it. Document what `--help` gets wrong:
- Parameters marked optional that are actually required (e.g., `protocol`/`sl_domain`/`env_name`/`env_id` for DEDICATEDGATEWAY)
- Deprecated values (e.g., `APIG` trigger type, use `DEDICATEDGATEWAY`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huaweicloud/HuaweiCloud-Devkit](https://github.com/huaweicloud/HuaweiCloud-Devkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
