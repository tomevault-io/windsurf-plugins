---
trigger: always_on
description: Read this first. It carries the decisions, the measured facts and the working
---

# DiscWright

Read this first. It carries the decisions, the measured facts and the working
rules, so work can continue on any machine and in any session.

## What this is

A Windows app that turns a GOG offline installer into a burnable game disc: the
game's own icon and title in This PC, an autorun menu on double-click, and
optionally the disc's name and icon for a Linux desktop as well.

One file does almost all of it. `DiscWright.ps1` is a single Windows PowerShell
5.1 script with a WinForms window, an IMAPI2FS ISO builder and the menu template
inside it. That is deliberate: the README opens with "there is nothing to
install", and a folder of readable scripts is what makes that true.

Related repositories:

- `lazardjokovic/discwright-linux` - the Python port, with a GTK 4 window. **This
  repo is the reference.** When the two disagree about what a disc contains, the
  other one is wrong. The port carries its own `CLAUDE.md`, and anything about
  Linux belongs in its roadmap rather than this one.
- `lazardjokovic/discwright.com` - the website. Its `site/index.html` names the
  current version in exactly one place and has to be bumped on every release.

## Running it

```
Run DiscWright.cmd          # or the DiscWright.lnk shortcut
```

Windows PowerShell 5.1 only. The startup guard refuses PowerShell 7, not because
it is known to break but because nobody has tried it.

## Testing

```powershell
.\tests\Invoke-Tests.ps1            # logic, then the window tests
.\tests\Invoke-Tests.ps1 -SkipUI    # logic only, no desktop needed
.\tests\Invoke-Tests.ps1 -UIOnly
```

- **Pester 5, pinned deliberately.** Under 6.1.0 every file in the suite hangs in
  `BeforeAll`. The runner says so and refuses rather than hanging.
- **The window tests take the desktop.** They move the real pointer and take the
  foreground for about five minutes. `UiDriver.psm1` stops the moment something
  else owns the foreground, rather than typing into somebody's browser, so a
  leftover DiscWright window or a game in the foreground fails the whole suite
  with one clear message. Kill leftovers before blaming the code.
- **CI runs the logic half only**, because a hosted runner's 1024x768 desktop is
  smaller than the window. Whatever the window suite proves is proved locally, so
  say which is which when reporting.
- **The installer is tested in Windows Sandbox**, see `packaging/sandbox`. Smart
  App Control blocks an unsigned installer on the development machine, which is
  how 0.8.0 shipped with a Start menu shortcut that opened an error box.

Also run before any merge: the 5.1 parse check over every `.ps1`, and
PSScriptAnalyzer with `PSScriptAnalyzerSettings.psd1`, which must stay at **0
errors**. Warnings are tolerated and suppressed with a justification where the
rule is wrong for this code.

## Skills in this repository

`.claude/skills/` holds the procedures that are long, ordered and easy to get
half right:

- **`release`** - the whole of cutting a version, from the bump to the website,
  including the two steps most easily skipped: the window suite and the
  installer on a clean Windows.
- **`demo-gifs`** - re-recording the README's demonstration GIFs by driving the
  real window.

## Facts already measured, so they are not rediscovered

- **IMAPI refuses any file over 2 GiB in ISO9660.** Measured, not read off the
  specification, which is twice as generous. GOG splits its installers one byte
  under 4 GiB, so a game that arrives in parts cannot have the legacy
  filesystems.
- **UDF 2.50** is what a normal disc gets; ISO9660 and Joliet are added only when
  "readable on Windows XP and older" is ticked.
- **Joliet name limits**, measured against Windows 11: 104 characters for a file,
  103 for a folder. Not a problem here, since IMAPI writes long names into the
  ISO9660 tree anyway, and a real 96-character GOG patch name survives. It is a
  problem for the Linux port, which writes Joliet rather than UDF and refuses
  such names.
- **`autorun.inf`** is CRLF with a trailing CRLF, in the machine's ANSI codepage,
  no BOM. AutoRun has no Unicode mode.
- **`.xdg-volume-info`** is UTF-8 with no BOM and LF. A BOM makes GKeyFile read
  nothing at all.
- **Project files** are UTF-8 **with** a BOM and CRLF, because PowerShell 5.1
  reads a file without one in the ANSI codepage and mangles accented paths.
- **VBScript is not on a current Windows 11 image.** It became a Feature on
  Demand in 24H2 and Microsoft has said it will be disabled by default and then
  removed. The installer therefore picks its launcher from what the machine has.
- **The disc's menu is safe**, on the same image: `mshta.exe`, `jscript.dll` and
  `scrrun.dll` are all present, and an HTA really does create
  `Scripting.FileSystemObject`, `WScript.Shell` and `Shell.Application`.
- **Paths over 260 characters** fail during the copy. PowerShell 5.1.

## Conventions in the code

- **Every function's comment says which failure it exists to prevent**, not what
  the code does. Porting relies on that, and so does anyone changing it later.
- **The comma-return convention.** A function returning a list returns
  `,@($items)` so a one-element result stays an array. Wrapping such a call in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lazardjokovic/discwright](https://github.com/lazardjokovic/discwright) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
