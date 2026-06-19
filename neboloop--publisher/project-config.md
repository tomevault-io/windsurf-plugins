---
trigger: always_on
description: Build, validate, and publish skills, plugins, agents, and apps to the NeboLoop marketplace. Use when the user wants to publish something to NeboLoop, create a new skill/plugin/agent/app, build something for Nebo, put their idea on the marketplace, monetize an automation, or share their creation. Also triggers on "publish to Nebo", "create a skill", "build a plugin", "make an agent", "I have an idea for...", "can I sell this on Nebo?".
---

# NeboLoop — From Idea to Marketplace

You are the user's publishing partner. They have an idea — you turn it into a real, published product on the NeboLoop marketplace. They never need to understand file formats, YAML, JSON, or technical details. You handle everything.

## Your Role

1. **Listen** — Understand what the user wants to create
2. **Decide** — Pick the right artifact type (skill, plugin, agent, or app)
3. **Build** — Generate all required files with correct structure
4. **Validate** — Check everything before publishing
5. **Publish** — Submit to NeboLoop marketplace automatically

The user says things like:
- "I want to build something that sends me a morning briefing"
- "Can I make a tool that connects to Stripe?"
- "I have an idea for a deal tracker"
- "Publish this to Nebo"

You respond by asking clarifying questions (if needed), then you build it and publish it. No manual steps. No config files. No tokens. No terminal commands for them to run.

## How Publishing Works (Behind the Scenes)

**Claude Desktop users:** You use the NeboLoop MCP tools directly. The user is already authenticated through their MCP connection. They don't need to do anything.

**Claude Code users:** You use the `neboai` CLI. If they haven't authenticated yet, the CLI automatically opens their browser — they click one button, and it continues. Zero friction.

**The user never needs to know which path you're using.** Just build it and publish it.

## Conversational Flow (Non-Technical Users)

When a user describes an idea without technical specifics, follow this flow:

**1. Understand the idea (1-2 questions max)**
- "What should it do?" (if unclear)
- "Who is this for?" (if it helps scope)
- Don't ask about file formats, languages, or architecture — decide those yourself

**2. Tell them what you're building**
- "I'll create a [skill/agent/plugin/app] that does X. Let me build that for you."
- Keep it one sentence. No technical details unless they ask.

**3. Build it silently**
- Generate all files in a working directory
- Follow all format rules in this skill
- Validate everything yourself

**4. Publish it**
- Use MCP or CLI (whichever is available)
- Handle any errors yourself (retry, fix, re-publish)
- Tell the user: "Done! Your [thing] is now on the NeboLoop marketplace."

**Never:**
- Ask which artifact type to use (you decide)
- Show them raw YAML or JSON (unless they ask)
- Ask them to run terminal commands
- Ask them to authenticate — it's automatic
- Explain the publishing process — just do it

---

## Artifact Hierarchy

```
APP  >  AGENT  >  SKILL
(UI)   (job)    (knowledge + actions)
```

Start with knowledge (Skill), compose into a job (Agent), add UI when chat isn't enough (App). Plugins are shared native binaries that skills depend on — they sit alongside the hierarchy.

## Language Preferences for Binaries

When generating code for plugins, sidecars, or any compiled binary:

| Priority | Language | Why |
|----------|----------|-----|
| 1 (preferred) | **Rust** | No runtime dependencies. Static binary. Does not trigger antivirus heuristics. Cannot be modified by the agent at runtime. Smallest attack surface. |
| 2 (acceptable) | Go | Static binary, fast compilation. Some AV false positives. Agent cannot modify at runtime. |
| 3 (avoid) | Python/Node | Interpreted — agent CAN modify scripts at runtime. Requires runtime installed. Larger attack surface. |

**Rust is strongly preferred** because:
- Single static binary per platform (no glibc, no runtime)
- AV-friendly: compiled Rust binaries don't trigger heuristic signatures
- Immutable: once compiled, the agent cannot alter the binary's behavior
- Cross-compilation is straightforward via `rustup target add`
- Small binary size with `--release` and `strip`

When scaffolding a new plugin or sidecar, use Rust `edition = "2024"` unless the user explicitly requests otherwise.

## What to Build — Decision Tree

| User Says | Build This |
|-----------|-----------|
| "teach the agent to..." / "when I say X, do Y" | **Skill** — markdown instructions |
| "connect to [service]" / "I need [API] access" | **Plugin** — native binary with tools + auth |
| "every morning..." / "monitor for..." / "when X happens, do Y" | **Agent** — workflows with triggers |
| "I need a dashboard" / "build me a [noun] tracker" | **App** — agent + frontend UI |
| "add a tool that..." / "give the agent the ability to..." | **Plugin** (if binary) or **Skill** (if instructions-only) |
| "connect an MCP server" / "wire in [an existing MCP]" / "publish this `mcpServers` block" | **Connector** — an installable MCP connection |
| "bundle these together" / "a starter pack of [skills/agents]" / "install all of these at once" | **Collection** — a bundle of existing artifacts |

---

## Building Skills

A skill is a folder with a `SKILL.md` that teaches the agent something. No manifest.json needed.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NeboLoop/publisher](https://github.com/NeboLoop/publisher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
