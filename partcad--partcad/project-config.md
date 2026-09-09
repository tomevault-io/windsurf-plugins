---
trigger: always_on
description: This repository contains all open source software that forms the PartCAD ecosystem. It ships **one** Python
---

# PartCAD

## Overview

This repository contains all open source software that forms the PartCAD ecosystem. It ships **one** Python
distribution, `partcad`, plus a `partcad-cli` compatibility shim; everything else here is an editor extension,
a CAD addon, or documentation.

### The packages, all inside the one wheel

* [src/partcad](./src/partcad/AGENTS.md):

  The core logic that enables maintaining digital thread for manufacturable physical products.

* [src/partcad_cli](./src/partcad_cli/AGENTS.md):

  The CLI interface to most of `partcad` functionality — the `pc` and `partcad` commands.

* [src/partcad_service_json_rpc](./src/partcad_service_json_rpc/AGENTS.md):

  A JSON-RPC service (`partcad-json-rpc` executable) exposing `partcad` functionality with methods that mirror
  the CLI. By default it runs a per-workspace background **daemon** (served over a socket / Windows named
  pipe); it can also serve over stdin/stdout or HTTP. It is the backend for `ide/vscode`, for `cad/freecad`,
  and for most `pc` commands, and the CLI manages it via `pc daemon start`/`stop`.

  The daemon owns the warm PartCAD context **and** the sandboxed Python runtimes that CAD wrappers execute in,
  so a client need not have a CAD environment at all. That is what decides whether a command runs in the client
  or on the daemon — see "Command boundary" in `src/partcad_cli/AGENTS.md`.

  **A remote daemon is never told to upgrade itself.** There is no upgrade or self-update method in the
  JSON-RPC surface and none may be added; that is a protocol rule, and it is the reason this package does not
  import `partcad_client`. Updating a *local* installation is `pc upgrade`, run by the client on its own
  machine.

* [src/partcad_utils](./src/partcad_utils):

  The lightweight pieces **every** package shares without a CAD-kernel dependency: logging, telemetry, user
  configuration — and the client/daemon rendezvous, `framing` and `workspace` (which socket serves which
  workspace, and whether anything is answering on it). The rendezvous lives here precisely because neither end
  owns it: a copy on each side is a copy that can disagree, and a disagreement is a client silently starting a
  second daemon.

* [src/partcad_client](./src/partcad_client):

  What a **client** does, and a daemon must not: discovering the daemon serving a workspace and connecting to
  it (`daemon`, `client`), replacing this installation of PartCAD (`selfupdate`), and opening a file in a
  third-party CAD application on this machine (`external`).

  All of it acts on **this machine**, from the process running out of it. A daemon can be remote, where
  "update PartCAD" would mean updating somebody else's installation and "stop the local daemons" somebody
  else's daemons; and a daemon that went looking for its neighbours would be racing every client on the
  machine. A client is one process acting on its own machine, which is what makes `pc upgrade` stopping every
  local daemon a sane thing to do rather than a distributed algorithm.

  `selfupdate` itself knows nothing even about that: a caller passes `before_install`, which `pc upgrade` uses
  to stop the local daemons and wait for them. `pc upgrade` (the host-level command; `pc update` refetches a
  package's imports and is unrelated) ends up here, as does the VS Code extension's "Update PartCAD" — by
  running `pc upgrade`. Nothing about daemons or upgrading is reimplemented in TypeScript.

  It also refuses: `pc upgrade` run inside a bundle the editor extension downloaded errors out and says to
  update the extension instead, since the extension owns that bundle.

  `external` is the same rule applied to a window instead of an installation. `pc open` (and the VS Code
  extension's per-part "Open in..." menu, by running it) starts FreeCAD on the screen of whoever ran the
  command — on this machine, with this machine's file, and never over the wire; there is no RPC method for
  opening a file and none may be added. A machine with no local installation can run the application in a
  container PartCAD keeps for it, named after the tool (`partcad-freecad`), with the workspace and the
  daemon's socket mounted at the paths they have here and the host's X display forwarded into it.

  One application in that table reads meshes and nothing else — Blender — so an object that is not already one
  is converted to STL before it is handed over. That conversion is the single thing `pc open` asks the daemon
  for, because a CAD wrapper is what does it; the window still opens here, and the registry still has no
  `open` method. Which object types are meshes is `object_types`, an inlined copy of PartCAD's own tables (a
  client must stay cheap to import) that a completeness test keeps honest.

* [src/partcad_ide_client](./src/partcad_ide_client/AGENTS.md):

  The Python side of the socket protocol `partcad` uses to display shapes in the IDE's `PartCAD Viewer`. Lazily
  imported by `partcad.viewer`, and by nothing else.

### Everything else

* [ide/vscode](./ide/vscode/AGENTS.md):

  Visual Studio Code extension for navigating through objects in a `partcad` project and UI interface to some

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [partcad/partcad](https://github.com/partcad/partcad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
