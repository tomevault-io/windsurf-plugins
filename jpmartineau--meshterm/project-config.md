---
trigger: always_on
description: MeshTerm is a full-featured TUI MeshCore client for your terminal. The UI is a
---

# MeshTerm — working notes for Claude

MeshTerm is a full-featured TUI MeshCore client for your terminal. The UI is a
prompt_toolkit session rendering Rich content (`meshterm/ui/`), tools register into a
menu (`meshterm/tools/`), services run in the background (`meshterm/services/`), and
everything heard is recorded to SQLite (`meshterm/persistence/`).

Run the tests with `python -m pytest -q`. Screens must stay readable at each platform's
`readable_cols` — 72 regular, 53 PicoCalc (see **Platforms** below); the dual-platform
gallery test (`tests/test_gallery.py`) is the enforcement point, and every picocalc case
is a hard gate.

## Git workflow

Commit directly on `main` — do not create topic branches unless explicitly asked (this
is a solo repo; branches just create divergence to merge back later). Commit freely, but
never `git push` unless explicitly asked in that moment.

## Licensing — what every file and every build carries

- **Every Python file opens with `# SPDX-License-Identifier: Apache-2.0`** (line 2 after a
  shebang). `tests/test_spdx.py` is the gate. Anything under another licence lives beside
  its own text and says so in its header — the GPL-2.0-only console-font script, the
  MeshCore MIT notice beside the XIAO patch — and never enters the distributed package.
- **The map credits OpenStreetMap in the map's own bottom-right corner**
  (`ui/attribution.py`): the full OpenFreeMap line on arrival, `© OpenStreetMap` after the
  first keystroke, `muted`, on both platforms and in the same words. *Which* corner is the
  frame's, not the drawing's, wherever the frame has one — **regular** sets the credit into
  the panel's **bottom border rule**, right-justified, one rule cell before the corner, the
  way a title sits in the top rule (`Screen.bottom_caption` → `frame._panel_box`), so the
  map's cells are untouched; **picocalc**, whose frame has no bottom rule at all, stamps it
  over the right end of the drawing's last row instead, overprinting what it covers. The
  minimap always stamps. The split is bound once via `platforms.on_platform`, never asked
  per frame. Never a title atom, never a footer character, never a key. The licence and URL
  live on the About page.
- The name and the wordmark are trademarks reserved in `NOTICE`, which Apache §4(d) makes
  every fork carry; `NOTICE` holds only what must travel with a redistribution.

## UX standards

These are binding. Every new screen, dialog, row, or hint follows them; when you touch an
old one that doesn't, bring it along. The enforcement points live in code — build through
them instead of hand-rolling:

- `ui/menus.py` — `exit_rows`, `menu_rows`, `lane_row`, `section_heading`,
  `confirm_discard`, `fit_cells`.
- `ui/markdown.py` — `render_markdown` (THE prose renderer: a page of writing, drawn in
  the language below).
- `ui/widgets.py` — `highlighted_hash` (THE key widget — shows a key, lights its hash),
  `format_ago` (prose ages),
  `format_age` (column ages), `channel_glyph`, `NODE_GLYPHS`, heard-age heat colouring.
- `ui/theme.py` — `name_style`/`node_style` (per-node hues, hash-derived), `snr_style`,
  the `you` white.

### Lexicon — one term per concept

| Term | Meaning |
|---|---|
| node | any device on the mesh — a radio broadcasting packets; roles: companion, repeater, room server, sensor. The umbrella term |
| contact | a node your device knows: **discovered** (heard broadcasting, not yet added) or **added** (in the contact list, messageable). Every contact is a node; not every node is a contact |
| heard | received from ("last heard", "first heard") — never "seen" in UX text |
| key | the full fixed-length value — a node's public key, a channel secret |
| hash | the short derived id — a key's first path-hash-mode bytes (the slice `highlighted_hash` lights), a channel hash, a path hop |
| path | an ordered hop spec you compose or force (`a1,3d,…`) |
| route | the concrete node sequence a trace walked or will walk |
| via | prefix for a packet/message's relay chain |
| Back | leave the current screen/list — Esc's word, and a row's only where leaving is a *choice* (see below) |
| Quit | leave the app (main menu, device splash) — nowhere else |

Node vs contact — the boundary: **node** is the hardware/participant sense — the map,
mesh walk, heard-nodes, relay hops, graph vertices, and node *types* (companion/repeater/room server/
sensor) all speak "node". **Contact** is the saved-identity sense — the Contacts screen,
the courier recipient, anything you *address*. The reception/persistence layer
(`observations.node`, `HeardNode`, `heard_nodes()`, `trace_hops.node`) stays "node"; the
sortable list you pick from is `contactlist.py` (`ContactListScreen`/`ContactRow`/
`ContactsSort`/`contacts_table`). The `contacts` tool lists the device's added contacts.

Setting vs preference — the other boundary, three-way and never blurred: a **setting** is
the *radio's* (`core/device_config.py`, read live from the companion, edited on Device
config); a **preference** is *MeshTerm's own behaviour* (`core/preferences.py`, defaults in
code, overrides in `preferences.toml`, edited on the Preferences page — `ctx.preferences`,
or `preferences.current()` where there is no context to reach through); and **config** is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jpmartineau/MeshTerm](https://github.com/jpmartineau/MeshTerm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
