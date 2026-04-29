---
trigger: always_on
description: Frank Investigator is a Rails 8.1 fact-checking pipeline that assesses claims extracted from news articles. It uses SQLite3, Solid Queue, Solid Cable, Solid Cache, Propshaft, and Turbo Streams for live updates.
---

# Frank Investigator — Development Guide

## Project Overview

Frank Investigator is a Rails 8.1 fact-checking pipeline that assesses claims extracted from news articles. It uses SQLite3, Solid Queue, Solid Cable, Solid Cache, Propshaft, and Turbo Streams for live updates.

## Core Principle: Truth Over Consensus

**A fact does not become false because a million sources repeat a falsehood, and a falsehood does not become true because it is popular.**

This is the foundational design constraint of the entire system. Every scoring, weighting, and aggregation decision must respect it:

1. **Authority trumps volume.** A single primary source (government data, court records, scientific papers, official corrections) always outweighs any number of secondary sources (news articles, blogs, social media) that contradict it. The `SECONDARY_WEIGHT_CAP` ensures that sheer volume of non-primary evidence can never dominate.

2. **Primary source veto.** If a primary-tier source disputes a claim and no primary source supports it, the verdict cannot be `:supported` — it is forced to `:mixed` and confidence is capped. Two opposing primary sources also force `:mixed`.

3. **LLM votes are weighted by confidence, not counted by heads.** A single model with high confidence outweighs two models with low confidence. We never use simple majority voting.

4. **Independence matters more than quantity.** Ten articles from the same wire service count as one editorial voice. The `IndependenceAnalyzer` clusters sources by editorial origin and the independence score reflects unique editorial groups, not article count.

5. **Verdicts are living, not permanent.** Assessments go stale and get reassessed when new evidence appears. The `VerdictSnapshot` audit trail tracks every change with the evidence state at that point, so we never lose history.

6. **Smear/viral campaign defense.** When 3+ secondary sources support a claim but zero primary sources exist, the system flags this as "unsubstantiated viral" and caps confidence at 0.45. Volume of gossip without original evidence (no confession, no court verdict, no official record) must never produce a high-confidence `:supported` verdict.

7. **Evaluative thesis claims are not factual verdicts by default.** Broad value judgments about a person's performance ("X was a good minister", "Y was a terrible leader") are opinion/framing unless the article decomposes them into measurable subclaims. The system should assess the underlying fiscal, legal, statistical, or documented performance claims, not ratify the headline's value judgment.

8. **Circular citation detection.** Articles that only cite each other (A→B, B→A) or cite sources with no substantive content are flagged as thin citation chains. The `CircularCitationDetector` walks `ArticleLink` relationships to identify echo chambers where outlets reference each other but none have original evidence. Citation depth score penalizes these patterns.

9. **Citation grounding required.** An article is only "grounded" if it links to substantive external sources outside the evidence set. Articles with no outbound citations or that only cite other articles in the same evidence set are treated as ungrounded, reducing the overall citation depth score.

10. **Headline-body divergence penalty.** Articles whose headlines make definitive claims ("accused of", "caught", "confirmed") while their body text hedges ("allegedly", "no police report filed", "sources say", "could not be verified") have their authority score discounted. A baiting article is unreliable evidence regardless of the outlet's general reputation.

11. **Headline citation amplification detection.** When Article B quotes Article A's sensational headline but not its qualifying body content, this is flagged as "headline amplification". This is a key mechanism in smear campaigns: one outlet writes a baiting headline, others cite it as established fact. The `HeadlineCitationDetector` catches this pattern and penalizes the evidence set's citation depth score.

12. **Rhetorical fallacy detection.** After claims are assessed, the `RhetoricalFallacyAnalyzer` examines the article's written structure against its own factual claims. It detects 16 fallacy types including classical fallacies and six patterns derived from Schopenhauer's 38 Stratagems ("The Art of Being Right", 1831): equivocation (#2), twisted conclusion (#9), false admission (#11), paradox framing (#13), odious categorization (#32), and faulty proof exploitation (#37). Of Schopenhauer's 38, 19 are covered by this system (6 as dedicated types, 13 via other analyzers like headline bait, source misrepresentation, and emotional manipulation); the remaining 19 are debate-specific tactics that don't reliably apply to written news analysis.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akitaonrails/frank_investigator](https://github.com/akitaonrails/frank_investigator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
