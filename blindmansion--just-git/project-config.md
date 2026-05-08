---
trigger: always_on
description: Git implementation running inside the just-bash virtual shell. All commands operate on an in-memory virtual filesystem — nothing touches real disk.
---

# just-git

Git implementation running inside the just-bash virtual shell. All commands operate on an in-memory virtual filesystem — nothing touches real disk.

## Runtime

- **Bun** — runtime, test runner, package manager. No Node/npm/pnpm/Vite.
- **TypeScript** strict mode, ESNext target, bundler module resolution.
- `bun test` to run tests. No build step.

## Architecture

### Operator API (`src/git.ts`, `src/hooks.ts`)

`createGit(options?)` returns a `Git` instance — the top-level entry point for sandbox operators. Provides hooks, identity/credential overrides, config overrides, and command restriction without touching internals.

```ts
const git = createGit({
  identity: { name: "Agent", email: "agent@sandbox.dev", locked: true },
  disabled: ["push", "rebase", "remote", "clone", "fetch", "pull"],
  credentials: async (url) => ({ type: "bearer", token: "..." }),
  config: {
    locked: {
      "push.default": "nothing",
      "merge.conflictstyle": "diff3",
    },
    defaults: {
      "pull.rebase": "true",
      "merge.ff": "only",
    },
  },
  hooks: {
    preCommit: ({ repo, index }) => {
      /* inspect index, reject if needed */
    },
    postCommit: ({ repo, hash, branch }) => {
      /* audit log */
    },
    beforeCommand: ({ command, fs, cwd }) => {
      /* block commands, inspect args */
    },
  },
});

const bash = new Bash({ cwd: "/repo", customCommands: [git] });
```

**`GitOptions`:**

- `fs` — `FileSystem` for standalone `exec()`. When set, `exec` calls don't need to pass `fs` per-call.
- `cwd` — default working directory for `exec()`. Defaults to `"/"`. Per-call `cwd` in `ExecContext` overrides this. Set to the repo root so every `exec` call finds `.git` automatically.
- `hooks` — `GitHooks` config object with named callback properties. All hooks are optional. Specified at construction time. Use `composeGitHooks(...hookSets)` to combine multiple hook sets.
- `disabled` — `GitCommandName[]` of subcommands to block. Disabled commands return unknown-command errors.
- `identity` — `IdentityOverride` with `name`, `email`, optional `locked`. When `locked: true`, overrides env vars (`GIT_AUTHOR_NAME`, etc.); when unlocked (default), acts as fallback when env vars and git config are absent. Identity values are surfaced through `git config user.name` / `user.email` reads — locked identity becomes locked config, unlocked becomes default config.
- `credentials` — `CredentialProvider` callback `(url) => HttpAuth | null`. Provides auth for Smart HTTP transport. Takes precedence over `GIT_HTTP_BEARER_TOKEN`/`GIT_HTTP_USER` env vars.
- `onProgress` — `ProgressCallback` `(message: string) => void`. Called with server progress messages (sideband band-2) during fetch, clone, and push over HTTP. Messages are raw text from the remote — format varies by server. Only fires for `SmartHttpTransport` (not local transport).
- `config` — `ConfigOverrides` with `locked` and `defaults` maps. `locked` values always win over `.git/config` — the agent can run `git config set` but the locked value takes precedence on every read. `defaults` supply fallback values when a key is absent from `.git/config` — the agent _can_ override these with `git config`. Keys are dotted config names (e.g. `"push.default"`, `"merge.ff"`). Applied transparently via `getConfigValue()` so all commands respect overrides automatically.
- `resolveRemote` — `RemoteResolver` callback `(url) => GitRepo | null`. Resolves non-HTTP remote URLs to a `GitRepo`, enabling cross-VFS transport. Called before local filesystem lookup. Return null to fall back to `findRepo` on the local VFS. Enables multi-agent setups where each agent has its own isolated filesystem but can clone/fetch/push between repos on different VFS instances via `LocalTransport`. Also enables resolving to server-backed repos for hybrid in-process/server scenarios.

**Hooks** (`GitHooks` interface — config-at-construction, named callbacks):

All hook event payloads include `repo: GitRepo`, giving access to the repo module helpers (`getChangedFiles`, `readCommit`, `readFileAtCommit`, etc.) inside hooks.

Pre-hooks can reject operations by returning `{ reject: true, message?: string }` (`Rejection`):

- `preCommit` — `{ repo, index, treeHash }`. Fires before commit is created.
- `commitMsg` — `{ repo, message }` (mutable message). Fires after preCommit, before commit write.
- `mergeMsg` — `{ repo, message, treeHash, headHash, theirsHash }` (mutable message). Fires before merge commit.
- `preMergeCommit` — `{ repo, message, treeHash, headHash, theirsHash }`. Fires before three-way merge commit.
- `preCheckout` — `{ repo, target, mode }`. `mode` is `"switch" | "detach" | "create-branch"`. Fires from both `git checkout` and `git switch`.
- `prePush` — `{ repo, remote, url, refs[] }`. Fires before object transfer.
- `preFetch` — `{ repo, remote, url, refspecs, prune, tags }`. Fires before fetch.
- `preClone` — `{ repo?, repository, targetPath, bare, branch }`. Fires before clone. `repo` is optional (repo doesn't exist yet).
- `prePull` — `{ repo, remote, branch }`. Fires before pull.
- `preRebase` — `{ repo, upstream, branch }`. Fires before rebase begins.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blindmansion/just-git](https://github.com/blindmansion/just-git) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
