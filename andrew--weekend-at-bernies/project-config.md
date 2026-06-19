---
trigger: always_on
description: Assess whether a given repository is a "bernie", a package that is still depended on but no longer actively maintained, and produce a remediation recommendation covering maintainer state and security posture. Use when reviewing a dependency, auditing a project's risk, or evaluating an open-source library before adopting it.
---


# bernie-check

Tell whether a repository is a bernie (effectively unmaintained, still propped up in production) and what a dependent should do about it. Pulls fresh data from ecosyste.ms and the OSSF Scorecard API; does not depend on any local database.

A bernie classification by itself is not the answer. The remediation a dependent should take depends on who owns the package, whether that owner is still around, what the package's shape is, and what the project's published security posture looks like. This skill bundles all of those into one assessment.

## Inputs

Either a repository URL passed as argument, or auto-detect from the current git working directory's `origin` remote.

```bash
# Explicit
TARGET="https://github.com/foo/bar"

# Auto-detect (run from inside a git repo)
TARGET=$(git remote get-url origin 2>/dev/null | sed -E 's#^git@github.com:#https://github.com/#; s#\.git$##')
```

Bail out early if the URL can't be derived or isn't a recognised git host. ecosyste.ms covers github.com, gitlab.com, bitbucket.org, gitlab.gnome.org, codeberg.org and several smaller hosts; non-github hosts may have thinner data.

Extract `OWNER` and `REPO` for later GitHub-specific calls:

```bash
read -r HOST OWNER REPO <<<"$(echo "$TARGET" | sed -E 's#^https?://##' | awk -F'/' '{print $1, $2, $3}')"
```

## Step 1: Fetch core data

All ecosyste.ms endpoints return JSON. Use curl with `-L` (the `/lookup` endpoints return 302 redirects to the canonical resource path) and a User-Agent that includes a contact email (ecosyste.ms asks for this in their TOS).

```bash
UA='bernie-check (your-contact@example.com)'

# Repository metadata (status, archived, pushed_at, metadata.files map, scorecard, owner_url)
REPO_JSON=$(curl -sLH "User-Agent: $UA" \
  "https://repos.ecosyste.ms/api/v1/repositories/lookup?url=$TARGET")

# Commit stats (note the field name is past_year_total_commits, not past_year_commits)
COMMITS_JSON=$(curl -sLH "User-Agent: $UA" \
  "https://commits.ecosyste.ms/api/v1/repositories/lookup?url=$TARGET")

# Issue/PR stats (active_maintainers is an array; take .size for the count)
ISSUES_JSON=$(curl -sLH "User-Agent: $UA" \
  "https://issues.ecosyste.ms/api/v1/repositories/lookup?url=$TARGET")

# Packages published from this repo (returns an array, one row per registry+name)
PACKAGES_JSON=$(curl -sLH "User-Agent: $UA" \
  "https://packages.ecosyste.ms/api/v1/packages/lookup?repository_url=$TARGET")
```

A 404 or empty body on any of these means the service hasn't indexed this repo yet. Fall back to whatever signals you have; do not treat absence as evidence of inactivity. Trigger a re-sync by hitting the `/lookup` endpoint once, ecosyste.ms processes the queue asynchronously, so a re-run in a day or two may fill in the gap.

## Step 2: Classify the repo

Apply the same logic as the local classify.rb. Thresholds:

  * `ACTIVE_COMMITS_PER_YEAR = 12`: twelve human commits in the past year is the floor for "active"
  * `STALE_RELEASE_DAYS = 365`: a release within the last year keeps the repo "active" regardless of commit count

Compute these fields from the responses (exact field names):

  * `days_since_release`: from `MAX(packages[].latest_release_at)` in `PACKAGES_JSON`, or fall back to `repo.pushed_at`
  * `days_since_push`: from `repo.pushed_at`
  * `human_commits`: `commits.past_year_total_commits - commits.past_year_total_bot_commits`
  * `active_maint`: `issues.active_maintainers.size` (the field is an array, not a count)
  * `closed`: `issues.past_year_issues_closed_count + issues.past_year_pull_requests_closed_count`
  * `merged`: `issues.past_year_merged_pull_requests_count`
  * `asked`: true if `issues.past_year_issues_count + issues.past_year_pull_requests_count > 0`
  * `archived`: `repo.archived == true`

Decision:

```
if repo.archived:
  bucket = "dead"
elif asked and no signs of life (no commits, no closes, no merges, no recent release):
  bucket = "dead"          # someone knocked, nobody answered
elif any signs of life:
  if human_commits >= 12 OR recent release:
    bucket = "active"
  else:
    bucket = "dormant"
elif recent commit OR recent push:
  bucket = "active"
else:
  bucket = "unknown"       # nothing happened either way; quiet but untested
```

Record the signals that drove the decision so the call can be argued over: `archived`, `commit:Nd`, `release:Nd`, `commits:N`, `active_maint:N`, `closed:N`, `merged:N`. The classifier is permissive: zero commits alone is never sufficient to call something "dead". You need evidence that someone tried to engage and nobody responded.

A repo is a **bernie** if `bucket in ("dead", "dormant")`. If the bucket is `active` or `unknown`, stop here and report accordingly: active means no remediation needed; unknown means there's not enough data yet.

## Step 3: Owner state (only for bernies)

Pull the owner record and decide whether you're looking at an individual or an org. The action paths diverge.

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andrew/weekend-at-bernies](https://github.com/andrew/weekend-at-bernies) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
