---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Local Setup

- If a `claude.local.md` file exists in the repository root, read it first. It contains personal environment-specific settings (HPC usernames, scratch directories, SLURM defaults, conda environments, cluster-specific commands, etc.) that override or supplement the general guidance in this file.

- All paths, environment names, and cluster-specific details in this file are examples only. Users should configure their actual environment in `claude.local.md`. See `claude.local.md.template` for an example configuration.

## Project Overview

cruijff_kit is a toolkit for research with social data and LLMs.

It allows users to design, conduct, and analyze computational experiments. When these experiments are well-designed, the results they produce will answer scientific questions.

cruijff_kit supports the following capabilities:
1. **Fine-tuning LLMs** using torchtune
2. **Evaluating LLMs** using inspect-ai
3. **Prompt engineering** using DSPy (not yet implemented)
4. **Agentic systems** using DSPy (not yet implemented)

## Principles

When working in this project, the following principles should guide your decisions:

1. **Scientific** - All work should emphasize correctness, computational reproducibility, and detailed logging. Each experiment should be logged such that it can be audited by a researcher or Claude.

2. **Modular** - This project will evolve over time and should be designed so that individual components can be added or changed with minimal impact on other components.

3. **Practical** - Work in this project is designed to do science, not win a programming contest. Don't over-engineer or do premature optimization. We don't need hundreds of lines of code to save 5 seconds.

4. **Privacy respecting** - Much of the data in this project is about people. All data should be treated with care, and some should never leave the user's computer system. Tasks should be designed with clear data governance.

5. **Self improving** - Always look for ways to learn from earlier experiments to design new experiments, improve skills, and improve analysis. The more work we do, the easier things should be because we have more designs, results, and logs from which to learn.

## Terminology

**Experiment**: A set of one or more runs designed to answer a scientific question.

**Run**: A single computational operation. Examples include:
- Fine-tuning an LLM with a specific dataset and recipe
- Evaluating an LLM (base or fine-tuned) with a specific dataset

**Pipeline**: Multiple runs combined together in sequence. Examples include:
- Fine-tune a model, then evaluate it on several different datasets
- Fine-tune models with different hyperparameters, then evaluate all models with the same dataset

Note: Some runs must complete before other runs can begin (e.g., a model must be fine-tuned before it can be evaluated).

## Architecture

For detailed architecture documentation, see [ARCHITECTURE.md](docs/ARCHITECTURE.md). Key highlights:

- **Two-stage configuration**: User-friendly YAML → generated torchtune configs + SLURM scripts
- **Custom torchtune recipes**: Enhanced with validation support and checkpoint management
- **Modular tools**: Separate scripts for finetune setup and evaluation setup
- **HPC-first design**: Built for SLURM clusters with automated script generation

## Skills

cruijff_kit includes Claude Code skills to streamline common workflows. These skills are optional - all workflows can also be performed manually.

### Setup
- **ck-setup** ✅ - First-time setup walkthrough OR validate-existing health check for `claude.local.md` (HPC paths, SLURM defaults, conda env). Run this before anything else.

### Primary Workflows
- **design-experiment** ✅ - Plan a series of runs that collectively make up an experiment
- **convert-tabular-to-text** ✅ - Convert tabular data to text representations for LLM experiments (use after design, before scaffold)
- **scaffold-experiment** ✅ - Create organized directory structures, configs, and SLURM scripts for all runs
- **run-experiment** ✅ - Submit jobs to SLURM and monitor their progress until completion
- **summarize-experiment** ✅ - Generate summary.md with key metrics (loss, accuracy) after experiment completion
- **analyze-experiment** ✅ - Generate interactive HTML visualizations from evaluation logs using inspect-viz; selects views based on experimental variables (model, factor, task, metric) and adds calibration / ROC plots when `risk_scorer` is present
- **analyze-to-pdf** ✅ - Convert analysis reports (markdown) to PDF using pandoc
- **create-quiz** ✅ - Turn one or two completed experiments into a self-contained, self-grading HTML quiz that tests a recipient's intuition. Each answer cites the row/figure/section it came from
- **archive-experiment** ✅ - Archive completed experiments, preserving all experiment files while deleting checkpoint directories

### Utility
- **check-release** ✅ - Weekly release check: review changes since last tag, draft changelog, optionally cut a new release
- **create-meeting-agenda** ✅ - Create weekly software meeting agenda in the wiki repo

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [niznik-dev/cruijff_kit](https://github.com/niznik-dev/cruijff_kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
