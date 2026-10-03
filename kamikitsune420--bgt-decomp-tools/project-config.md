---
trigger: always_on
description: General-purpose tooling for recovering code and assets from games built with **BGT
---

# BGT & NVGT Games — Decompilation Toolkit

General-purpose tooling for recovering code and assets from games built with **BGT
(BlastBay Gaming Toolkit)**, the 2010 audio-game engine by BlastBay Studios, and
with its open-source successor **NVGT**.

This document covers BGT in depth. NVGT lives in `tools/nvgt/` (the
`bgtdecomp.nvgt` subpackage, merged from nvgt-source-recovery) and is documented
in `docs/nvgt.md`. The two share a scripting language and almost nothing else:
packaging, encryption, AngelScript version and asset packs all differ.
`tools/engine.py` (`bgt identify`) decides which half an executable needs, and
claims an engine only when that engine's own self-verifying layer agrees.

This folder ships **only this document and the Python tools**. It holds no game
files, no extracted binaries and no per-title findings — those belong in the folder
for the game you are working on. Everything here is reproducible from a game
executable in one command.

---

# What these tools assume, and why it holds

A BGT game is the BGT runtime stub with the compiled AngelScript module appended as
a **PE overlay** — past the last section, so it is never mapped into memory and no
section dump will show it.

The important property: **the runtime is the same across titles.** BGT games do not
need per-title reverse engineering. This has been confirmed against three unrelated
shipped titles spanning 2014–2019 — same trailer, same key, same container magic,
same compression, same pack format, all opened by this toolchain unmodified.

The corollary is worth stating because it is a common assumption: a developer
shipping several BGT games is almost certainly shipping **stock BGT**. If a title
does not open, suspect a different BGT *version* before suspecting a modified engine.
What genuinely varies between builds is the **AngelScript version** BGT was compiled
against, which changes how strings are serialized *inside* the module — not how the
module is stored.

---

# The chain

```
game.exe
  |- PE stub (BGT runtime, itself LZ77-compressed inside BGT's exec.bin)
  `- overlay
       |- "<n> "               ASCII decimal + space; n = size of an optional
       |                       embedded pack. Usually 0, giving the bytes "0 ".
       |- AES-256-CBC          key = SHA256(keygen(seed)), IV = key[:16]
       |    |- "<32 hex>=<flag> printf<len>\0"      container header
       |    `- LZ77 blob, ending " <uncompressed length>"
       |         `- AngelScript bytecode
       `- 12-byte trailer      "xproc10\0" + LE u32 overlay offset
```

The AES key is **generated, never stored** — which is why no string search in a BGT
executable ever finds one:

```python
v = seed                       # 0x11 in stock BGT
for _ in range(32):
    v *= 3
    v = 5 if v >= 0x80 else (0xd if v == 0 else v)
    key_material.append(v & 0xff)
aes_key = sha256(key_material)
```

`bgtlib.find_seed()` recovers the seed by trial decryption rather than assuming it,
so a build that changed it still opens. The container header — 32 hex characters
followed by `=` — is a strong enough constraint that a wrong seed is never mistaken
for a right one.

Note the container header is **fixed-shape but not fixed-length**: the declared
length is decimal, so a larger module means a longer header. Compute it, never
hardcode it.

## UPX-packed runtimes

Some titles ship the runtime compressed with UPX. The overlay is untouched --
UPX leaves anything past the last section alone -- so unpacking the script
does not care. What breaks is everything that reads the *engine*: `asBCInfo[]`
is inside the compressed blob. `upx.py` rebuilds the memory image following UPX
3.96's own `PeFile::unpack0`, and `as_opcodes.extract()` reads from that:

1. PackHeader (`UPX!`, just before the second section's raw data) names the
   method, filter and lengths.
2. NRV2B/2D/2E decode to exactly `u_len`; `c_adler` and `u_adler` must match.
   (`u_adler` is taken before unfiltering, so it proves decoding only.)
3. The stream's last dword locates the original PE header and section table.
4. The code range `[codebase, codebase+codesize)` is unfiltered (`filter/ct.h`,
   `filter/cto.h`; ids 0x11-0x16, 0x24-0x26). Unimplemented filters leave code
   filtered and say so -- data sections, where the opcode table lives, are
   never filtered.
5. Relocations are replayed: UPX stores each relocated dword big-endian as
   `value - imagebase - rvamin`. Missing this step leaves every pointer in
   `.rdata` wrong while the data *looks* plausible.

Checked against `upx -d` on a shipped title: `.text` and `.data` identical, and
`.rdata` differs only in the import descriptors, which are deliberately not
rebuilt. `bgt opcodes` on the packed file and on the `upx -d` copy give the
same table. The trailer stores an **absolute** overlay offset, so a `upx -d`
copy (or any PE rewrite) has a stale trailer; `split_overlay` then looks for the
overlay at the end of the PE image, and `bgt identify` reports the staleness,
because BGT's own loader will not find the script in such a copy.

---

# Tools

Pure Python 3. Needs `pycryptodome`. Nothing here needs `capstone` — AngelScript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KamiKitsune420/bgt-decomp-tools](https://github.com/KamiKitsune420/bgt-decomp-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
