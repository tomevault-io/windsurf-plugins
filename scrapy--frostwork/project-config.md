---
trigger: always_on
description: Frostwork is a **treeless, one-pass HTML extraction engine**: a Rust core that answers a scraper's
---

# AGENTS.md — working in Frostwork

Frostwork is a **treeless, one-pass HTML extraction engine**: a Rust core that answers a scraper's
CSS/XPath selectors in a single streaming scan, **without building a DOM**, staying value-identical to
lxml/libxml2 on the common web. No parse tree, **no fallback**. Tagline: *"Frost never takes root."*

Start with [README.md](README.md) for the pitch + API, [docs/DESIGN.md](docs/DESIGN.md) for how/why
it's built, and [docs/COMPATIBILITY.md](docs/COMPATIBILITY.md) for the exact supported/divergent/
unsupported contract.

## Build & test

```bash
cargo test                     # Rust unit vectors (page-object layer included; python feature OFF)
cargo build --release          # builds the `differ` + `bench` binaries the harness below uses

# differential vs lxml — the correctness gate; needs Python + the pinned oracle toolchain:
python -m venv .venv && .venv/bin/pip install -r requirements-test.txt   # (= make bootstrap)
.venv/bin/python tools/diff_lxml.py      # GATE: DIVERGE + CRASH must be 0
.venv/bin/python tools/enc_check.py       # encoding parity vs Parsel
.venv/bin/python tools/bench_matrix.py    # throughput vs Parsel

# Python bindings (PyO3 + maturin) — the `python` cargo feature; see docs/PYTHON.md:
.venv/bin/maturin develop                 # build extension + install `frostwork` (editable) into .venv
.venv/bin/python -m pytest tests/test_python.py   # bindings + Page + web-poet + parsel cross-check
```

Note: the `python` feature builds an `extension-module` cdylib. `cargo test`/`cargo build`/the bins
must NOT enable it (they'd fail to link libpython); maturin builds only the `--features python` cdylib.

The `Makefile` bundles these into one-command gates (`make help` lists them): `make ci` = `test`
(unit + clippy) + `gate` (differential + encoding parity vs lxml) + `gate-corpus` (value parity over
the fixture corpus) + `gate-seq` (every tag sequence up to length 4, compared on the whole tree) +
`fuzz-smoke` + `py` (which now also type-checks the shipped package) + `gate-webpoet` (the web-poet
integration vs parsel, compared on the whole item, plus the derived upstream-surface snapshot) +
`gate-webpoet-mutate` — the minimum pre-release check. Individual targets run their own piece.

The limits of that gate are worth knowing before trusting a "100% parity" number — each bullet below is
a way it has read 100% while the engine was wrong:

- **It only sees generated pages.** A generator reproduces the malformations its author thought of, so it
  is not evidence about the real web — the `dd`/`dt` same-tag close and the dropped-end-tag text split
  were both found on real doc-generator output while the generated gate read 100%. `make gate-corpus`
  runs the gate's verdict over `tests/corpus` (self-authored, but shaped like what broke us, and
  proven to discriminate); point `CORPUS=<dir>` at a real corpus for what fixtures cannot give.
  `make corpus-real` fetches ~30 real pages (gitignored, never vendored) chosen for doc-GENERATOR variety
  and gates over them — the one check that sees markup nobody here wrote or imagined.
  **A 1000-page Common Crawl sample then found things none of the above did**, and the shape of what it
  found is the point: a tag name the engine truncated (`<p<mip-img …>` reported as a `<p>` that is not in
  the document — a FALSE POSITIVE, the outcome no-fallback exists to prevent), a `<meta charset>` past the
  prescan window, two whole tree-construction rules (what ends `<head>` also starts `<body>`; a `<!DOCTYPE>`
  does not break a text node) and the missing document-frame synthesis behind them, a decoder sweep that
  had been *sampled* rather than exhaustive and so read "full parity" while euc-jp differed on the wave
  dash, and documented divergences whose stated scope was too narrow. A 2000-page sample then found the
  next one down: end-tag scope is a PRIORITY comparison, not a set of boundary elements, so `</tr>` cannot
  unwind an open `<tbody>` and a table generator's rows lost their cells. A 10000-page sample then found
  three more, all in the document frame and all hidden by an EXCLUSION rather than a short list: the frame
  probes skipped the three frame names themselves ("asking where `<head>` nests is meaningless"), so a
  page whose first tag is `<head>` got no `<html>` at all, a `<body>` written after `</body>` was ignored
  where libxml2 starts a second one, and `<body>` — which out-ranks every end tag — was missing from the
  priority order because the derivation had ruled it "never open". Two more 10000-page samples then found
  three more, and the most valuable was in the TOKENIZER rather than the tree: only an ASCII letter after
  `</` starts an end tag, so `</%>` is a bogus comment that SPLITS a text node while `</>` is ignored and
  does not — reading both as "scan a name, skip to `>`" merged a page's copyright line and was also the
  single largest source of unattributed divergences in the malformed-HTML fuzzer (NOVEL 93 -> 5). The
  other two were the frame again: `<frameset>` ends the head but starts no body, and content after
  `</html>` gets a second ROOT `<html>` rather than no parent at all. A third 10000-page sample found four

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scrapy/frostwork](https://github.com/scrapy/frostwork) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
