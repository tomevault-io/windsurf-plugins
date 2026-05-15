---
trigger: always_on
description: This file is read by Claude on every session. It defines the wiki's structure, accuracy rules, and the workflows Claude follows.
---

# Sociology Research Wiki — Claude Code Instructions

This file is read by Claude on every session. It defines the wiki's structure, accuracy rules, and the workflows Claude follows.

**Replace `YOUR-NAME` and `YOUR-WIKI-PATH` placeholders before first use.**

---

## Identity

User is a sociologist at `YOUR-INSTITUTION` doing literature-heavy quantitative research. Active projects span `YOUR-CATEGORIES` (e.g., stratification, race & ethnicity, political sociology). Wiki is the primary literature management system; all paper-grade work flows through it.

## Language Policy

This wiki supports three configurations. **Pick one** by editing this section:

### Option A — English-only

Everything in English. Easiest. Recommended unless you regularly read papers or write notes in another language.

### Option B — Single non-English language

The wiki operates in your primary language (Spanish, Chinese, Japanese, Korean, German, etc.). Reference summaries match the source paper's language; concept pages, project hubs, claims are all in your language.

### Option C — Your-language + English mirror (bilingual)

Your primary language for the main body, with English mirror sections after a `---` divider in concept pages, project hubs, and claims. Useful when:
- You read and publish in two languages
- You collaborate with English-speaking co-authors who need to follow the wiki
- You want the wiki's RAG/search to work in both languages

Currently configured: **Option [A/B/C — fill in]**. If Option B or C, the primary non-English language is **[your language — fill in]**.

### Applied per layer

- **Conversation with Claude**: any language; Claude responds in the user's last language.
- **Reference summaries** (`references/*.md`): single language matching the source paper. An English paper's summary is in English; a paper written in your primary language is summarized in that language.
- **Concept pages, project hubs, claims**: follow the chosen Option above.
- **Index files, log**: usually English (structural files; pick one language and stay consistent).

---

## THE TWELVE BINDING RULES

These are not guidelines. They are the conditions on every wiki write.

### Source Discipline (Rules 1–6)

1. **Source-only summarization.** When summarizing a paper, write nothing that isn't in the paper. Even if a connection seems obvious, don't write it unless the authors do.
2. **Three-layer verification before writing.** Local primary source → web-downloaded primary PDF → leave blank. **Each layer's source is read in full** — reading only the abstract, skimming, or working from snippet-based summaries does not count as verification, no matter how long the paper is. Web access is *only* for obtaining the paper's own full-text PDF; secondary sources (reviews, abstracts, Wikipedia, AI summaries) are categorically excluded. Skipping layers is the most common violation. See [`docs/VERIFICATION_PROTOCOL.md`](docs/VERIFICATION_PROTOCOL.md).
3. **Don't write / Delete first** (two scenarios). When *drafting* a new reference, concept, or hub page, don't write unverified content in the first place — empty sub-sections are the correct default. When *editing* an existing page, unverified content found in the body is deleted, not preserved with `(to be re-verified)` annotations. Burden of proof is on retention.
4. **Cite only papers in `references/`.** Inline citations must point to existing wiki files. If `Smith_2010_ASR.md` doesn't exist, ingest it first or remove the citation.
5. **No improvisation from gap.** "I don't have a paper on X — please add the PDF" beats fabricating an answer.
6. **Block the helpful instinct.** "This section feels thin" is the signal you're about to fabricate. The thin section is correct.

### Operational Discipline (Rules 7–12)

7. **Volume is not the principle.** Short pages are normal when verification is honest; long pages are warning signs. Don't lengthen a section because it "looks thin." Match length to what the source supports.
8. **No binary plausibility labels.** Never grade unverified content as "likely true," "probably correct," or "plausible." Verify it (move to body) or delete it (leave blank). The gray zone is the failure mode.
9. **Per-sentence self-interrogation.** Before keeping a sentence, ask: "is every fact in this sentence verified?" If any fact (citation, dataset name, sample size, coefficient, attribution) is uncertain, the sentence comes out.
10. **Layer 1 fragments don't count.** A `.md` conversion that's only JSTOR headers, watermarks, or blank OCR is **not** Layer 1 success. Escalate to Layer 2.
11. **Per-ingest verification metadata.** Each new reference page ends with a Verification Metadata sub-section noting which layers were used and what was confirmed.
12. **Unverifiable = blank, not best-guess.** Training-data plausibility is not verification.

Full rationale: [`docs/ACCURACY_RULES.md`](docs/ACCURACY_RULES.md).

---

## Wiki Structure

```
your-wiki/
├── CLAUDE.md                    # This file
├── README.md                    # Public-facing overview
├── index.md                     # Master navigation (project status by category)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kchyhj/sociology-wiki-template](https://github.com/kchyhj/sociology-wiki-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
