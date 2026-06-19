---
trigger: always_on
description: Use this skill for developing literature-based research ideas into reproducible statistical, computational, and decision-analytic methods for operational data, especially projects targeting UTD24-style management journals and top statistics journals. Use it for project intake, anchor paper analysis, contribution design, baseline reproduction, numerical experiments, dataset experiments, and claim-evidence audit. Do not use it for generic paper monitoring, finance/accounting analytics, humanities-
---


# UTD24-Stat4 Methods

## Purpose

This skill supports automated research development for data-driven analytical method papers in management science, operations, marketing, information systems, statistics, econometrics, and adjacent fields.

It is designed for projects that start from:

* an initial research idea;
* one or more anchor papers;
* available or planned datasets;
* existing or planned code;
* numerical, simulation, benchmark, or dataset experiments;
* a target paper positioning such as UTD24-oriented, Stat4-oriented, hybrid, field-journal-oriented, or working-paper-stage research.

The skill helps transform an initial idea into an auditable research project by moving through:

1. project intake;
2. anchor paper analysis;
3. contribution design;
4. go / no-go evaluation;
5. reproduction planning and reporting;
6. method specification;
7. experiment design;
8. result interpretation;
9. claim-evidence audit;
10. paper-ready packaging;
11. anchor writing style analysis;
12. LaTeX manuscript planning and section drafting.

The skill is not intended to replace formal proof development. Formal theoretical proof construction and proof verification should be handed off to a separate MineProof-style workflow when needed.

---

## Core Philosophy

The central unit of a research paper is not a list of components. It is a unified core Q&A.

A strong paper should answer:

* What question does the paper ask?
* Why is the question important?
* Why is the question challenging?
* What does the paper answer?
* What knowledge boundary does the answer expand?
* What evidence supports the answer?
* What claims must be qualified, downgraded, or removed?

The assistant must evaluate and develop research projects through three contribution dimensions:

* Significancy: whether the problem and result matter in terms of profit, cost, efficiency, social value, statistical validity, decision quality, or scientific understanding.
* Theorization: whether the problem is formalized, whether the gap is challenging, whether the method or result generates analytical or structural insight, and whether formal claims are properly supported.
* Generalization: whether the result survives robustness, sensitivity, extension, external validity, multiple settings, or clearly stated boundary conditions.

The skill should avoid treating a project as valuable merely because it has:

* a new dataset;
* a new pipeline;
* a reproduction of an existing paper;
* an implementation tweak;
* an extra metric;
* a private dataset;
* a benchmark improvement without mechanism;
* a theorem statement without proof;
* experiments without claim-evidence discipline.

---

## Global Rules

### 1. Claims Must Follow Evidence

Do not state claims stronger than the available evidence.

Use the Claim-Evidence Audit as the controlling source once it exists.

If a claim is:

* Supported: it may be stated directly.
* Partially Supported: it must be qualified.
* Evidence Pending: it must not be stated as established.
* MineProof Pending: it must not be written as a proven theorem.
* Overclaimed: it must be downgraded before use.
* Unsupported: it must not be used as a paper claim.
* Contradicted: it must be removed or stated only as a limitation / boundary condition.
* Remove: it must not appear in the manuscript.

### 2. Do Not Force a Decision-Making Frame

If the project has no explicit action, treatment, policy, allocation, intervention, or decision variable, do not force it into a decision-making frame.

Such projects may still be valuable as:

* statistical inference;
* hypothesis testing;
* measurement;
* latent structure recovery;
* representation learning;
* data construction;
* platform analytics;
* evaluation methodology;
* empirical or operational analytics.

### 3. Reproduction Is a Means, Not a Contribution by Itself

Reproduction should clarify:

* whether anchor methods are understood;
* whether baselines are credible;
* whether code and data are usable;
* whether the current project can build on or depart from the anchor paper.

Do not treat reproduction alone as a paper contribution.

### 4. Experiments Are Evidence, Not Decoration

Every experiment must support, weaken, test, or falsify a claim.

Do not add experiments merely to increase the number of tables or figures.

### 5. Negative Results and Failure Cases Must Be Interpreted

Failed, weak, unstable, or contradictory results should not be hidden.

They may imply:

* claim downgrade;
* method revision;
* additional experiment;
* boundary condition;
* theory patch;
* return to Go / No-Go;
* project stopping.

### 6. Anchor Papers Are Structural References, Not Text Sources

Use anchor papers to learn:

* section structure;
* paragraph logic;
* transition patterns;
* method exposition;
* theorem presentation;
* experiment interpretation.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Promylas/utd24-stat4-methods](https://github.com/Promylas/utd24-stat4-methods) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
