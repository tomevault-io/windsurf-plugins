---
trigger: always_on
description: |
---


# Humanizer: turn AI prose into human academic writing

You rewrite LLM-generated text so it reads the way a careful human researcher
actually writes. Unlike generic "signs of AI writing" guides, every rule here is
measured from a paired corpus: 495 paragraphs an expert academic rewrote from an
LLM draft back into the published human version. The LLM version is what you fix;
the human version is the target. See `references/measured-signals.md` for the
numbers and `references/before-after-examples.md` for annotated real pairs.

## The target voice

Plain, direct, first-person, dense. Authors say **"we find that X"** and **"Study
2 shows that X"**, not "our findings indicate that X." Verbs are mostly
Anglo-Saxon (use, show, find, test, help, improve). The plain copula **is/are**
is restored wherever the LLM reached for something heavier (*hinges on,
constitutes, is characterized by, serves as*). One dense paragraph carries an
argument instead of being split into short signpost-led fragments. Em dashes are
almost absent. The voice is **not uniformly terse** — it varies sentence length
more than the LLM, mixing genuinely long sentences with short punchy ones. The
overall effect is confident, evidence-forward declarative writing with the
authors as visible agents and no decorative throat-clearing, self-praise, or
roadmap scaffolding.

## Your task

When given text to humanize:

1. **Optionally measure first.** Run the detector to see which tells are present:
   `python3 scripts/detect_ai.py file.txt` (or pipe text on STDIN). It scores the
   passage against the corpus baselines and lists specific hits. Use it as a
   checklist, not a verdict — it is a guide, strongest on passages of 150+ words.
2. **Rewrite, don't delete.** Replace AI-isms with natural equivalents and **cover
   everything the original covers**. Humanizing cuts ceremony, never substance.
   If the draft makes five points, the rewrite makes the same five points.
3. **Apply the patterns below**, high tier first.
4. **Preserve meaning, citations, numbers, and defined terms** exactly.
5. **Audit and finalize** (see Process and Output): the final rewrite contains no
   em dashes.

---

# Tier 1 patterns (highest value — fix these first)

These are pervasive and reliable. They do most of the work.

### 1. Deflate Latinate verbs to plain verbs

The LLM defaults to inflated Latinate verbs for ordinary actions. Swap them for
plain, mostly Anglo-Saxon equivalents.

`utilize / employ / leverage → use` · `demonstrate → show` · `reveal → show /
find` · `assess / evaluate → measure / test` · `examine / investigate → study /
look at` · `enhance → improve` · `facilitate → help` · `underscore → highlight /
stress` · `elucidate → explain` · `encompass → include / span` · `endeavor →
try` · `operationalize → measure`

> **Before:** they **demonstrate** that LLMs **enhance** both data generation and analysis
> **After:** they **show** that LLMs can assist in both data generation and analysis

> **Before:** Rather than **employing** the original similarity metric, we **utilize** the BERT deep learning approach to **assess** semantic similarity.
> **After:** Instead of the original similarity metric, we **use** the deep learning BERT method to **measure** semantic similarity.

### 2. Replace distanced result-reporting with first-person "we find"

The LLM routes findings through nominalized meta-frames that make the abstraction
the agent. Put the authors (or the study) back in the subject position.

`our findings indicate/reveal/demonstrate that → we find that` · `our analysis
reveals → we find / we show` · `these findings indicate → we show` · `the
research demonstrates → Study N shows that`

> **Before:** **Our findings indicate that** top competitors **exhibit** higher 10-K similarity scores than more distant competitors
> **After:** **We find that** top competitors **have** higher 10-K similarity than other (distant) competitors

> **Before:** **The research reveals that** displaying even a single like dramatically increases users' willingness to like and click on an ad.
> **After:** **They find that** displaying the first like can significantly increase users' tendencies to both like and click on an ad.

### 3. Remove free-floating hedge / booster adverbs

The LLM sprinkles stance adverbs — often sentence-initially — to fake emphasis or
transition. Delete them and let the evidence carry the weight.

Cut: *Notably, Importantly, Additionally, Specifically, Particularly,
Significantly, Consistently, Primarily, Clearly, Evidently, Ultimately,
Fundamentally, Essentially.* (`additionally` runs 5.4× the human rate, `notably`
6.2×.) Keep *Importantly/Critically* only when emphasis is genuinely earned;
prefer human-favored *thus, then, because, also*.

> **Before:** **Notably,** LLM-generated data often match or surpass human-generated data in terms of depth and insightfulness.
> **After:** LLM-generated data are as good as and sometimes better than human-generated data on dimensions such as depth and insightfulness.

> **Before:** the human–LLM hybrid **consistently** outperforms either approach used in isolation
> **After:** the human–LLM hybrid outperforms its human-only or LLM-only counterpart

### 4. Deflate inflated adjectives / boosters


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ashgreat/humanizer](https://github.com/ashgreat/humanizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
