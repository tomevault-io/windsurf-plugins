---
trigger: always_on
description: The complete development workflow is documented as a Mermaid diagram:
---

# Repository instructions

## Development workflow

The complete development workflow is documented as a Mermaid diagram:

```text
docs/WORKFLOW.md
```

Read it to understand the full cycle: edit → local deploy → commit → release
→ CI publish → post-release install test. The sections below elaborate each
phase with exact commands.

## Source of truth

- Work only in this repository's root directory.
- Never edit `lib/index.js`, `lib/client.js`, or the installed package under `~/.dsh/profiles` directly.
- Host source lives in ordered fragments under `src/host/`.
- Browser source lives in ordered fragments under `src/client/`.
- Keep each fragment below 450 lines; split by responsibility before it exceeds the limit.

## Feature design authority

Before implementing vision or document capabilities, read:

```text
docs/VISION_AND_DOCUMENT_HANDOFF.zh-CN.md
```

Do not expand that scope (especially generic composer file upload or DSH core changes) without explicit user approval.

## DeepSeek Harness version dependency

- This plugin targets **`@deepseek-ai/dsh` `0.1.1-rc.2`** and the matching
  `@deepseek-ai/dsh-*` packages (`^0.1.1-rc.2`), with `@deepseek-ai/cordis`
  `^4.0.1`. See `peerDependencies` in `package.json`.
- The vision path depends on APIs introduced in `0.1.1-rc.2`:
  `AttachmentStore.readImageRequest()`, and `offloadRequestImagesWithPolicy` /
  `requestImageHandleText` from `@deepseek-ai/dsh-llm`. Earlier releases
  (`0.1.0-rc.x`) lack these and will not run the current adapter.
- Do not lower the baseline below `0.1.1-rc.2` without an explicit,
  user-approved compatibility change.

## Required workflow

Before editing:

```bash
git status --short
```

After editing:

```bash
npm run build
npm test
npm run check
git diff --stat
git diff
```

Commit source and generated `lib/` artifacts together. Use Conventional Commit-style messages.

Deploy only after tests pass:

```bash
npm run deploy
```

The deploy command creates a rollback backup and atomically replaces the profile package. Restart `dsh web` after Host changes; hard-refresh the browser after Client changes.

## Dependency management

`scripts/deploy.mjs` does **not** run `npm install` — it copies files directly.
Runtime `dependencies` must therefore be kept in sync in two places:

1. Add the package to `dependencies` in `package.json` and run `npm install`.
2. Add the same name to the `bundledDeps` array in `scripts/deploy.mjs`.

`deploy.mjs` copies each entry from local `node_modules/` into the staged
plugin directory. Both `npm publish` (npm installs deps automatically from
`dependencies`) and `github:` source installs (pnpm installs deps into the
profile's hoisted `node_modules/`) resolve deps without needing them bundled
in the tarball — so **no `bundledDependencies` field is used**.
`peerDependencies` and `devDependencies` are excluded from `bundledDeps`.

## Release file checklist

Every file that must be present in the installed plugin must appear in **both**:

- `files` array in `package.json` (controls `npm publish` tarball)
- `releaseEntries` array in `scripts/deploy.mjs` (controls local deploy)

Currently required entries: `cordis.patch.yml`, `lib/index.js`, `lib/client.js`,
`package.json`, `README.md`, `README.zh.md`, `CHANGELOG.md`, `AGENTS.md`,
`LICENSE`, `docs/**`.

> `cordis.patch.yml` is mandatory: DSH loads this package as a bundle and
> reads `dsh.bundle.patch` at boot time. A missing file causes `ENOENT` on start.

## GitHub authentication

This repository has no stored SSH key or long-lived token. Use the global
`github-auth` Pi skill to obtain a session-scoped OAuth token:

```bash
# Get (or reuse cached) token
TOKEN=$(python3 ~/.pi/agent/skills/github-auth/scripts/get_token.py)

# Push commits
REPO=$(git remote get-url origin | sed 's|https://github.com/||')
git push "https://${TOKEN}@github.com/${REPO}" main

# Push a tag
git push "https://${TOKEN}@github.com/${REPO}" v0.x.y
```

The token is cached in `/tmp/.pi_github_token` for the OS session.
Do not echo or log the token value.

## Releases

- Update `CHANGELOG.md` before a release.
- Use SemVer with `npm version patch|minor|major`.
- Build a local release artifact with `npm run pack:local`.
- Do not reuse a released version for changed runtime code.
- Pushing a `v*` tag triggers the GitHub Actions Release workflow, which
  publishes to npm via OIDC Trusted Publishing (no `NPM_TOKEN` needed)
  and creates a GitHub Release automatically.

## Post-release installation test

After every tag push, verify both install sources before declaring the release
done. Run the steps below in order.

### 0. Wait for CI to publish

```bash
TOKEN=$(python3 ~/.pi/agent/skills/github-auth/scripts/get_token.py)
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://api.github.com/repos/lujianjun19/dsh-llm-github-copilot/actions/runs?per_page=3" \
  | python3 -c "
import sys,json
for r in json.load(sys.stdin).get('workflow_runs',[]):
    print(r['name'], '|', r['status'], '|', r['conclusion'] or '-', '|', r['head_branch'] or '')
"
```

Wait until the `Release` workflow shows `completed | success` for the new tag.

### 1. Helper — clean the profile slot

Run this before each source test to start from a clean state:

```bash
cd ~/.dsh/profiles/web
python3 -c "

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lujianjun19/dsh-llm-github-copilot](https://github.com/lujianjun19/dsh-llm-github-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
