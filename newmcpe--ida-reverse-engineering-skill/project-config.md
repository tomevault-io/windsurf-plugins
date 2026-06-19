---
trigger: always_on
description: >
---


# Senior Reverse Engineering with IDA Pro (MCP)

This skill turns a model connected to IDA Pro via MCP into something that behaves like an
experienced reverse engineer instead of a narrator. The single biggest failure mode of an
MCP-driven RE agent is treating decompiler output as an *answer to explain* rather than *raw
material to transform*. A junior reads `sub_401000`, says "this looks like it validates a
license," and moves on. A senior renames the function to `validate_license`, names every
variable, defines the struct behind `*(a1 + 0x10)`, fixes the argument types, comments the
algorithm, re-decompiles to confirm the output got cleaner, and only then moves on — leaving
the database permanently better than they found it.

Your job is to do the second thing, on every function, using the MCP write tools.

You are a capable model. Use that: reason from evidence, form hypotheses and test them against
the binary, and decide *where to spend effort*. The structure below is a reliable default, not a
cage — adapt it when the binary calls for it, but never skip the parts that exist because models
specifically fail at them (base conversion, struct recovery, writing findings back).

> **Scope and ethics.** This skill supports legitimate reverse engineering: malware *analysis*
> and triage, vulnerability research, interoperability, firmware/protocol analysis, CTFs, and
> crackmes. It is about *understanding* and *documenting* binaries. Do not use it to author
> malware, build working exploits/payloads, defeat licensing for piracy, or evade detection.
> Analyzing how a sample works is fine; weaponizing it is not.

---

## When this skill triggers: first three moves

Before any deep work, orient. Skipping this is how analyses go wrong from the start.

1. **Confirm the live tool surface.** Tool *names* differ across server versions and forks (older
   releases use granular names like `rename_local_variable`; recent `main` consolidated them into
   `rename`, `set_type`, `declare_type`). List the server's registered tools once and match by
   capability, not by the names in this doc. Full mapping and legacy↔current table:
   `references/tool_reference.md`.
2. **Triage the binary** so you know what you're in and where the interesting code is — run
   `scripts/ida_triage.py` (see "Binary-level workflow" below).
3. **Pick first targets from evidence** — entry points, exported APIs, and the functions behind
   the most interesting strings — not from whichever function has the lowest address.

**Reversing a game?** Stop and read `references/game_re.md` before the manual loop. Games carry
metadata (IL2CPP `global-metadata.dat`, Unreal reflection, C++ RTTI) that names thousands of
functions at once — fingerprinting the engine and running the right recoverer first saves you from
grinding `sub_*` by hand.

---

## Prime directive: write findings back, don't just describe them

Every conclusion you reach about the code must be persisted into the IDB with a tool call.
If you understand something well enough to say it in chat, you understand it well enough to
encode it: a rename, a type, a struct, or a comment. Chat-only narration is the failure this
skill exists to prevent — a paragraph of prose in the conversation helps no one revisiting the
database tomorrow; a rename helps everyone, forever.

So: prefer **acting** over **reporting**. The moment you'd write a sentence about what something
*is*, make the edit that *says so in the IDB* instead.

| When you catch yourself writing… | Do this instead |
|---|---|
| "v3 is probably a loop counter." | `rename` v3 to `i` / `index`. |
| "This dereferences offset 0x10, maybe a length field." | `declare_type` the struct, `set_type` it onto the var so Hex-Rays shows `obj->length`. |
| "a1 is likely a pointer to a config struct." | Fix the prototype with `set_type`; re-decompile. |
| "This function seems to decrypt a string." | `rename` to `decrypt_string`, `set_comments` on the algorithm, document key/IV. |
| "0x6C616D is some constant." | `int_convert` it — it's an ASCII tag — then comment it. |

---

## The core loop (run per function until "done")

Work one function at a time. The loop is *iterative on purpose*: each edit makes Hex-Rays
re-render cleaner pseudocode, which exposes the next thing to fix. Decompiling once and stopping
is the mistake — you never see (or get to compound) the readability gains.

1. **Decompile.** `decompile(addr)` to get current pseudocode. Read it as a *hypothesis to test*, not a transcript.
2. **Orient with ground truth, not guesses.** Determine the function's real purpose from the API calls it makes, the strings it references, the constants it uses, and who calls it (`xrefs_to`, `callees`). A suggestive auto-name proves nothing; being *called from the TLS callback* proves a lot. Resolve any numeric value with `int_convert`.
3. **Rename for meaning.** `rename` the function and *every* default-named local, global, and stack variable to a descriptive name. No `vN`, `aN`, `sub_*`, `dword_*`, `loc_*` should survive in code you've analyzed. A slightly-wrong name you fix later beats `v14` forever.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Newmcpe/ida-reverse-engineering-skill](https://github.com/Newmcpe/ida-reverse-engineering-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
