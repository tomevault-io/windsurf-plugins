---
trigger: always_on
description: // Copyright © Kindel, LLC - http://www.kindel.com
---

<!--
// Copyright © Kindel, LLC - http://www.kindel.com
// Published under the MIT License - Source on GitHub: https://github.com/tig/mcec
-->

# AGENTS.md: driving (and testing) MCEC with an agent

MCEC 3.0 (the **Model Context Environment Controller**) gives an AI agent eyes, hands, and a safe
front door on Windows (Windows **computer use**, in the sense Claude, Codex, and similar agents use the
term). This file is the **self-reinforcing guidance loop**: the canonical guidance an
agent needs, plus the recipe to *dogfood* it (drive MCEC through its own MCP server) so the guidance
stays honest. Each time MCEC changes, re-run the dogfood and refine the guidance below and in
`AgentServer.Instructions`.

## The built-in guidance (single source of truth)

The connect-time guidance an MCP client shows the model is authored in
[`src/Agent/AgentInstructions.md`](src/Agent/AgentInstructions.md). It is **embedded into the exe at build
time** and returned in the MCP `initialize` response (`result.instructions`) via `AgentServer.Instructions`,
which loads the embedded file and collapses each blank-line-separated paragraph to one line.

> Work the loop **observe → target → act → observe**.
> 1. **Target** a window by `window` (title substring), `process` (name without `.exe`), `className`,
>    or `foreground:true`. At least one is required; a call with no target fails by design.
> 2. **Observe**: `query` dumps the UI Automation tree (controlType, name, automationId, bounds,
>    state, value; bounded by `maxDepth` and `maxNodes`); `capture` returns a PNG (renders composited
>    WinUI/WPF surfaces correctly). **Check results before trusting them:** a `capture` with
>    `errorCategory: capture-blank` is a black/empty frame (minimized/cloaked/occluded/locked session);
>    restore or foreground the window and retry, don't trust the image; a `capture-fallback` warning
>    means PrintWindow was refused and the picture may be wrong; a `query` with `truncated:true` (a
>    `tree-truncated` warning) hit the node cap; raise `maxNodes` or target a deeper window. `warnings`
>    are non-fatal; `errorCategory` tells you how to recover. (Shape: `docs/agent_control.md` and
>    `agent-tool-result.schema.json`.)
> 3. **Act**: prefer `invoke` (`by` name/automationId/classname; `action` invoke|toggle|setvalue|
>    setfocus|expand|collapse|select) over coordinate clicks. `invoke` **fast-fails** if the control isn't present (it does not
>    wait), so `find`/`wait-for` the control first; an `invoke` that returns `no-target` means it hasn't
>    appeared yet; `wait-for` it rather than retrying blindly. Use `select` for TabItem/ListItem/RadioButton.
>    To **drag** (resize a window by its sizing border, move one by its title bar, or drag a slider/handle;
>    no `invoke` for these), use the `drag` tool: give a `from` and `to`, each an element `{ by, value }` or
>    an absolute screen pixel `{ x, y }`, plus optional `path` waypoints; the whole press-move-release is
>    dispatched **atomically** (prefer it over hand-rolling `mouse:lbd`/`mouse:mt`/`mouse:lbu`, which can
>    interleave with other input). To **click** a point `invoke` can't reach (a custom-drawn cell, a
>    canvas/map coordinate, a bare pixel), use the `click` tool. Before firing an app's own keyboard
>    shortcut at a specific surface (e.g. a MAUI GraphicsView), `focus` it first; keystrokes only reach the
>    foreground window's focused control. `send_command` is the raw escape hatch for any other MCEC command
>    (keystrokes, a single mouse action, launch). Re-`query` after acting; bounds have moved.
> 4. **Verify** with another `query`/`capture`.
>
> **Compose creatively.** Many tasks have no single dedicated tool; build them from primitives. Launch an
> app with the `launch` tool (preferred). Use `invoke` action:select for tabs/list/radios.
> Drag/resize/move with the `drag` tool; switch a tab by `invoke` select or `click`;
> record a window by passing its `query`'d bounds as the `record` region;
> wait on window state with the `windows` tool + a `timeout` and a `condition` (appears/disappears/foreground).
> A capable agent uses the *full* command set; reach for a raw
> `send_command` before concluding something can't be done.
>
**There is exactly one copy: edit that file.** It is the observe → target → act → observe playbook
(targeting; observation with `query`/`capture`/`record`/`displays`; `invoke`, `drag`, `click`, `focus` and
`send_command`; the result
envelope; creative composition of primitives; the on-screen overlay; and the security gates). Nothing here
to keep in sync.

## Security (do not regress)

Three independent, **off-by-default** gates; see [`docs/agent_control.md`](docs/agent_control.md):
`AgentCommandsEnabled` (the observation opt-in, separate from actuation), per-command `Enabled`, and
`McpServerEnabled` (HTTP floor, localhost-bound). Every agent action is logged with an `AGENT-AUDIT:`
line. An agent that hits "agent commands are disabled" should tell the user, not retry.

Three safety features layer on top (see [`docs/safety-emergency-stop-and-provisioning.md`](docs/safety-emergency-stop-and-provisioning.md)):

- **Emergency stop**: a global "dead man's switch" hotkey (default `Ctrl+Alt+Shift+S`) the operator

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tig/mcec](https://github.com/tig/mcec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
