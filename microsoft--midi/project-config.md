---
trigger: always_on
description: Windows MIDI Services. See [README.md](README.md) for what the project is, and
---

# Instructions for coding agents

Windows MIDI Services. See [README.md](README.md) for what the project is, and
[CONTRIBUTING.md](CONTRIBUTING.md) for contribution process.

## This repository is public

You may be given access to non-public source or documents to investigate a problem. Treat them as
input to your decisions only. **Nothing from them may appear in this repository, or in issues,
pull requests or commit messages** — not file or symbol names, not internal interfaces or keys, and
not the mechanism behind a behavior. Describing externally observable behavior is fine; explaining
how it is implemented internally is not. If a document seems to need the internal mechanism to make
sense, rewrite it in terms of what a customer can observe.

## The configuration file is off limits

Windows MIDI Services keeps its configuration in `%ALLUSERSPROFILE%\Microsoft\MIDI` in a
`.midiconfig.json` file named by a registry value. **The JSON schema, the file name, the folder and
the registry value are implementation details, not a contract.** They can change in any release,
without notice.

There are exactly two supported ways to read or change that configuration:

- **The Windows MIDI Services API** — the `Windows.Devices.Midi2.ServiceConfig` namespace and the
  per-transport config types. It serializes access with the service, merges entries instead of
  rewriting the file, makes its own backups, validates what it writes, and applies the change to the
  running service as well as to disk.
- **The in-box tools built in this repository** — the MIDI Settings app, the `midi` console and the
  PowerShell module — which call that same API.

Everything else is unsupported:

- **Do not write, ship, publish or recommend a script or tool that opens, parses, edits, merges,
  backs up or restores the configuration file directly.** Care taken does not make it supported.
  Hash checks, exclusive file handles, before-images and temp-file swaps still race the service and
  the Settings app, still leave debris in `ProgramData`, and still break the day the format changes.
- **No one outside Microsoft may ship a product that manipulates the configuration file directly**,
  for any reason, including backup, restore, migration or "personalization recovery".
- If the API cannot express what is needed, that is an API gap. Say so, and file an issue for it.
  **Do not fall back to the file.** An unfinished feature is a better outcome than a tool that
  corrupts a customer's MIDI configuration after the next update.

### Developer-only tools inside this repository

A tool in this repository that must touch the file directly — a test fixture, a diagnostic, a repair
tool — is allowed, but it has to carry this disclaimer **in the source and in any documentation for
it**, so that neither a human nor another agent learns the pattern from it:

> **UNSUPPORTED — DO NOT COPY THIS APPROACH.** This code reads and writes the Windows MIDI Services
> configuration file directly. Microsoft does not support direct manipulation of that file by
> anything other than the in-box MIDI tools. Its format, its name, its location and the registry
> value that selects it are implementation details and can change in any release without notice, so
> this code can corrupt a configuration or stop working at any time. It exists for development and
> diagnostics inside the Windows MIDI Services repository only. Applications and third-party tools
> must use the Windows MIDI Services API (`Windows.Devices.Midi2.ServiceConfig`) instead.

## Writing

Everything in this repository is authored in **en-US**, including identifiers, comments, UI strings
and documentation. en-GB spellings are defects, not style preferences. Full rules and word list:
[.github/instructions/en-us-spelling.instructions.md](.github/instructions/en-us-spelling.instructions.md).
Check your work with `build/check_en_us_spelling.ps1 -Path <file or folder>`.

**Never hard-wrap markdown.** A paragraph is one line, however long. So is a bullet, including its continuation text, and so is a table row. Put newlines only between blocks — between paragraphs, between bullets, and around headings and code fences. Code fences and front matter keep their own line breaks. There is no column limit. This applies to every markdown file here, to GitHub issues and pull request descriptions, and to reports filed against other repositories.

**Public documentation is written for an eighth grader.** Short sentences, everyday words, and the plain term rather than the clever one. Contractions are welcome. Explain why a setting matters, not just what it is.

**Leave the jargon out.** That includes the figures of speech agents reach for by reflex: *load-bearing*, *smoking gun*, *deep dive*, *surface it*, *unpack*, *low-hanging fruit*, *rabbit hole*, *move the needle*. Write what actually happens instead — "removing this breaks every published link", "this is the proof", "show it in the UI".


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/MIDI](https://github.com/microsoft/MIDI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
