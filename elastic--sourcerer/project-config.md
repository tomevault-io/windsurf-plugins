---
trigger: always_on
description: - `sourcerer setup [--config <file>] [--include-experimental] [CATEGORIES...]`
---

# Sourcerer

## CLI

Commands:

- `sourcerer setup [--config <file>] [--include-experimental] [CATEGORIES...]`
  (reads the config's `hosts:` to generate per-host citation skills; without `--config`, uses
  only the built-in host defaults). Valid categories: `all` (default), `agents`, `skills`,
  `tools`, `templates`, `dashboards`, `workflows`. Experimental resources live under
  `elastic/*/experimental/` and are excluded by default; pass `--include-experimental` to
  include them.
- `sourcerer index <org>/<repo> [-b <branch>] [-t <tag>] [-c <commit>]` (single-repo path
  defaults to `git.host` = `github`)
- `sourcerer index --config <file> [--prune] [--dry-run] [--no-backfill]`
- `sourcerer prune [--config <file>] [--dry-run]` (config-driven retention prune is skipped
  without `--config`; the orphan sweep always runs)
- `sourcerer mcp-proxy [-e <env>]` (run a stdio MCP proxy that forwards to the Kibana
  Agent Builder MCP endpoint; intended to be launched by Claude Desktop via `mcpServers`)
- `sourcerer help`

`sourcerer --version` (a top-level flag, not a subcommand) prints the installed version.

All commands that communicate with Elasticsearch or Kibana accept `--insecure` (flag, default
off). When set, TLS certificate verification is disabled — useful for locally-hosted clusters
with self-signed certificates that are trusted in your environment. Also configurable via the
`ALLOW_INSECURE_TLS` environment variable (set to `1` or `true`); the env var is read by all
commands including `mcp-proxy` when launched by Claude Desktop via its `env` block.

The `--config` file is `sourcerer.yml` (see `specs/sourcerer-yml.md` and `sourcerer.example.yml`
for the authoritative schema). Two optional top-level sections: `hosts:` (override/extend the
built-in git-host defaults) and `sources:` (what to index).

### Multi-host support (`git.host`)

Content is namespaced by git host as well as org/repo. Each `sources[i].git` block names a single
concrete `host`, `org`, `repo`, and `ref_type` (no wildcards or arrays). Known hosts have built-in
clone + citation URL templates in `src/sourcerer/hosts.py`; `hosts:` in the config overrides or
adds hosts. For providers whose deployments are instance-scoped (AWS CodeCommit, Azure DevOps, GCP
Secure Source Manager), define one `hosts:` entry per deployment with the region/project/instance
baked into the URL templates; `git.org` then holds only the plain org or project name.

### Indexing multiple repos with a config

`sourcerer index --config sourcerer.yml` indexes many repos, branches, and tags in one run. The
config's `sources:` is a YAML list, one entry per (host, org, repo, ref_type). See
`sourcerer.example.yml`.

#### Source fields (per `sources[i]` entry)

| Field | Required | Description |
|-------|----------|-------------|
| `git.host` | yes | Host id (a `git.host` value); a built-in or a `hosts:`-defined id |
| `git.org` | yes | Org name (may include a `+extra` segment for some providers) |
| `git.repo` | yes | Repo name |
| `git.ref_type` | yes | `branch`, `tag`, or `commit` |
| `match` | yes | For `branch`/`tag`: pattern string or list of patterns matched against ref names (version DSL + glob), a ref matches if any pattern hits. For `commit`: a commit SHA/prefix string or list of them (see below). |
| `since` | no | Index-side inclusion floor: the earliest commit to start indexing from. See below. Not valid for `git.ref_type: commit`. |
| `retain` | no | Retention policy (see below). Omit to keep forever. For `git.ref_type: commit`, only `age` is valid. |
| `mode` | no | `snapshot` (default) or `delta` (branch or tag). See below. |

#### `mode` (snapshot vs. delta)

`snapshot` (default): content is commit-addressed. A HEAD advance on a branch
indexes a whole new snapshot under the new commit.

`delta` (branch or tag; rejects `since`/`retain` -- there is no per-commit history for either
to apply to): content is ref-addressed instead. A HEAD advance runs `git diff
--name-status` between the previously-completed commit and the new tip and only deletes/
reindexes the paths git reports changed -- add/modify/delete/rename/copy -- instead of
reindexing the whole tree. A missing diff base (force-push, GC'd, or the first index) rebuilds
the whole ref namespace. The refs join doc publishes `status: indexing` before any content
change and `status: complete` (with the new commit) only after the deletes/indexes/refresh all
succeed, so a crash mid-update leaves the prior commit and content in place.

Delta mode is especially useful for fast-moving tags that are force-updated many times a day
(e.g. `deploy@8`-style Serverless promotion tags): snapshot mode would mint a fresh full snapshot
per force-update; delta mode diffs only what changed, keeping indexing cost proportional to the
diff size regardless of tag-move frequency.

```yaml
- git:
    host: github
    org: elastic
    repo: serverless-gitops
    ref_type: branch
  match: main
  mode: delta
```

#### `git.ref_type: commit` (pinning an explicit commit)

Pins one or more commits directly, rather than matching named refs. `match` entries are
7-40 hex chars - a full 40-char SHA, or a shorter prefix (git's own "short hash" convention;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elastic/sourcerer](https://github.com/elastic/sourcerer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
