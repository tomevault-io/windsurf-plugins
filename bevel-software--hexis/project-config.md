---
trigger: always_on
description: This is a git-backed knowledge base. You are the primary agent responsible for
---

# Knowledge base

This is a git-backed knowledge base. You are the primary agent responsible for
maintaining it.

> **This file is managed by the platform.** It lives at the repository root as
> `{{agentsFile}}`, and every server restart replaces it with the current
> template, so edits made here are overwritten. Deployment- or team-specific
> conventions belong in files of your own — anywhere under
> `{{knowledgeBaseDir}}/`, linked from wherever they are needed.

**Read `mcp-description.md` at the repository root first.** It says what this
knowledge base contains and when to consult it. Agents connected over MCP
receive the default branch's copy inline at the start of every session; a
clone like this one reads the copy on its own branch.

**There is no required format for knowledge.** Write markdown the way the
subject wants to be written: prose, tables, checklists, diagrams, whatever
serves the reader. Nothing here parses your files into a schema or rejects a
document for having the wrong shape. If a deployment layers a structured
knowledge graph on top, it brings its own conventions and its own guide; this
one describes the platform underneath, which stores files and controls who may
change them.

## Directory Structure

```text
knowledge-base/
├── {{knowledgeBaseDir}}/        ← the knowledge itself; organise it however suits you
├── {{skillsDir}}/               ← shared skills, organised by who owns them
├── {{pluginsDir}}/              ← one folder per plugin: its tools, and links to skills
├── roles.yaml            ← identity → role mapping (Admin-only edits)
└── access.md             ← repo-root access-control rules
```

(The three root names above are this deployment's own — a deployment may
rename them in its setup screen, and this guide is rendered with the names in
effect each time it is written.)

Tool paths are workspace-relative, and the workspace root holds this
repository as the `knowledge-base/` folder: a file in it is
`knowledge-base/{{knowledgeBaseDir}}/Foo.md`. Write the prefix where you can —
it is the path every tool reports back — but a path without it is PLACED under
`knowledge-base/` rather than refused, so `{{knowledgeBaseDir}}/Foo.md` names that
same file, and so does the root-anchored `/knowledge-base/{{knowledgeBaseDir}}/Foo.md`
the app's Copy path gives you. Nothing you send can land beside the repository,
where git would never see it. `.` or `..` segments, backslashes and every other
absolute path are refused.

Only those three folders are structural. `{{skillsDir}}/` holds shared skills at any
depth — the folder that holds a `SKILL.md` is the skill, and everything above
it is ownership (`{{skillsDir}}/<scope>/…/<skill>/SKILL.md`, with an `access.md` in
any scope folder that needs its own rules). `{{pluginsDir}}/` has a layout the
platform reads:

```text
{{pluginsDir}}/<Plugin>/plugin.json                  the manifest (Agent Plugins) — what makes the folder a plugin; its `name` is the plugin's identity
{{pluginsDir}}/<Plugin>/skills/<skill>/SKILL.md      a skill that lives inside the plugin
{{pluginsDir}}/<Plugin>/mcp.json                     MCP servers (authoritative)
{{pluginsDir}}/<Plugin>/software.bevel.hexis/tools/  `.tool` manuals
{{pluginsDir}}/<Plugin>/access.md                    who can read/write the plugin
{{pluginsDir}}/personal-<user-id>/…                  one per person: private
```

**The manifest's `name` is the plugin.** It is a kebab-case identifier
(`sales-team`), and it is what every grant spells (`plugin/sales-team/read`),
what the URLs and the catalog key on, and what the compiled marketplace
publishes the plugin as. `displayName` is what people see it called ("Sales
Team"); absent, the folder name is shown. Rename a plugin from its page in the
app: an identifier change rewrites every grant that names it, in one commit —
editing `name` by hand leaves those grants pointing at a plugin that no longer
exists.

**A plugin LINKS shared skills rather than containing them.** Its manifest
lists skill paths under `extensions["software.bevel.hexis"].skills` — each
entry is one skill folder or a folder of skills under `{{skillsDir}}/`:

```json
{ "extensions": { "software.bevel.hexis": { "skills": ["{{skillsDir}}/Engineering/deploy", "{{skillsDir}}/Sales"] } } }
```

One skill, stored once, can be listed by many plugins. A plugin's effective
skills are the ones inside its folder plus everything its links resolve to.
Do not edit that list by hand: linking is done from the plugin's page in the
app, because it is two edits at once — the manifest entry AND a grant on the
skill (see *Access control* below). A manifest entry without the grant lists
a skill the plugin's members cannot read; the app shows such a link as
needing setup and offers Repair.

**Ownership decides who may read a skill, never the plugin.** A shared
skill's readability comes from the `access.md` rules on its own folder and
the scopes above it. A plugin that links a skill someone cannot read simply
does not show it to them.

**Symlinks are not supported anywhere under `{{pluginsDir}}/`.** Access control
resolves rules by path, and a symlink is a second path to the same content —
the two can disagree about who may read what. The platform never creates

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Bevel-Software/Hexis](https://github.com/Bevel-Software/Hexis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
