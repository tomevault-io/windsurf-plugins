---
trigger: always_on
description: **Project Vision:** TangleClaw is an open-source, local-first AI-native SDLC orchestration platform. We are building the execution and control plane for fleets of specialized AI agents (Project Managers, Architects, Builders) across the software development lifecycle.
---

# CLAUDE.md

**Project Vision:** TangleClaw is an open-source, local-first AI-native SDLC orchestration platform. We are building the execution and control plane for fleets of specialized AI agents (Project Managers, Architects, Builders) across the software development lifecycle.
<!-- This repo is governed by the Prawduct Claude Code plugin — see "Governance (Prawduct)"
below, under the PRAWDUCT:ANCHOR marker.

Three owners, and the markers are the boundaries. Everything ABOVE the BEGIN:tangleclaw
marker is hand-maintained and safe to edit, EXCEPT the "Governance (Prawduct)" section under
PRAWDUCT:ANCHOR — that one belongs to the plugin and its doctor check grades on it. Everything
between the BEGIN:tangleclaw and END:tangleclaw markers is written by TangleClaw and replaced
on its next write, so do not hand-edit inside it.

Do not spell either marker in full anywhere else in this file. TangleClaw counts occurrences
before splicing, and a second literal makes the count read as malformed — it then refuses to
write and its block silently freezes at whatever it last contained.

The "Global Rules" section below is a hand-maintained MIRROR of data/global-rules.md, the
file TC injects into every OTHER (non-plugin-governed) project's config. The two are pinned
equal by test/repo-governance-reference.test.js, so edit data/global-rules.md and copy it
here, never one alone. Where a mirrored rule does not hold for THIS repo, the exceptions
section immediately above it is the authority. -->

## Core Rules (Enforced)

- Update CHANGELOG.md with every change
- All functions must have JSDoc comments
- Write tests alongside implementation
- Follow session wrap protocol before ending
- All port assignments go through PortHub

## Extension Rules

- Update docs in same commit as code changes
- Use decision framework before adding code
- Independent Critic review after medium+ work

## This Repo's Exceptions To The Global Rules Below

The Global Rules section that follows is TangleClaw's guidance for the projects it manages, mirrored
here. TangleClaw is itself one of those managed projects, so the rules apply — except where this
repo's own tooling makes them wrong. Each exception below is the authority for this repo; the
mirrored rule stays as written because it is correct everywhere else.

- **Do not tag releases by hand.** The Releases & Versioning rule says to suggest
  `git tag -a vX.Y.Z && git push --tags` after a substantive merge. Not here: `.github/workflows/release.yml`
  creates the tag and publishes the Release as separate steps, so a hand-made tag races the workflow
  and can pin a commit whose `version.json` disagrees with the tag. `docs/release-process.md` is the
  authority on how a release is cut here, and it cites this exception.
- **The post-commit hook is opt-in**, and is not the tagger — see `docs/release-process.md`.

# Global Rules

These rules apply to all TangleClaw-managed projects, across all engines. Edit them from the TangleClaw landing page or via the API.

## General

- Follow the project's existing code style and conventions.
- Prefer small, focused commits over large monolithic ones.
- Keep functions short and single-purpose.
- Write commit messages that explain *why*, not just what.

## Build Plans & Chunks

Some engines store plans in their own global directory (Claude Code uses `~/.claude/plans/`), so plans get lost across sessions and collide between projects.

**Rule: Keep plans local to the project, in TangleClaw's directory — not an engine's.**
- After creating/updating a plan in plan mode, copy it to `<project-root>/.tangleclaw/plans/<name>.md`.
- The location is deliberately engine-neutral: a plan is project state, not engine state, so a project that switches engines keeps its plans. TangleClaw's wrap reads `.tangleclaw/plans/` first and still falls back to a legacy `.claude/plans/` directory where one exists, so existing projects keep working — but new plans go in the TangleClaw location.
- Memory entries and handoffs must reference the project-local copy by **absolute path** — never ambiguous relative paths like `.tangleclaw/plans/...`.
- Don't rely on an engine's global plans directory as the cross-session source of truth.

**Rule: Make plans and design docs openable from anywhere, not just as a local file path.** A local file path can't be opened from another machine, and the operator often reads on a different device.
- **To the OPERATOR:** When you present a substantial plan, design doc, or reference deliverable to the operator, hand them the **shareable hosted link**, never a bare local file path. Published links must use the MagicDNS format (e.g. `https://cursatory.tail123678.ts.net:8443/plans/<projectId>/<file>.md`). Never use engine-hosted artifact links (like claude.ai artifacts); if the operator clicks it, it must be MagicDNS.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jason-Vaughan/TangleClaw](https://github.com/Jason-Vaughan/TangleClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
