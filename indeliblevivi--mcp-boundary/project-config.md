---
trigger: always_on
description: - `src/skills/mcp-boundary/` is the author source for the active skill, agent metadata, assets, and bundled references. It is the only maintained and distributed skill.
---

# MCP Boundary repository contract

## Truth owners

- `src/skills/mcp-boundary/` is the author source for the active skill, agent metadata, assets, and bundled references. It is the only maintained and distributed skill.
- `src/plugin/plugin.json` is the author source for the Codex-native manifest. It uses top-level `skills` and `interface` fields.
- `plugins/mcp-boundary/` is generated distributable output. Change `src/`, legal, provenance, or approved public assets first, then run `python3 scripts/build_plugin.py --write`.
- `plugins/mcp-boundary/.codex-plugin/plugin.json` is the only distributed manifest and is copied byte-for-byte from the author source. Do not hand-edit it.
- `guide/` and `lab/` are ordinary maintained subtrees of this repository, not mirrors of still-authoritative upstream repositories. `provenance/UPSTREAMS.lock.json` owns the initial-import pins and continuing-authority record; `provenance/SOURCES.json` owns the exact-copy and influence map.
- `tools/guide-validation/` is the maintained authority for Guide validators used by current tests and CI.
- `guide/skill/mcp-server-engineering/` is frozen historical release and evaluation material, including its self-contained historical scripts. It is not active, not distributed, not a recommended installation path, and not a maintenance target. Current callers must not execute its scripts.
- `evaluations/` owns behavior cases and scoring rules for the installed skill. Registered cases do not establish behavior change without an executed run.
- `brand/`, `site/`, and `previews/` contain the one approved public identity: Offset + Porcelain. Do not reintroduce private identity comparisons, alternative marks, alternative palettes, or appearance controls.
- Root `.github/workflows/` owns repository CI and deployment: `pages.yml` is the canonical Pages path and publishes only `site/` to the `github-pages` environment, and `validate.yml` runs the plugin, Guide, and Lab checks. The imported `guide/.github/workflows/` and `lab/.github/workflows/` files are retired historical sources and are not executed.
- `docs/current-state.md` owns volatile source/package/publication status. README files own durable public product and installation behavior.

## Boundaries

- The distributed plugin is a pure-skill package. Adding an MCP server, app connector, hook, lifecycle service, authentication flow, telemetry, or remote runtime to it is an architecture change that requires owner intent. The executable Lab under `lab/` is repository evidence source, not plugin runtime.
- Keep one distributed skill. Do not add a second install path, revive the historical Guide skill, or copy its workflow wholesale into the active skill. The active skill must stay self-contained: its references must resolve inside its own package, and installed execution must not depend on reading `guide/`.
- There is no upstream re-import, updater, or synchronization path. Do not reintroduce a script that replaces `guide/` or `lab/`; later upstream changes arrive as ordinary reviewed pull requests with their own provenance note.
- Keep root `plugins/mcp-boundary/plugin.json` absent. Its presence selects the submission system's Agent Plugins conversion path and bypasses the native manifest metadata, including composer icon and logo fields.
- ZIP submission currently admits skills only. Keep `interface.screenshots` and packaged screenshot assets absent; public website previews remain outside the plugin package.
- Treat bundled protocol profiles as dated evidence. Verify current official sources for “latest” protocol, host, SDK, or plugin-platform claims.
- Preserve the file-level license and provenance map in `LICENSING.md` and `provenance/SOURCES.json`.
- Private Faye/Cove continuity, design studies, raw exports, and handoffs never enter this worktree or a remote. Keep them in the configured private-continuity root outside the worktree.
- Do not change the Pages topology or custom domain, submit to an external plugin directory, install into a user profile, or publish a release without matching authority. Building a local ZIP does not imply any of those states.

## Verification

For a source change, run the narrow relevant checks. The full local release-candidate set is:

```bash
python3 scripts/build_plugin.py --write
python3 scripts/build_plugin.py --check
python3 -m unittest discover -s tests -p 'test_*.py'
python3 tests/static_check.py
skill-validate src/skills/mcp-boundary
skill-validate plugins/mcp-boundary/skills/mcp-boundary
python3 scripts/package_plugin.py
```

Remote CI reproduces the current Codex skill-schema checks with pinned PyYAML via `scripts/validate_skill_schema.py` and validates both the author source and generated package. Local `skill-validate` remains the release-candidate command backed by the installed Codex Skill tooling.

Run the affected subtree checks when changing `guide/`:

```bash
cd guide
PYTHONDONTWRITEBYTECODE=1 python3 -m unittest discover -v
python3 ../tools/guide-validation/validate_version_register.py VERSION-REGISTER.json
python3 ../tools/guide-validation/check_profile_mirrors.py VERSION-REGISTER.json

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IndelibleVivi/mcp-boundary](https://github.com/IndelibleVivi/mcp-boundary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
