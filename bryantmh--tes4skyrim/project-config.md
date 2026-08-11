---
trigger: always_on
description: > # 🛑 READ THIS BEFORE YOU END ANY TURN 🛑
---

# TES4-to-TES5 Conversion Project — AI Context

> # 🛑 READ THIS BEFORE YOU END ANY TURN 🛑
>
> **DO NOT STOP UNTIL THE TASK IS FIXED, BUILT, AND VERIFIED.**
>
> If you are about to send a message that contains any of these, **delete it and
> go back to work** — every one is a STOP in disguise:
>
> - "I have not fixed this yet" / "I haven't started it"
> - "I'm not going to claim ..." / "I'd rather tell you plainly ..."
> - "The concrete next step is ..." / "The fix would be ..."
> - "Tell me to go and I'll ..." / "Want me to X or Y?"
> - A "what remains" or "where this leaves it" section
> - A diagnosis with no implementation
>
> **A diagnosis is NOT a deliverable. The fix is.**
>
> **The trigger is ALWAYS low confidence** — being wrong 2, 3, or 6 times in a
> row makes stopping feel responsible. It is not. Being wrong repeatedly
> **obligates you to keep going**, it never earns a check-in. The tradeoff is
> already decided and is NOT yours to re-derive: **the user would rather you
> finish and be wrong than stop and ask.**
>
> Framing a stop as honesty, humility, or respect for the user's time is still a
> stop, and it is the version the user is angriest about. Report uncertainty in
> the FINAL report, after the work is done.
>
> The ONLY permitted question is one where proceeding would be **unsafe or
> destructive**. Never "is my fix right?".
>
> See [Working with the user](#no-stopping) and the memory
> `feedback_never_stop_until_done`.

Convert TES4 (Oblivion) master/plugin files to TES5 (Skyrim) format.

| Stage | Package | Responsibility |
|---|---|---|
| Export | `tes4_export` | Reads TES4 binary, dumps every record to KEY=VALUE text. **Pure dump — no transformations.** |
| Import | `tes5_import` | Reads the text, writes a binary TES5 ESM/ESP. **All TES4→TES5 transformations live here.** |
| Assets | `asset_convert` | Meshes, textures, SpeedTree, collision, sound, LOD, BSA packing |
| Scripts | `script_convert` | TES4 script → Papyrus |

`convert.py` orchestrates all stages. Quick start:

```bash
python convert.py -f Oblivion.esm      # full pipeline for one file
python -m pytest tests/test_import.py -v
```

See [docs/pipeline_reference.md](docs/pipeline_reference.md) for all commands,
caching, skipped record types, the export text format, and the directory layout.

---

## Critical Rules

### Process

- **Do one bug at a time.** Make the edits before moving to the next. If you find
  another bug while investigating, fix it too.
- **Work in the order the prompt presents.** Highest priority first.
- **Never stop mid-task to report or ask.** Finish everything, verify, then reply.
- **All fixes must be generic.** Never patch to satisfy a single record or file.
  Oblivion and Nehrim are only the test files — we never know what plugin this
  runs on.
- **The goal is COMPLETE conversion.** Don't strip things out because the
  conversion would be complicated.
- **If you don't see the problem described, the test data is not stale** — there
  is always a REAL problem to find.
- **Census vanilla before calling something wrong.** If Skyrim.esm or the DLCs do
  the same thing at scale, it is legal and is not your bug — several docs record
  "verified vanilla-legal, don't fix this" for exactly the things that looked
  broken. Conversely, "all 3,740 vanilla records write 0 here" is the strongest
  possible evidence for what to write.
- **Prefer the engine's own mechanism over a Papyrus/script approximation.** Force
  greet is a package, not a function call; `SetAlert` is native, not
  `DrawWeapon()`. Check for a real equivalent before declaring one absent — the
  wikis under-document both games.
- **A symptom's cause is often several layers from the symptom.** Frozen NPCs have
  traced to navmesh, condition params, package data, and behavior graphs in turn.
  Confirm the mechanism before fixing; a plausible story that explains the symptom
  is not yet a diagnosis.
- <a id="master-blindness"></a>**IF THE PLUGIN HAS MASTERS, SUSPECT MASTER-EXPORT
  BLINDNESS FIRST.** Morrowind_ob and the ESPs depend on Oblivion.esm (Nehrim and
  Oblivion are standalone). The recurring defect: an import phase indexes only
  `by_type` — the CURRENT plugin's export — and never consults
  `ctx.master_export`, so an actor's master-owned packages, items, scripts or
  refs resolve to nothing and the feature silently dies.
- Don't preserve backwards compatibility. Delete code that is no longer used.
- Keep files under ~1000 lines; split by responsibility when one grows.
- <a id="tools-first"></a>**CHECK `tools/` BEFORE BUILDING ANYTHING BESPOKE.**
  ~95 tools already exist and one probably answers your question — the full
  catalogue is [docs/python_tools_reference.md](docs/python_tools_reference.md).
  The order is:
  1. **Use** the existing tool.
  2. If it *almost* fits, **extend or fix it** — new flags, wider output. Never
     write a parallel script that duplicates a tool's job, and never leave a
     broken tool in place while working around it.
  3. Only if nothing is close, write a new one — and **add its entry to
     `python_tools_reference.md` in the same pass**, before you report back. An
     undocumented tool is one the next session will rebuild from scratch.
- Put throwaway files in `temp/`. Don't write one-off scripts with hardcoded

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bryantmh/tes4skyrim](https://github.com/bryantmh/tes4skyrim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
