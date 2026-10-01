---
trigger: always_on
description: **[English](#english) · [Español](#español)**
---

# Horizun Revit MCP — guide for agents / guía para agentes

**[English](#english) · [Español](#español)**

You are in the repository of **Horizun Revit MCP**: the MCP bridge between a
client (Claude Code, Codex, any MCP client) and an Autodesk Revit running on
this machine. Part of the [Horizun Hub](https://horizunhub.com) ecosystem.

Estás en el repositorio de **Horizun Revit MCP**: el puente MCP entre un cliente
(Claude Code, Codex, cualquier cliente MCP) y un Autodesk Revit corriendo en esta
máquina. Parte del ecosistema [Horizun Hub](https://horizunhub.com).

> **The name is written `Horizun`** — capital H, the rest lower case — every time
> it appears to a user, in any language. Never `HORIZUN`, never `horizun` as a
> word. It is a brand, not an acronym and not shouting. The only upper-case forms
> that exist are the tool names (`horizun_*`, always lower case) and the
> environment-variable prefix (`HORIZUN_REVIT_YEAR` and friends) — those are code
> identifiers, never how you refer to the product in prose.
>
> **El nombre se escribe `Horizun`** — H mayúscula, el resto en minúscula — cada
> vez que aparece ante un usuario, en cualquier idioma. Nunca `HORIZUN`, nunca
> `horizun` como palabra. Es una marca, no una sigla ni un grito. Las únicas
> formas en mayúscula que existen son los nombres de las herramientas (`horizun_*`,
> siempre en minúscula) y el prefijo de variables de entorno (`HORIZUN_REVIT_YEAR`
> y demás) — esos son identificadores de código, no cómo te refieres al producto.

---

## English

If the user asked you to install it, this is the whole procedure.

### Install the published release (default)

For “install this repo” or “install everything necessary”, use the published
installer. It includes the MCP server runtime and the add-ins; Git and the .NET
SDK are not required. Check Windows, an installed Revit 2023–2027, and that Revit
is closed first. Read the unsigned-release disclosure in README.md before using
the explicit acknowledgement below.

```powershell
powershell -ExecutionPolicy Bypass -File .\install-release.ps1 -AllowUnsigned
```

The bootstrap downloads the installer from the latest stable GitHub release,
verifies its SHA-256 against that same release, installs it and schedules safe
client registration. Report the durable installation status, including any
pending client restart or first Revit start; exit code zero alone is insufficient.
If it fails, preserve the error and installation log. Do not silently switch to
source compilation or install a development SDK to work around a release failure.

### Build from source (development or explicitly requested)

Everything is compiled from this tree, against the Revit already installed on
this machine. No executable is downloaded.

**Prerequisites** — check them first; the script checks them too:

- Windows with at least one Revit 2023–2027 installed
  (`C:\Program Files\Autodesk\Revit <year>\RevitAPI.dll` exists).
- The exact .NET SDK 10.0.400 on PATH (`dotnet --version` answers), fixed by
  `global.json` so release bytes do not depend on the latest installed patch.
  Revit ≤ 2024 still builds against .NET
  Framework 4.8 — the SDK-style projects restore the reference assemblies from
  NuGet, so the Visual Studio targeting pack is NOT required when NuGet restore
  is available. Verified on a machine without the pack: 2024 compiled with zero
  warnings. Only a fully offline machine needs the pack itself.
- **Revit closed.** The script refuses to run with Revit open and changes
  nothing when it refuses.

**The command:**

```powershell
powershell -ExecutionPolicy Bypass -File .\install.ps1
```

It detects the Revit years present, compiles the add-in for each against its own
API, compiles the MCP server, installs everything, and **verifies by reading
every installed binary back** (stamped commit + SHA-256 against what was
staged). A build failure changes nothing; a later failure rolls back through its
undo ledger and reports the exact state.

Resulting paths:

- Add-in: `%APPDATA%\Autodesk\Revit\Addins\<year>\Horizun\`
- Server: `%LOCALAPPDATA%\Programs\Horizun\MCP\server\horizun-mcp.exe`

### Configure the MCP client

**Use the EXACT path the installer printed**, already expanded for this machine.
Do not retype it with `%LOCALAPPDATA%`: `cmd.exe` expands that variable and
**PowerShell does not**, so a config written that way points somewhere that does
not exist and the client shows no tools without saying why.

Installation and registration are two internal phases but one user action. Do not
rewrite the configuration of the Claude/Codex process that is currently running:
it may overwrite the edit when it exits. The installer runs
`complete-install.ps1`, which waits for active clients to close, registers beside
existing MCP entries, verifies the configuration, and completes
`horizun_health` after Revit's first start. Report the durable state from
`%LOCALAPPDATA%\Horizun\install-status.json`. The commands below are manual
recovery only.

```powershell
# Claude Code — user scope makes it available across projects
claude mcp add --scope user horizun-revit -- "C:\Users\<you>\AppData\Local\Programs\Horizun\MCP\server\horizun-mcp.exe"

# Codex

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HorizunGroup/horizun-revit-mcp](https://github.com/HorizunGroup/horizun-revit-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
