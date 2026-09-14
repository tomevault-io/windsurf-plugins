---
trigger: always_on
description: CLI for batching files over SSH so a remote coding agent gets one folder path.
---

# AGENTS.md

CLI for batching files over SSH so a remote coding agent gets one folder path.

Human notes: `CONTRIBUTING.md`. Leave `web/AGENTS.md` and `web/CLAUDE.md` alone (Next.js regenerates them).

## Layout

- CLI: `src/` (TypeScript). Do not commit `dist/`
- Docs site: `web/` (Next.js). Product docs: `web/content/docs/`. Search-intent guides: `web/content/learn/`
- npm package: `@nyxsky404/vmup`; command: `vmup`

## When to touch what

| Kind of work | Changelog | Docs | Version bump + npm |
| --- | --- | --- | --- |
| User-facing CLI (command, flag, default, output, JSON, error, exit code) | Yes | Yes, every page that states the fact | Only when cutting a release |
| Docs/site/Learn copy only | No | Yes | No |
| Tests / refactor, same behavior | No | No | No |
| New docs or Learn page | No (unless it ships with a CLI change) | Page + nav + SEO (see below) | No |

## 1. CLI change → docs everywhere

A user-facing CLI change is not done until every surface that describes it matches the code.

### Always

1. `CHANGELOG.md` — bullet under `## Unreleased`
2. `web/content/docs/changelog.mdx` — **same bullets** (hand-copied, not generated)
3. `--help` text in `src/cli.ts` if a command/flag/description changed
4. Tests in `src/*.test.ts` if behavior changed, then `npm run build && npm test`

### Then every page that states the fact

| If you changed | Also update |
| --- | --- |
| Command | `web/content/docs/reference/commands.mdx`, matching guide, `README.md` Commands |
| Flag | `reference/flags.mdx`, `src/cli.ts`, guides that name it, `README.md` Usage |
| Config key / default | `reference/config.mdx`, `README.md` Config, `src/constants.ts` if the default lives there |
| Env var | `reference/env.mdx` (and config page if it overlays the same key) |
| `--json` fields | `reference/json.mdx`, `README.md` JSON, `src/output.ts` |
| Exit code | `reference/exit-codes.mdx`, `README.md` |
| Error string | `reference/errors.mdx` |
| TTL / sweeper / prune | `explain/ttl.mdx`, `guides/cleanup.mdx`, `README.md` Cleanup, homepage TTL copy if the default changed |
| Clipboard / prompt | `guides/clipboard.mdx`, `guides/clipboard-for-agents.mdx`, `guides/screenshots-to-agent.mdx`, `README.md`, `web/content/docs/index.mdx`, `web/app/(home)/page.tsx`, `web/lib/shared.ts` (`appDescription`) |
| Collect mode (args, picker, clip, watch) | matching guide, `explain/how-it-works.mdx`, homepage beats |
| File types / `--video` / `--force` | `reference/file-types.mdx`, `guides/restrict-types.mdx` |
| SSH / profiles | `guides/ssh.mdx`, `guides/profiles.mdx`, `explain/ssh-modes.mdx`, `README.md` Profiles |
| Quickstart steps | `quickstart.mdx`, `README.md` Quickstart, `web/lib/schema.ts` HowTo if steps changed |
| Product one-liner / defaults | `README.md`, `web/content/docs/index.mdx`, `web/lib/shared.ts`, homepage |

`docs/plans/vmup-design.md` only if the product definition changed (pipeline, naming, defaults, platforms).

`README.md` is what npm shows. Stale README = stale npm page after publish.

## 2. Cut a release (version + changelog + npm)

Do this only when publishing. Everyday PRs stay on `## Unreleased`.

### Changelog freeze

In **both** `CHANGELOG.md` and `web/content/docs/changelog.mdx`:

- Rename `## Unreleased` to `## x.y.z — YYYY-MM-DD`
- Put an empty `## Unreleased` back on top
- Keep the two files identical

### Version (all of these, same number)

CLI version is **hardcoded**. Updating only `package.json` will not change `vmup --version`.

- `package.json` → `version`
- `package-lock.json` (via `npm install` at repo root after the bump)
- `src/constants.ts` → `PACKAGE_VERSION` (this is `--version` and the update notice)
- `web/package.json` → `version` (private site; keep in lockstep)
- `web/package-lock.json`
- `web/lib/shared.ts` → `softwareVersion` (JSON-LD)
- `web/content/docs/install.mdx` — “You should see x.y.z”
- `web/content/docs/reference/commands.mdx` — “Version is x.y.z in this tree”

Do **not** bulk-replace version strings in `src/update-check.test.ts`. Those are fixtures.

### Before `npm publish`

```bash
npm run build && npm test
npm pack --dry-run
```

Confirm:

- [ ] Git working tree is what you intend to ship (docs + changelog + version on the same commit)
- [ ] `prepublishOnly` will run `build` + `test` (already in `package.json`; do not skip with `--ignore-scripts`)
- [ ] `npm pack --dry-run` includes `dist/`, `scripts/install.sh`, `README.md`, `LICENSE`, `CHANGELOG.md`
- [ ] Pack list does **not** include `src/`, `web/`, tests (`dist/**/*.test.js` is excluded), `.env`, keys
- [ ] `package.json` `homepage` is `https://vmup.dev`
- [ ] `package.json` `files` lists any new runtime artifact (a new script under `scripts/` is omitted until added here)
- [ ] `vmup --version` after build prints the **new** version (`node dist/cli.js --version`)
- [ ] CI would pass: root `npm test` (Node 18+) and `web/` `npm run build`

Publish the CLI package only (`@nyxsky404/vmup` at repo root). Never publish `web/` (`private: true`).

```bash
npm publish
```

Access is already `publishConfig.access = public`.

### After publish

- [ ] `npm view @nyxsky404/vmup version` is the new version
- [ ] npm README still looks right (it is repo `README.md`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nyxsky404/vmup](https://github.com/nyxsky404/vmup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
