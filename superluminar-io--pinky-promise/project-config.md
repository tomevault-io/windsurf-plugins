---
trigger: always_on
description: This plugin manages API contracts between producer and consumer services. It integrates with the superpowers development workflow.
---

# pinky-promise

This plugin manages API contracts between producer and consumer services. It integrates with the superpowers development workflow.

**Requires superpowers to be installed and active.**

## Working on this plugin

**These rules apply only when the current project IS the pinky-promise plugin repository** — i.e. the working directory contains `skills/api-spec-brainstorming/`. In all other projects, skip this entire section. It is included here so it ships with the plugin and is available to contributors working on the plugin itself.

### Semver commitment

| Change | Bump |
|---|---|
| Bug fix, skill clarification, improved wording | patch |
| New optional IDL/bindings field, new skill, new auth type, new CLAUDE.md hook | minor |
| Rename or remove any field in contract files or `bindings.json` | **major** |
| Change the semantics of an existing field | **major** |
| Change the registry layout (where files live, naming) | **major** |
| Change `credentials.json` structure | **major** |
| Remove or rename a skill | **major** |
| Change a CLAUDE.md hook in a way that stops it from firing | **major** |

### Before making any change

1. Check whether it is breaking. If you are unsure, assume it is.
2. Determine the correct semver bump using the table above.
3. Apply the bump to `plugin.json`, `marketplace.json`, and `package.json` in the same commit as the change — never leave them unversioned. All three must always carry the same version string.

**For non-breaking changes:** apply the bump and proceed.

**For breaking changes:** stop and tell the user:
> "This change is breaking — it requires a major version bump. It will invalidate existing registry files / consumer workflows for anyone on the current version. Do you want to proceed?"

Wait for explicit approval before continuing.

### When a breaking change is approved

1. Bump `pinkyPromiseVersion` in all skills that write registry files (brainstorming, import, publish) from the current value to the next integer.
2. Update the version check threshold in all skills that read registry files (guardian, contract-check) to match.
3. Update `plugin.json` and `marketplace.json` version fields.
4. Propose building a migration skill:
   > "Should I add an `/api-spec-migrate` skill that reads v[old] registry files and rewrites them to v[new] format? This lets existing users upgrade their registries without re-publishing every service."

Wait for the user's decision before implementing the migration skill.

## Registry is the only source of truth for API contracts

**NEVER** read another service's code, spec files, or any other files outside the current project to infer its API. This means:

- No `find`, no directory traversal, no reading files from `..`, `~`, or any path outside the current working directory
- No reading `.json`, `.proto`, `.yaml`, `.go`, `.ts`, or any other file from a sibling service directory to infer what that service provides
- No assumptions based on what happens to be checked out locally

When implementing a client or validating a consumer, the **only** permitted source of truth for what another service provides is its published spec in the registry — fetched via a fresh `git clone` of `API_REGISTRY_REPO` into `.pinky-promise/registry/`. If the registry is unreachable or has no entry for the service, say so and stop.

## Configuration

Set the registry URL in this project's CLAUDE.md (below this file's content) or in `.claude/settings.json`:

```
API_REGISTRY_REPO=git@github.com:yourorg/api-registry.git
```

If `API_REGISTRY_REPO` is not set, all skills warn and skip silently — they never block work.

## Session start

If `API_REGISTRY_REPO` is configured:
1. Identify the current service name (from project directory, `.pinky-promise/draft-spec.json`, or draft spec in context)
2. Resolve `API_REGISTRY_REPO`: read `.claude/settings.json` (check `env.API_REGISTRY_REPO`) then project `CLAUDE.md` (line matching `API_REGISTRY_REPO=`). Use the Read tool — no shell execution. If not found, skip silently.
3. Fetch the registry fresh — always clone from `API_REGISTRY_REPO`, never read from local service directories:
   ```bash
   rm -rf .pinky-promise/registry
   git clone --depth 1 --filter=blob:none --sparse "$API_REGISTRY_REPO" .pinky-promise/registry 2>/dev/null
   git -C .pinky-promise/registry sparse-checkout set "services/<service-name>" 2>/dev/null
   ls .pinky-promise/registry/services/<service-name>/ 2>/dev/null | sort -V | tail -1
   ```
4. If a spec version is found, read it into context silently:
   ```bash
   cat .pinky-promise/registry/services/<service-name>/<latest-version>.json
   rm -rf .pinky-promise/registry
   ```
   Do not announce this to the user when it succeeds. If the clone fails, clean up and warn the user once:
   > "⚠️ pinky-promise: could not reach the API registry (`$API_REGISTRY_REPO`). Contract checks and the change guardian are disabled for this session. Check your SSH key and network access."

   Do not block the session — continue without registry data, but make sure the user knows the safety net is off.

## When the user is designing or building a service


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [superluminar-io/pinky-promise](https://github.com/superluminar-io/pinky-promise) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
