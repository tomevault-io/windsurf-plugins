---
trigger: always_on
description: An [MCP](https://modelcontextprotocol.io) server inside the running app, so that an agent investigates the
---

# Shark Dive's agent surface — agent guide

An [MCP](https://modelcontextprotocol.io) server inside the running app, so that an agent investigates the
heap dump **in the window somebody is looking at** rather than one of its own.

This file is scoped to `shark/shark-dive/shark-dive-agent/`. Its parent,
`shark/shark-dive/AGENTS.md`, has the app-wide rules — the heap dump being read off the UI thread, a
verdict being an argument to every read — and they all apply here. This one only records what is specific to
being talked to by a program that is not this app.

## What the pieces are

| File | What it is |
| --- | --- |
| `AgentHeapDump.kt` | The seam: one open heap dump, as everything here sees it. The app implements it over a window; the tests implement it over a `HeapDive` and three fields. |
| `AgentTools.kt` | Every tool, each a name, a schema and one read. Where the refusals are. |
| `AgentPlace.kt` | Where a tab is, as one string an agent can be answered with and hand back. Both directions. |
| `AgentMethod.kt` | The method, as prose handed to the model twice. |
| `AgentJson.kt` | Shark Dive's model as JSON. |
| `AgentTool.kt` | One tool, its arguments read strictly, and `AgentRefusal`. |
| `McpSession.kt` | JSON-RPC, one message per line. |
| `AgentSessionFile.kt` | One session on disk, both ways: what a call is written as, and what it reads back as. |
| `AgentServer.kt` | The loopback socket a run publishes, and the file that says where. |
| `AgentStdioBridge.kt` | `--mcp-stdio`: the pipe an MCP client launches. |
| `AgentStdioServer.kt` | And `--no-ui`: the same tools over this process's own stdio, for a run with no window. |
| `AgentCommandLine.kt` | `--agent <tool> name=value …`: one call typed at a window, over the same socket. And `--agent-help`, generated from the registry. |
| `harness/start-harness.sh` | Opens a window and prints the command that throws an agent at it. |
| `harness/eval/run-eval.sh` | Throws an agent at a heap dump whose answer is known, and scores what it did. The dumps and the scoring are `shark-dive-eval`. |

Nothing here is public API — the module is in `modulesWithoutPublicApi`, like the rest of Shark Dive — with
two deliberate exceptions, `AgentServer`/`AgentStdioBridge`/`AgentHeapDump*` because the app calls them, and
`AgentRefusal` because the app throws it.

## The refusals are the feature

The whole point of this being a server rather than a library is that **it can say no**, and it works with any
client because saying no is all it does — nothing here ever calls a model.

- `set_verdict` refuses a blank reason (through `LeakStatusOverride`'s own `require`) and refuses a verdict
  that contradicts one already recorded unless it is told to flip it.
- `conclude` refuses until the heap dump agrees that **one** reference is at fault, and the refusal says which
  of the three reasons it is: nothing `STUCK`, nothing `EXPECTED` above it, or *these* steps in between
  with no verdict. Same rule as `faultyReferenceIndexOrNull`, read off the chain rather than asked of it,
  because the three ways it answers null are three different things to do next.
- Every tool takes a `reason`, and it is enforced in `AgentTool.call` rather than only asked for in the
  schema: a client is free to ignore a schema.

So a change that makes any of these easier to satisfy is a change that removes the reason this module exists.
An agent that has narrowed a chain to three unexplained steps must not be able to report a root cause, however
confident it is. `AgentToolsTest` walks that exact story — refused, then a verdict, then concluded — and it is
the test to keep working.

The `reason` is traceability and not a quality gate. Asking a model to explain itself does not make it right,
and [the research says it can make it worse](https://arxiv.org/abs/2504.09664); what it buys is a session log
someone can follow afterwards instead of a conclusion they have to trust.

## A session file has two readers, and neither is in this process

`AgentSessionFile` writes `~/.shark-dive/agents/sessions/agent-<when>-<id>.jsonl`, one file per
connection, a JSON object per line, the newest `KEEP_SESSION_COUNT` kept. What reads it back is **the window's
*Agent logs* screen and the eval in `notes/agent-eval.md`** — one artefact, two readers, which is why the
reading half lives here beside the writing half and is tested with it. A field written and never read back is
a row of that screen saying nothing.

What follows from that, and reading the code won't tell you:

**A call is described before it is answered, not after.** `McpSession.callTool` asks `AgentTools.target` what
the call is about and only then invokes the handler, so **a refused call still records its place** and its row
is still clickable. That is deliberate: the refusals are the half of a session worth reading afterwards, and
a refusal nobody can follow up on is a dead end on the screen. `target` derives the place from the argument
*names* rather than from a second list of tool names — except for the four tools that take no argument saying
where they are, which are named in `placeOrNull` because **every call that goes somewhere in the window has to

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [square/leakcanary](https://github.com/square/leakcanary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
