---
trigger: always_on
description: Licensed to the Apache Software Foundation (ASF) under one
---

<!--
    Licensed to the Apache Software Foundation (ASF) under one
    or more contributor license agreements.  See the NOTICE file
    distributed with this work for additional information
    regarding copyright ownership.  The ASF licenses this file
    to you under the Apache License, Version 2.0 (the
    "License"); you may not use this file except in compliance
    with the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing,
    software distributed under the License is distributed on an
    "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    KIND, either express or implied.  See the License for the
    specific language governing permissions and limitations
    under the License.
-->
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Apache Allura is an open source software "forge" (à la SourceForge/GitHub): source repo hosting (Git/SVN/Hg),
ticket tracker, wiki, discussion forums, blog, and more, organized into projects/neighborhoods. It's a Python
web app built on TurboGears 2, MongoDB (via the Ming ODM), and Solr for search — extended via a plugin
("tool") architecture.

## Repository layout

This is a multi-package monorepo. Each top-level directory is an independently `pip install`-able package:

- `Allura/` — the core platform: web framework glue, models, auth, permissions, task/event bus, base
  controllers, templates. Everything else depends on this.
- `AlluraTest/` — shared test harness/fixtures (`alluratest.controller`) used by every other package's tests.
- `Forge*/` (ForgeTracker, ForgeWiki, ForgeDiscussion, ForgeGit, ForgeSVN, ForgeBlog, ForgeChat, ForgeActivity,
  ForgeLink, ForgeShortUrl, ForgeFiles, ForgeFeedback, ForgeUserStats, ForgeImporters) — individual "tools"
  (plugins) that plug into the core platform. Each has its own `setup.py`/`pyproject.toml`, own package dir
  (lowercase, e.g. `forgetracker/`), own tests, and its own `*.egg-info`.
- `scm_config/`, `solr_config/` — config for git/http and Solr, used by the Docker dev environment.
- `scripts/` — one-off admin/migration scripts (run with the app's Python env), e.g.
  `scripts/convert_encrypted_field.py`, `scripts/migrations/`.

Sibling repos extend this one as plugins rather than living in this monorepo: SourceForge's commercial
layer `forge-classic` (dev checkout typically `/src/forge-classic`, deployed `/var/local/forge-classic`)
and its theme `sftheme` (`/src/sftheme`, `/var/local/sftheme`). This repo has zero references back to
either — the dependency runs one-way, entirely through the entry-point mechanism described below. See
"Working with sibling extension repos" further down for how to debug across that boundary.

Each Forge package registers itself with the core platform via entry points declared in its
`pyproject.toml` (older packages may still show these in `setup.cfg`/`*.egg-info`), most importantly:
```toml
[project.entry-points.allura]
Tickets = "forgetracker.tracker_main:ForgeTrackerApp"
```
To find the root controller for a tool, look up its `[project.entry-points.allura]` entry, find that
`Application` subclass, and look at its `root` attribute.

## Setup / running locally

Full install instructions are in `Allura/docs/getting_started/installation.rst` (or the Docker equivalent
`install_each_step.rst`/README) — not worth duplicating here. In short, a working system needs MongoDB, Solr,
and the Allura web app + `taskd` background worker all running. `docker-compose.yml` defines all of these
services (`web`, `taskd`, `solr`, `mongo`, `outmail`, `inmail`, `http`).

Key `.ini` config files live in `Allura/`: `development.ini` (local dev), `test.ini` (tests),
`docker-dev.ini` (docker dev), `production-docker-example.ini`.

To run the web/taskd processes in the foreground for interactive debugging (e.g. with `ipdb`):
```bash
cd Allura
gunicorn --reload --paste development.ini -b :8080     # web
paster taskd development.ini --nocapture                # taskd (background tasks/events)
```
Docker equivalents are in `Allura/docs/development/contributing.rst`.

## Tests

Tests use `pytest`. Run everything from the repo root:
```bash
./run_tests                 # all packages, in parallel, plus npm run lint-es6
./run_tests -p ForgeTracker # just one package
./run_tests --coverage      # with coverage
./run_tests -n X # number of processes to be used per suite
./run_tests -m X # number of parallel processes to be used per suite
```
Or run a single package/test directly (faster while iterating):
```bash
cd ForgeTracker && pytest
cd ForgeTracker && pytest forgetracker/tests/functional/test_root.py::TestRootController::test_new_ticket -v
```
Notes:
- `AlluraTest` always runs first (it's imported by everything else; catches syntax errors early).
- `ForgeGit` and `ForgeSVN` are NOT safe to run with `pytest-xdist` multiprocessing (`run_tests` handles
  this automatically).
- Functional/controller tests subclass `allura.tests.TestController`; its `self.app` is a WebTest app with
  `c.project` preset to the `test` project and `c.user` preset to `test-admin`. Tools under test are
  mounted at `/<entry point name>/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/allura](https://github.com/apache/allura) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
