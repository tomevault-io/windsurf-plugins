---
trigger: always_on
description: handles this — don't bypass.
---

# CLAUDE.md — nom-vn AI assistant notes

Auto-loaded inside `nom-vn/`. Parent: `PPlanning/CLAUDE.md`.

## Scope

Named after chữ Nôm (historical) but targets **modern VN in chữ Quốc Ngữ**.
Don't prioritize Hán-Nôm corpora.

## Datasets

VN test corpora live in [`benchmarks/data/`](benchmarks/data/); catalogue in
[`docs/datasets.md`](docs/datasets.md). Prefer these over hand-curated text:

- Sentences (4 registers) → `diacritic_eval_v0.txt` (CC0)
- Declarative → `udhr_vi/` (PD) · Classical → `wikisource_vi/` (PD)
- Long-form → `wiki_vi/articles.jsonl` (CC-BY-SA)
- Conversational → `tatoeba_vi/` (CC-BY) · Born-digital PDF → `udhr_vi/udhr_vie.pdf`
- OCR images → `synthetic_ocr_vi/` (CC0) · Legal → `legal_vi/` (PD)
- Office docs → `office_vi/` (PD)

Regenerate: `python benchmarks/data/_fetch_all.py` and
`python benchmarks/data/synthetic_ocr_vi/render.py`.

## Component build workflow

For every new module / model integration, run all stages:

1. **Research.** Survey HF + 2024-26 papers for the smallest VN-finetuned
   open-source model. Record license, file format, public VN benchmark
   number, base arch. Document under `research/<YYYY-MM-DD>-<topic>/` or
   `docs/sota_*` with citations.
2. **Build.** Smallest dependency surface meeting the quality goal. Lazy
   imports, Protocol seams, frozen dataclasses on hot paths.
3. **Test with real models.** Unit tests use fakes; before claiming a
   feature ships, write at least one integration run against the actual
   downloadable model.
4. **Benchmark on real VN data.** Tiny synthetic fixtures show nothing
   when every retriever scores 1.000. Use Zalo Legal QA mirror, ViQuAD,
   MIRACL-vi, or a wiki sample. Commit a baseline JSON.
5. **Iterate as a grid.** Same fixture, same metrics, same warmup +
   best-of-N. Report config in result JSON (silent defaults desync from
   claims).
6. **Honest empties beat fake numbers** (parent §12).
7. **Cross-check vs the model's published number.** Differs >10 % rel on
   the same metric → stop and investigate. Common causes: register
   mismatch, wrong tokenization, wrong max_length, fp16/fp32, cold-start,
   wrong metric variant. Document divergence in `docs/benchmark.md`.

### File-format trust ladder

| Format | Status | Why |
|---|---|---|
| `safetensors` | ✅ always preferred | Deterministic, zero-RCE on load |
| HF `.bin` | ⚠️ accept from major lab when no safetensors exists | Pickled but HF Hub provides SHA256 + audited at scale; document the choice in the wrapper docstring |
| `.pkl` / `.pickle` | ❌ auto-reject | Same RCE without HF accountability (PyVi v0.1) |
| Opaque native (CRFsuite, …) | ⚠️ when format spec is public | Deterministic but opaque; prefer in-tree reimpl when accuracy gap is small |

Reimplement a 5kb CRF tokenizer; don't reimplement a 1B-param transformer.

## Autonomous improvement loop

When the user says "continue until done" / "don't stop" / "improve until
done" / open-ended "keep improving":

1. Build a checklist (TaskCreate) from current bench/doc state, pending
   tasks, latest user steering.
2. Decide next item by ROI = user-visible impact / engineering cost. Tie
   → prefer the one that closes a measurement gap.
3. Execute. Don't ask between tasks.
4. **Blocked?** Research first (web, HF Hub, GitHub issues, model cards).
   Still blocked after one round → one-line summary, mark pending, move
   on.
5. **Off-the-shelf before training.** Before recommending a fine-tune,
   exhaustively bench public Apache/MIT/BSD/safetensors candidates on
   the same corpus.
6. **Multi-corpus required for adoption claims.** Bench on ≥2 distinct
   registers. >10 pp spread → register-overfit; pick a
   register-conditional default.
7. Each item ends in a commit. No long-running uncommitted state.
8. **Always double-check before claiming a number:**
   - Implausible metric check (0 % / 100 % / sub-30 % when card says 90 %
     → bench bug).
   - Cross-reference upstream number (>10 % rel diff → investigate).
   - Dump 5 raw I/O samples and read them. (We caught a 0/800 metric bug
     this way 2026-04-26.)

## VN-language gotchas

### Encoding & normalization

- **NFC vs NFD.** "ề" (U+1EC1) ↔ "e" + combining marks. NFC-normalize
  before any compare. `nom.text.normalize` does this. **Audit every new
  training corpus** with `unicodedata.normalize('NFC', t) == t` on a
  sample. (`tmnam20/Vietnamese-News-dedup` is ~79 % NFD; v5 mixed-source
  training regressed -15.45 pp because of this. `has_diacritics` filter
  does NOT catch NFD because U+0111 'đ' is a distinct codepoint.)
- **đ has a stroke, not a diacritic** (U+0111). `strip_diacritics` must
  replace explicitly.
- **Stacked diacritics.** "ờ" = ơ + grave; multiple precomposed +
  decomposed forms exist. Don't roll your own normalizer.
- 6 vowel-modifiers × 5 tones × 2 modifiers ≈ 60 vowel forms.

### Tokenization & word boundaries

- VN spaces between syllables, not words. "thành phố Hồ Chí Minh" =
  1 word, 5 tokens.
- **bkai-vietnamese-bi-encoder needs underscored input.** "đường thủy" →
  "đường_thủy". Raw text drops R@1 by 15-20 pp. `BKaiEmbedder._segment`
  handles this — don't bypass.
- **`.split()` is wrong for measuring quality.** UD treebank ships
  spaces around punctuation; modern seq2seq attaches them. Always
  `normalize_punct()` both sides — see

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nrl-ai/nom-vn](https://github.com/nrl-ai/nom-vn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
