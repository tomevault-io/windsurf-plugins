---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is a **format specification repo**, not an application. It defines `hunt.md` — an open, vendor-neutral Markdown format for threat-hunting playbooks — plus a hunt library and a reference Python implementation.

The specification is the product. [SPEC.md](SPEC.md) is normative; everything else follows it. When spec and code disagree, the spec wins unless the user says otherwise — and a spec change should be reflected in the tooling, the template, and the example hunts in the same change.

## Commands

Tooling lives in [tools/](tools/) (Python ≥3.10, stdlib + PyYAML only — keep it dependency-free so it can be vendored into a runtime's import path).

```bash
cd tools
pip install -e .                                        # or: pip install pyyaml

python -m huntmd validate ../hunts/kerberoasting.md                 # lint (huntbase profile, default)
python -m huntmd validate ../hunts/kerberoasting.md --profile format # lint against the neutral spec only
python -m huntmd convert  ../hunts/kerberoasting.md                 # hunt.md → Huntbase definition YAML
python -m huntmd convert  ../hunts/kerberoasting.md --to cacao      # hunt.md → CACAO v2 playbook JSON
python -m huntmd convert  ../my-hunt.definition.yaml                # definition → hunt.md (best-effort inverse)
python -m huntmd convert  ../some-cacao-playbook.json               # CACAO → hunt.md (draft, TODO-marked)
python -m huntmd convert  ../hunts/kerberoasting.md --to misp --date 2026-09-08  # → MISP event JSON (pin the date so fixtures don't drift)
python -m huntmd convert  ../hunts/kerberoasting.md --to misp --result ../examples/results/kerberoasting-run.yaml  # + threat-hunt-finding
python -m huntmd convert  ../some-misp-event.json                   # MISP → hunt.md (exact via attachment, else draft)
python -m huntmd convert  ../some-misp-event.json --split -o ../hunts/  # one file per threat-hunt-hypothesis (SPEC §3.8)
python -m huntmd validate ../hunts/kerberoasting.md --profile misp  # HUNT-EX classifiability warnings
python -m huntmd validate ../hunts/kerberoasting.md --profile quality  # opt-in "more than a rule" checks (PROFILES §5)
python -m huntmd validate ../hunts/kerberoasting.md --max-tlp green  # publication gate (public repo policy)
python -m huntmd validate ../examples/results/run.yaml               # lint a run result (SPEC §12)
```

`validate` exits non-zero only on **errors**; warnings pass. Conversion direction is inferred from file extension and *shape* (`Event`/`info`+`Object` ⇒ MISP event, `nodes` ⇒ Huntbase definition, `workflow` ⇒ CACAO), not a flag.

CI ([.github/workflows/lint.yml](.github/workflows/lint.yml)) runs the checks below on every PR, and a separate job enforces the publication boundary (`--max-tlp green` — this repo is public). After touching `core.py`, `cacao.py`, `misp.py` or `results.py`:

1. `validate` + `convert` (all three targets) over every file in [hunts/](hunts/).
   **Compatibility rule:** a 0.5 hunt must lint with the same errors and warnings after your change — `check.py` asserts this against frozen copies in [tools/tests/fixtures/](tools/tests/fixtures/). New checks on 0.5-valid content are `info`, or live in `--profile quality`.
2. **Round-trip must stay exact** for repo hunts: `md → cacao → md` preserves step kinds, slugs, targets, parameters and every edge. This is load-bearing — it's what the CACAO profile claims in [PROFILES.md](PROFILES.md).
3. `python tools/tests/check.py` — the actual suite (stdlib only). Covers all of the above plus guardrails, confidence/`unavailable:` handling, result validation, and MISP (`md → misp → md` byte-exact via the attachment; objects-only events import as lint-clean drafts).
4. **Live MISP check** (opt-in, needs an instance): `MISP_URL=… MISP_KEY=… python tools/tests/e2e_misp.py` — pushes every hunt in `hunts/`, re-imports byte-exact, checks templates/taxonomy presence and `hunt-ex` tag search. Run it after touching `misp.py`'s object shapes; MISP drops malformed/unknown-template objects *silently*, so unit tests can't catch that class of bug.
5. **Corpus check**: [examples/cacao-import/fetch-corpus.sh](examples/cacao-import/fetch-corpus.sh) pulls 49 real CACAO playbooks from six projects; all must import, parse and lint clean (332 steps preserved). Requires `gh` + network. The vendored conversions in [examples/cacao-import/](examples/cacao-import/) are the offline fixtures.

## Architecture

### Three layers, kept separate on purpose

1. **Format** ([SPEC.md](SPEC.md)) — the IR and Markdown syntax. Vendor-neutral by rule: no product, agent, or model may be named in the core format.
2. **Profiles** ([PROFILES.md](PROFILES.md)) — adapters from the IR to a runtime (Huntbase), interchange/sharing targets (CACAO v2, MISP/HUNT-EX), or docs-only. Platform specifics belong **here, never in SPEC.md**. The capability matrix at the top of PROFILES.md is the contract: ✅ native / ⚠️ documented substitution / ❌ lint-and-reject.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huntbase-io/hunt-md](https://github.com/huntbase-io/hunt-md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
