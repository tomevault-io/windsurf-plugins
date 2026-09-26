---
trigger: always_on
description: Operational rules for this repository. Each one is here because ignoring it cost a
---

# Working on Bonfire

Operational rules for this repository. Each one is here because ignoring it cost a
release, and most of them now have a check behind them — the rule and the check are
listed together so it is obvious which ones are still only a promise.

## Run this before you push

```
tests/run.sh          # or tests\run.ps1 on Windows
```

Builds the plugin (Release, `-warnaserror`) and runs all 37 harnesses — 28 JavaScript
and 9 C#, about 1,500 assertions. CI runs the same command, so the desk and the pipeline
cannot disagree about what "the tests pass" means.

```
tests/run.sh cs10     # the C# harnesses again, against the .NET 10 build
```

The plugin multi-targets `net9.0;net10.0`, because Jellyfin 10.11.x runs on .NET 9 and
12.0 runs on .NET 10. **Only `net9.0` ships.** It is the one of the two that loads on
*both* servers — the .NET 10 runtime accepts a net9 assembly, which is what every release
since 1.6.0.1 has been and is confirmed on a live 12.0 server — so one artefact and one
manifest entry covers everybody. A net10.0 artefact would install on 12.0 and silently
fail to load on 10.11, and `targetAbi` cannot express the difference because it filters on
the Jellyfin version, not the runtime.

`net10.0` is built and *run* anyway, so it cannot rot between now and the day .NET 9
support is dropped — a build nobody has loaded is not a working build. Change the
packaging step in `release.yml` to net10.0 only when 10.11.x support is being dropped, and
bump `targetAbi` in the same commit. The source already compiles clean against the
Jellyfin 12.0.0 packages with `-warnaserror`, so that day is a two-line change.

*Check:* `tests/js/buildtargets.js` fails if the csproj stops building either framework, if
the packaging step names anything but net9.0, if either workflow stops installing either
SDK, or if any C# harness pins itself to one framework.

A `*.scan.js` file is a survey, not a gate: it prints what it finds and always exits 0,
so the runner skips it alongside `*.verify.js`. Counting one would add a harness that can
never go red.

`node --check Web/profiles.js` is **necessary and not sufficient.** It passed against the
defect that made 1.5.2 and 1.5.3-beta dead on arrival: a stray backtick inside the
stylesheet template literal produced valid JavaScript that threw on the first call.
`tests/js/inject.test.js` is what catches that class of bug, because it executes the
startup path instead of reading it.

It is also the wrong engine. `node --check` runs today's V8, where `?.` parses happily;
webOS 5 is Chromium 68 and one token it cannot parse takes the **whole file** down — no
gate, no switcher, no message. `tests/js/jsbaseline.js` holds the floor, and its first
four assertions are the detector proving it can go red.

## The rules

**Never edit `Web/profiles.js` through a shell heredoc.** Eleven recorded ways this has
silently produced broken or misplaced code — shell mangling of backslashes and `$`,
CRLF/LF mismatches making a pattern match zero times, regexes eaten by the shell. Use an
editor tool, or write the script to a file first and run it. This bit again while writing
the plan that produced these rules.

**Read the jellyfin-web component before touching an injection selector.**
Do not infer markup from memory or from how it looked in an older release. Fetch it:

```
curl -sL https://raw.githubusercontent.com/jellyfin/jellyfin-web/release-10.11.z/<path>
```

*Check:* `tests/js/selectors.test.js` fails on any selector used by an injection function
that is not recorded in `tests/upstream-selectors.json`, with the file it was verified in.
`tests/js/selectors.verify.js` re-checks that ledger against upstream (needs network, not
part of CI). Twelve dead selectors have been removed from the code; one is left and
tracked.

**The ledger is only as good as its matcher, and it has been wrong twice.**
`selectors.verify.js` grepped for the token as a substring, so `.navMenu` "verified"
against `navMenuOption` — an entry whose own note said it was not a container class sat
marked `upstream` for weeks. And `selectors.test.js` read one string literal per query
call, so a selector written as `'a, b, ' + 'c, d'` had half of it silently unexamined.
Both are fixed. If you add a way to write a selector, check the extractor still sees it.

**A new harness must FAIL against the build carrying the bug before it is allowed to
pass.** Every JS harness takes the source path as `argv[2]` precisely so it can be pointed
at an older checkout:

```
node tests/js/inject.test.js /path/to/old/profiles.js
```

This is the only thing that distinguishes a test from a restatement of the code.
`session.js` was green for three releases while the fix it covered had never once
executed, because it modelled a sequence no browser produces.

**Distrust any check that computes an aggregate, or that answers a coarser question than
the one you care about.** Three releases have shipped a bug past a green check of this
shape:

- A CSS check compared specificity *totals*, so a guard on one state counted as cover for
  a different unguarded state. It passed against the build carrying the bug.
- A rule was asserted *present* — and was, while being outranked by a later rule with the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AHouseOfBards/Bonfire-JellyProfiles](https://github.com/AHouseOfBards/Bonfire-JellyProfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
