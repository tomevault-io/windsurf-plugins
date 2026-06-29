---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

An open-source educational repository containing:
- **LaTeX**: Beamer presentation slides and two-column cheatsheets for Data Science courses (Python, ML, DL, NLP, GenAI, RAG, Quantum Computing, etc.)
- **Code**: Python scripts and Jupyter notebooks demonstrating the concepts covered in the slides

## LaTeX Build System

Compile a specific deck from the `LaTeX/` directory using MikTeX's `texify`:
```bat
cd LaTeX
texify -cp Main_Seminar_AI_ClaudeCode_Presentation.tex
```

Compile all decks matching a pattern (Windows):
```bat
cd LaTeX
for /r %i in (Main_Seminar_*Educators*.tex) do texify -cp %i
```

Compile everything:
```bat
cd LaTeX
make_all.bat
```

## LaTeX Architecture

### 4-level content hierarchy

```
Course (40hr)     Main_Course_*_{Presentation,CheatSheet}.tex
                    └─ course_*_content.tex
                         └─ \input{workshop_*_content}  (+ course-specific extras)

Workshop (4-16hr) Main_Workshop_*_{Presentation,CheatSheet}.tex
                    └─ workshop_*_content.tex
                         └─ \input{seminar_*_content}  (seminar layer only)

Seminar (1hr)     Main_Seminar_*_{Presentation,CheatSheet}.tex
                    └─ seminar_*_content.tex
                         └─ \input{<domain>_<topic>}  (raw topic files)
```

Every deliverable has two output forms sharing the same content file:
- `Main_*_Presentation.tex` — Beamer slides (`\documentclass{beamer}`, uses `template_presentation.tex`)
- `Main_*_CheatSheet.tex` — Two-column landscape notes (`\documentclass{article}`, uses `template_cheatsheet.tex`)

CheatSheet column count convention: Seminars use `multicols{3}`; Workshops use `multicols{2}`.

`\usepackage{beamerarticle}` in `template_cheatsheet.tex` makes Beamer `\begin{frame}` environments compile correctly in article mode — no frame stripping needed.

### Naming conventions
- Topic files: `<domain>_<topic>.tex` (e.g., `maths_linearalgebra_matrices.tex`)
- Content aggregators: `<type>_<subject>_content.tex`
- Driver files: `Main_[Course|Seminar|Workshop]_<Subject>_[Presentation|CheatSheet].tex`
  - Seminar ≈ 1 hour, Workshop ≈ 1 day, Course ≈ 1 week/semester
- Every Seminar and Workshop **must** have both a `_Presentation.tex` and a `_CheatSheet.tex` driver

### Known issues
- `seminar_latex4research_conent.tex` — filename typo (`conent` vs `content`); the file and all references would need renaming together
- `workshop_python_content.tex` and `workshop_maths4ml_content.tex` still use raw files (not yet routed via seminar layer — deferred)
- `seminar_quantum_content.tex` — orphaned; its former driver files (`Main_Seminar_Tech_Quantum_*`) were merged into `Main_Seminar_Tech_QuantumComputing_Overview_*`. Kept as reference; safe to delete.

### Quantum Computing course (added May 2026)
Full 4-level hierarchy for the course "Quantum Computing for Non-Physicists":
- **Course**: `Main_Course_QuantumComputing_{Presentation,CheatSheet}.tex` → `course_quantumcomputing_content.tex`
- **Workshops** (4): `Main_Workshop_QuantumComputing_{Foundations,Circuits,Algorithms,Advanced}_{Presentation,CheatSheet}.tex`
- **Seminar**: `Main_Seminar_Tech_QuantumComputing_Overview_{Presentation,CheatSheet}.tex` (~2 hr overview + QML)
- **Topic files** (14): all prefixed `quantum_` (e.g. `quantum_intro_motivation.tex`, `quantum_gates_circuits.tex`)
- Existing files `quantum_basics_intro.tex`, `quantum_maths_intro.tex`, `quantum_machinelearning_intro.tex` pre-date the course and are referenced optionally (heavier math; commented-out in the new overview seminar)

### Machine Learning course restructured (June 2026)
Full 4-level hierarchy for the 40-hour ML course "Machine Learning for Graduate Students":
- **Course**: `Main_Course_MachineLearning_{Presentation,CheatSheet}.tex` → `course_machinelearning_content.tex`
- **Workshops** (6 × driver pairs):
  - W1 Python for ML (8h): existing `workshop_python_content.tex`
  - W2 Foundations (4h): `workshop_ml_foundations_content.tex`
  - W3 Regression (4h): `workshop_ml_regression_content.tex`
  - W4 Tree-Based & Ensemble (8h): `workshop_ml_treebased_content.tex`
  - W5 Supervised II — KNN/SVM/NB (8h): `workshop_ml_supervisedII_content.tex`
  - W6 Unsupervised & Deployment (8h): `workshop_ml_unsupervised_content.tex`
  - Standalone all-ML workshop (W2–W6, no Python/demos): `Main_Workshop_MachineLearning_{Presentation,CheatSheet}.tex`
- **Seminars** (10 × driver pairs): `seminar_ml_{intro,dataprep,regression,decisiontree,ensemble,knn,svm_nb,clustering,dimreduction,deployment}_content.tex`; drivers are `Main_Seminar_ML_{Intro,DataPrep,Regression,DecisionTree,Ensemble,KNN,SVM_NB,Clustering,DimReduction,Deployment}_{Presentation,CheatSheet}.tex`
- **New demo/assign files**: `ml_course_demo_regression_housing.tex`, `ml_course_demo_svm_digits.tex`, `ml_course_assign_knn_wine.tex`, `ml_course_demo_clustering_customers.tex`, `ml_course_assign_pca_digits.tex`
- **Upgrade status**: Seminar 1 (Intro) upgraded; track remaining 9 in `LaTeX/todo_ml_seminar_upgrade.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yogeshhk/TeachingDataScience](https://github.com/yogeshhk/TeachingDataScience) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
