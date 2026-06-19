---
trigger: always_on
description: Agents can assemble UI. What they cannot reliably preserve is the product
---

# Ghost

Agents can assemble UI. What they cannot reliably preserve is the product
surface composition behind that UI: hierarchy, density, restraint, repetition,
trust, flow, and the choices that make a surface feel intentional.

Ghost keeps that surface composition in a repo-local `.ghost/` fingerprint package. The public npm shape
is one package, `@anarchitecture/ghost`, with one user-facing bin, `ghost`.
The CLI validates, computes, compares, and emits deterministic packets. The
host agent does the interpretive BYOA work through the installed `ghost` skill.

## Build & Run

```bash
pnpm install          # install dependencies (pnpm 10+, Node 20.19+ or 22.12+)
pnpm build            # build all packages
pnpm test             # vitest across packages
pnpm check            # biome, typecheck, file-size, CLI manifest drift
pnpm dump:cli-help    # regenerate apps/docs/src/generated/cli-manifest.json
```

Run the public CLI after building:

```bash
node packages/ghost/dist/bin.js <command>
pnpm --filter @anarchitecture/ghost exec ghost <command>
```

## Architecture

Ghost is **BYOA (bring your own agent)**. Claude Code, Codex, Cursor, Goose, or
another host agent reads, decides, and writes. Ghost is the deterministic
calculator the agent reaches for: schema validation, inventory/cache helpers,
structural diffs, drift checks, comparison math, and handoff packets.

The canonical root `.ghost/` package follows:

```text
config.yml
fingerprint/
  manifest.yml
  prose.yml
  inventory.yml
  composition.yml
  enforcement/checks.yml
  memory/intent.md
  memory/decisions/
  sources/cache/
```

The three root layer files under `fingerprint/` are the core model:

- `prose.yml` for surface intent.
- `inventory.yml` for curated material, exemplars, and source links.
- `composition.yml` for experience patterns.

Supporting files live under purpose folders. `enforcement/checks.yml` is
validation and enforcement, not generation input. `memory/intent.md` and
`memory/decisions/` hold optional human context and rationale. Generated cache
in `sources/cache/` is optional source material; it answers what exists, while
core fingerprint layers answer what the surface is trying to do. `.ghost/config.yml`
remains local adapter/routing config outside portable context. Ordinary Git
review is the approval boundary for fingerprint edits.

Legacy `resources.yml`, `map.md`, `survey.json`, and `patterns.yml` may still
appear in older repos or as migration source material. They are not canonical
fingerprint input for new Ghost work.

## Packages

| Package | Published? | Description |
| --- | --- | --- |
| `packages/ghost` | yes: `@anarchitecture/ghost` | Unified public package. Ships the `ghost` CLI, fingerprint package authoring, checks, advisory review packets, comparison, drift stance verbs, and the unified skill bundle. |
| `packages/ghost-core` | no | Private historical shared package. Runtime code needed by npm is folded into `packages/ghost/src/ghost-core`. |
| `packages/ghost-fleet` | no | Private fleet view across many Ghost bundles. Consumes workspace exports from `@anarchitecture/ghost`. |
| `packages/ghost-ui` | no | Reference design system: shadcn registry plus `ghost-mcp` MCP server. |
| `apps/docs` | no | Docs site. |

## CLI Commands

| Command | Description |
| --- | --- |
| `ghost init` | Create `.ghost/fingerprint/` with manifest, core layers, and enforcement checks. |
| `ghost scan` | Report fingerprint layer readiness and BYOA next-step guidance. |
| `ghost inventory` | Emit raw repo signals as JSON for optional cache/source material. |
| `ghost lint` | Validate a fingerprint package or single artifact. |
| `ghost verify` | Validate fingerprint evidence and exemplar paths, typed check refs, and optional rationale files. |
| `ghost describe` | Print optional `fingerprint/memory/intent.md` or direct markdown section ranges. |
| `ghost diff` | Structural prose-level diff between direct fingerprints. |
| `ghost survey <op>` | Legacy/cache helpers for optional `ghost.survey/v1` workflows. |
| `ghost check` | Run active `ghost.checks/v1` deterministic gates against a diff. |
| `ghost review` | Emit an evidence-routed advisory review packet grounded in fingerprint layers, inventory exemplars, checks, and the diff. |
| `ghost compare` | Pairwise or composite comparison over packages or direct fingerprints. |
| `ghost ack` | Record stance toward the tracked fingerprint in `.ghost-sync.json`. |
| `ghost track` | Shift the tracked fingerprint. |
| `ghost diverge` | Declare intentional divergence on a dimension. |
| `ghost emit <kind>` | Emit `review-command`. |
| `ghost skill install` | Install the unified `ghost` agentskills.io bundle. |

`ghost scan --format json` is deterministic readiness and source-signal state.
It does not run an LLM.

## Public Exports

- `@anarchitecture/ghost` for the combined surface.
- `@anarchitecture/ghost/scan` for scan readiness, source signals, and stack discovery.
- `@anarchitecture/ghost/fingerprint` for fingerprint package authoring, linting, verification, parsing, and serialization.
- `@anarchitecture/ghost/drift` for check/review/compare/stance helpers.
- `@anarchitecture/ghost/core` for shared schemas, types, and loaders.
- `@anarchitecture/ghost/cli` for `buildCli()`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [block/ghost](https://github.com/block/ghost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
