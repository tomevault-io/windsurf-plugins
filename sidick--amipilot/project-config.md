---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

AmiPilot is an object-level GUI automation system for classic AmigaOS: an
on-Amiga automation server plus a host-side Python client that drive real
Amiga GUIs semantically (find a window/gadget by ID, label, or role; act
with genuinely synthesised input; assert on state) rather than by pixel
coordinates or screen scraping. The full design, locator-tier model, wire
protocol, and phase sequencing live in `docs/implementation-plan.md` —
read that before making architectural decisions; this file only covers
what's needed to build and navigate the code day to day.

**Current state:** v1.1 released (v1.0 was phase 1.0 complete — the
first full release; v1.1 closes every gap v1.0 itself named as open —
requesters, pointer-only menu items, `layout.gadget`-nested children —
and adds `PICK`, interactive pick-mode discovery). The cooperative
geometry port is real too (issue
#49, shipped in v1.1): the escape hatch for gadgets nested
inside a `window.class` window's `layout.gadget`, permanently invisible
to structural walking on classic AmigaOS 3.x. A manifest gains a
version-2 record pair, `WHEREPORT <port-name>` and `WHEREGADGET
<logical-name> <window-name>`, naming a small optional ARexx port the
*application itself* exposes, answering `WHERE <name>` with its own
live `GetAttr(GA_Left/GA_Top/GA_Width/GA_Height)` geometry;
`CLICK`/`TYPE @name` then act on that geometry with a genuine
`input.device` click (`AmipClickWindowRelative()`,
`server/src/action.c`) exactly as they would for a plain `GADGET` name
-- discovery is cooperative, but actuation stays real input, unlike
`MUIREXX`, where the target's own port does the acting too. The
standalone `WHERE` verb (`server/src/where.c`) is a diagnostic probe:
`GETTEXT`/`DRAG` have no path through a `WHEREPORT` at all, an honest
stated limit (RC 10, "geometry only") rather than a silent fallback.
See `manifest/SPEC.md`'s "The cooperative geometry port" section for
the full wire contract, including its "Clash guard" note on why
`WHEREPORT` resolution is exact-match only (no `MUIREXX`-style `.1`
fallback) and why applications should use a dedicated port name.
Verified end to end against `fixtures/classact-app`'s own new
`CAAPP.WHERE` port (`tests/copperline/run.sh`'s `run_where_check`,
`tests/copperline/where-test.py`) -- all three of that fixture's
gadgets, previously named by nothing at all (its manifest deliberately
named zero gadgets as the honest example of this exact limit), are now
addressed purely via `WHEREGADGET`; `CAApp.golden` is unchanged,
confirming the walker's own view of the window genuinely didn't change.
A real bug found building this, live (2026-08-09): a `RexxMsg`
constructed by hand via `CreateRexxMsg()`/`FillRexxMsg()`/`PutMsg()` --
the same recipe `MUIREXX`'s own `AmipMuiRexxSend()` (`server/src/
muirexx.c`) already used successfully against real MUI-Demo -- arrives
at the receiver with its node type left at `NT_MESSAGE`, not
`NT_REPLYMSG`; `rexxsyslib.library`'s own `IsRexxMsg()` reports such a
message as not a genuine `RexxMsg` at all despite every other field
being correct. Two attempted sender-side fixes (pre-marking
`ln_Type = NT_REPLYMSG` before sending; using a genuine
`CreateArgstring()`-backed `rm_Args[0]`) changed nothing. Root cause:
`IsRexxMsg()` checks `ln_Type == NT_REPLYMSG`, which `ReplyMsg()`
sets -- the right check for a *sender* inspecting its own reply, not
a *receiver* validating an incoming, not-yet-answered command, which
is legitimately still `NT_MESSAGE` regardless of how the sender is
built. The field that actually means "this is a command invocation"
is `RXCOMM` itself (already set correctly by every sender here) --
fixed by having `CAAPP.WHERE` validate incoming messages with
`rm_Action & RXCOMM` instead of `IsRexxMsg()`, see the doc comment on
`HandleWhereMessage()` in `fixtures/classact-app/src/main.c`. A
genuinely standard, ordinary ARexx port -- nothing dedicated-port-only
about this fix, any third-party implementer (e.g. AmiAuth) needs only
the same one-condition change.

`MENUPICK`'s pointer-based fallback for shortcut-less items is real
too now (issue #63, shipped in v1.1): previously,
`MENUPICK` only worked for items with a real keyboard shortcut
(`AmipMenuPickByShortcut()`); an item with none was rejected outright.
`AmipMenuPickByPointer()` (`server/src/action.c`) now drives the
pointer path automatically for such items -- a genuine synthesized
RMB-down/move/move/RMB-up sequence, the same "real input.device
events, not a shortcut" principle `CLICK`/`DRAG` already follow, not a
forged `IDCMP_MENUPICK` message (considered and rejected: it would
bypass real IDCMP/activation/`WFLG_RMBTRAP`/`IDCMP_MENUVERIFY`
handling entirely, a false-positive risk this project's own design
principle exists to prevent). Two real, non-obvious things had to be
found live (2026-08-09) before this worked: (1) RMB-down alone only
switches the screen's own title bar into menu mode -- the pulldown
itself doesn't open until the pointer is actually moved onto the
target menu's own title text, confirmed only after a plain RMB-down

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sidick/amipilot](https://github.com/sidick/amipilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-15 -->
