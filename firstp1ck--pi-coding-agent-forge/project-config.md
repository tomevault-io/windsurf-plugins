---
trigger: always_on
description: This file applies to the whole repository. A more specific `AGENTS.md` in a package directory may add package-specific rules, but it must not weaken the documentation layers defined here.
---

# Repository documentation rules

This file applies to the whole repository. A more specific `AGENTS.md` in a package directory may add package-specific rules, but it must not weaken the documentation layers defined here.

## Documentation goal

Write documentation for the person reading it:

- `README.md` is the friendly starting point for users.
- `TECHNICAL.md` is an advanced user reference.
- `DEVELOPMENT.md` is for contributors and implementation details.

Do not put every fact in the README. Do not hide essential usage or safety information in a developer document. Move information between layers instead of deleting it.

## Required documentation layers

### `README.md` — human guide

The README answers:

1. What is this package or skill?
2. Why would I use it?
3. What can it do?
4. How do I install it?
5. How do I use it for the first time?
6. Is there anything important to configure or keep safe?
7. Where can I learn more?

Use plain language, short sections, and realistic examples. Keep exact command names, but explain what each important command does.

The README may contain:

- a one-sentence purpose;
- user-facing features;
- installation instructions;
- a practical first-use flow;
- common commands or example requests;
- essential requirements;
- prominent safety and privacy warnings;
- links to `TECHNICAL.md` when advanced user information exists.

The README must not contain:

- internal API endpoints;
- request or response payloads;
- schemas or protocol details;
- internal algorithms or architecture;
- source-file maps;
- contributor test commands and fixtures (user-facing health checks are allowed);
- benchmark implementation methods;
- repository contribution or package-publication internals. User-facing release steps remain allowed when release management is the package’s purpose.

### `TECHNICAL.md` — advanced user reference

`TECHNICAL.md` is still user documentation. It is not a developer dump.

It may contain:

- complete user commands and options;
- user-editable settings and environment variables;
- runtime requirements and supported platforms;
- storage and configuration locations users may need;
- compatibility and operational limitations;
- security and privacy behavior;
- safe update, migration, and rollback behavior;
- troubleshooting guidance;
- links to `DEVELOPMENT.md` for implementation details.

It must not contain:

- HTTP/API/RPC endpoint catalogs;
- request, response, or event payload formats;
- tool schemas or protocol contracts;
- internal state machines, algorithms, hash construction, or locking design;
- the package’s implementation source layout or implementation-specific file names;
- development setup, local linking, or repository package-publishing internals;
- test suites, fixtures, benchmarks, or contributor validation commands.

An API key, service address, user-visible confirmation hash, or health-check command may appear in `TECHNICAL.md` when an end user must configure, verify, or approve it. The package’s internal calls, hash construction, and test implementation belong in `DEVELOPMENT.md`.

### `DEVELOPMENT.md` — contributor guide

Create `DEVELOPMENT.md` only when the package has contributor or implementation information to preserve.

It is the correct place for:

- API and RPC endpoints;
- payloads, schemas, protocols, and event contracts;
- architecture and internal control flow;
- source layout and important implementation files;
- internal algorithms, hashes, locks, queues, caches, and storage formats;
- tool contracts intended for integrators;
- development installation and local linking;
- contributor tests, fixtures, benchmarks, and validation commands;
- repository maintenance and package-publication internals;
- contributor-only migration notes.

Start it with navigation back to the user documents:

```markdown
# Development guide: Package name

Contributor-only implementation, API, architecture, testing, and maintenance information.

[Back to README](README.md) · [Advanced user technical reference](TECHNICAL.md)
```

## Package and extension README structure

Use this structure for top-level `pi-extension-*` and `pi-package-*` packages:

````markdown
# Friendly package name

One sentence explaining the user outcome.

## What you can do

- Three to five specific user-facing features

## Install

```bash
pi install npm:@firstpick/package-name
```

## How to use it

A short first-use flow, followed by the most useful commands or examples.

## Before you start

Only essential setup, safety, or privacy information. Omit this section when it adds no value.

## Technical details

See [TECHNICAL.md](TECHNICAL.md) for complete commands, configuration, compatibility, security, and troubleshooting information.
````

Adapt headings when a package has a better user-facing name, such as `Start it`, `Keep it private`, or `Remote access`. Do not remove the purpose, features, installation, practical usage, or technical-reference link.

## Skill README structure

Use this structure for top-level `pi-skill-*` packages:

````markdown
# Friendly skill name

One sentence describing the outcome.

## Helpful when


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Firstp1ck/pi-coding-agent-forge](https://github.com/Firstp1ck/pi-coding-agent-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
