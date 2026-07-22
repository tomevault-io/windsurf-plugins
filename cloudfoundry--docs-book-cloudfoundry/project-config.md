---
trigger: always_on
description: Instructions for AI coding agents working on the **Cloud Foundry** documentation set. This doc set is built from multiple repositories; the book repo (this repo) assembles the TOC and references content that lives in sibling docs repos.
---

# AGENTS.md

Instructions for AI coding agents working on the **Cloud Foundry** documentation set. This doc set is built from multiple repositories; the book repo (this repo) assembles the TOC and references content that lives in sibling docs repos.

## Project overview

Product documentation for **Cloud Foundry** (OSS). Content is published at docs.cloudfoundry.org. Source is Markdown with ERB templating (`*.html.md.erb`), HTML tables, and template variables. The book is built with Bookbinder/Middleman.

**Multi-repo layout:** This repository (**docs-book-cloudfoundry**) is the **book** (aggregator). It does not hold most of the topic content. It holds:

- **Subnav/TOC** (e.g. `master_middleman/source/subnavs/_cf-subnav.erb`) that define the navigation and point at content using path prefixes such as `/buildpacks/`, `/bbr/`, `/concepts/`, etc.
- Those paths are resolved at build time from **sibling documentation repositories** (see the "Repositories in the doc set" table and the "Quick reference: TOC path → repo" table below).

When you change or add a topic that is referenced in the TOC, the actual file often lives in a content repo (e.g. **docs-buildpacks**, **docs-bbr**), not in this repo. When you rename or move a file in a content repo, you must update the TOC in **this** repo so the book still links to the correct path.

**Cross-repo change awareness:** When you make a change in any of the repos listed in this doc set, **alert the user** if that change might also require updates in other related repos—especially **docs-book-cloudfoundry** (e.g. TOC links, new or removed topics, renamed paths). Prompt them to check the other repo(s).

## Repositories in the doc set

| Repository | Role | Path prefix in TOC (examples) |
|------------|------|-------------------------------|
| **docs-book-cloudfoundry** (this repo) | Book: TOC, subnav, assembly | — |
| **docs-buildpacks** | [Buildpacks](https://github.com/cloudfoundry/docs-buildpacks) | `/buildpacks/` |
| **docs-bbr** | [BOSH Backup and Restore](https://github.com/cloudfoundry/docs-bbr) | `/bbr/` |
| **docs-deploying-cf** | [Deploying Cloud Foundry](https://github.com/cloudfoundry/docs-deploying-cf) | `/deploying/` |
| **docs-cf-admin** | [CF Admin](https://github.com/cloudfoundry/docs-cf-admin) | `/adminguide/` |
| **docs-cf-cli** | [CF CLI](https://github.com/cloudfoundry/docs-cf-cli) | `/cf-cli/` |
| **docs-cloudfoundry-concepts** | [Cloud Foundry concepts](https://github.com/cloudfoundry/docs-cloudfoundry-concepts) | `/concepts/` |
| **docs-dev-guide** | [Developer guide](https://github.com/cloudfoundry/docs-dev-guide) | `/devguide/` |
| **docs-loggregator** | [Loggregator](https://github.com/cloudfoundry/docs-loggregator) | `/loggregator/` |
| **docs-services** | [Services](https://github.com/cloudfoundry/docs-services) | `/services/` |
| **docs-uaa** | [UAA](https://github.com/cloudfoundry/docs-uaa) | `/uaa/` |
| **docs-credhub** | [CredHub](https://github.com/cloudfoundry/docs-credhub) | `/credhub/` |
| **docs-running-cf** | [Running Cloud Foundry](https://github.com/cloudfoundry/docs-running-cf) | `/running/` |

The repos listed above are the full set used in the Cloud Foundry book. The TOC in this repo is the single place that ties all content together.

**Quick reference: TOC path → repo** (all repos use **master** branch)

| TOC path | Repo |
| -------- | ---- |
| `/buildpacks/` | docs-buildpacks |
| `/bbr/` | docs-bbr |
| `/deploying/` | docs-deploying-cf |
| `/adminguide/` | docs-cf-admin |
| `/cf-cli/` | docs-cf-cli |
| `/concepts/` | docs-cloudfoundry-concepts |
| `/devguide/` | docs-dev-guide |
| `/loggregator/` | docs-loggregator |
| `/services/` | docs-services |
| `/uaa/` | docs-uaa |
| `/credhub/` | docs-credhub |
| `/running/` | docs-running-cf |

## Branch

This doc set uses a single version. All repositories use the **master** branch. Keep content and TOC in sync on **master** when making cross-repo changes.

**Before editing any file in any content repo, verify the repo is on `master`:**

```powershell
git -C <repo-path> rev-parse --abbrev-ref HEAD
```

If the result is not `master`, run `git -C <repo-path> checkout master` to switch it, then confirm the switch succeeded before proceeding. Do this for every affected repo at the start of any task, before making any edits.

**Scope all searches to the 13 listed workspace repos only.** The workspace parent folder may contain other repositories (e.g. commercial doc sets). Do not act on, report, or include results from any repo not listed in the "Repositories in the doc set" table above.

### Scripts (run from docs-book-cloudfoundry or with optional workspace root)

- **`scripts/ensure-master.ps1`** and **`scripts/ensure-master.sh`** — Switch every doc set repo to the `master` branch. Call at the start of a task so all repos are on master before edits. Optional arg: workspace root (default: parent of this repo).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudfoundry/docs-book-cloudfoundry](https://github.com/cloudfoundry/docs-book-cloudfoundry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
