---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

I have hyperthroidism. It is often difficult to adjust dosage of medication etc between blood tests. I've tracked all apple watch/whoop data for years however and can cross reference periods of time where I was different degrees of hyper thyroid or trending towrds it. 

I would like to research and build/train ML models to find small patterns in a variety of data(RHR, HRV, Respitory rate, sleep etc) that indicate beginning current or end of hyperthyroid periods.

1. Initial Research Phase: 
- create a research plan
- understand data structure of available input data(assume apple health output data with >3 years of all data tracked by apple watch/whoop)
- There will be some subjective/ambiguous output data that we should make a decision on how to use/label training sets. I have both thyroid labs(T3, TSH, T4) from recent times and can label period from memory in the past
- All ML/statictical methods are on the table so we will need to research this
- Research of existing methods(if any) similar to this exist
- Results of this phase should include:
  - Understanding of data structure
  - A plan on how to create data set(including labeling efficiently)
  - proposed approachs for ML(can include multiple that we will benchmark in phase 2)
  - Fill in (Requests of user) section with request data/action
  - Research output to research.md
2. Implementation Phase
- should include data labelling activities 
- implementing proposed ML models/data pipelines from Phase 1
- Should implement robust experiment tracking infrastructure 
- implementation should be relatively portable across machines/environements
- use python and most effective packages for data/ML 
- Output:
  - data should be labeled/pipelines created to automate
  - ML model training should run effectively 
  - ready for multi-approach experimentation 
3. Experimentation Phase
- Run all ML approaches and record benchmarking data
- Include hyperparam tuning for all models
- run this process iteratively 
- Analyze results and create an md file writing up summary of results 
- Add to Request of User if new approaches need to be signed off on
- Output:
  - Successful runs of all Model/params
  - benchmark data in some file
  - ability to make decicision on the correct approach 
4. Productionization/infernce Phase
- Use chose approach from prior steps to build an infernece pipeline that can be integrated to some app(seperate project) to create the output predictions 
- Output:
  - local command line version of inference pipeline
  - approach or packages to use in other project 

Approach and ideas
These are just ideas and dont need to be incorporated if not effective
- Use timeseries approaches to use more then just the current data to predict states(ie trailing propobailities)
- Use trends in output to weight towards future output
- Above I'm basically saying to be baysian 
- Output current state and forcases

Hardware Access
- Local dev is on a 16GB macbook m3 pro
- Access locally to a titan RTX with 24Gb vram
- Open to using cloud services if necessary but only for vram or necessary speed up

Post model training ideas:
- p0:combine mild and moderate labels
- p0: have we considered neural networks? provide reason why not or start implementation
- p1: There are some periods here that are "life events" ie trips weddings etc. I think this data should be ignored as there are many extra factors about it. I can create a file noting the dates of all of these


## Always Rules 

- Respond concisely: only relevant info, code, plans
- Think relatively deeply: contrary to concise response think alot
- Do not comment code
- Flag strange patterns in data and add to request of user
- Programming language: python
- add new research and experiments to research.md

## Workflow preference

- Plan -> confirm -> code -> test -> iterate
- commit changes prior to new tasks and refactors
- use version control smartly 

## Development Commands

```bash
# Parse Apple Health export (run once after new export)
venv/bin/python src/parse_health_export.py

# Parse Whoop HRV and create unified HRV (SDNN + RMSSD z-score normalized)
venv/bin/python src/parse_whoop_hrv.py --whoop-csv data/physiological_cycles.csv

# Extract features into 5-day windows
venv/bin/python src/feature_extraction.py

# Generate labeling visualization
venv/bin/python src/visualize_for_labeling.py

# Train models (requires data/labels.csv)
venv/bin/python -m src.train --model random_forest
venv/bin/python -m src.train --model xgboost
venv/bin/python -m src.train --model all  # run all baselines

# With semi-supervised learning
venv/bin/python -m src.train --model xgboost --semi-supervised

# Sequence models (LSTM/GRU)
venv/bin/python -m src.train_sequence --model lstm --seq-length 6
venv/bin/python -m src.train_sequence --model gru --seq-length 6

# View experiment results
venv/bin/mlflow ui
```

## Inference Commands

```bash
# One-time: train and save production models
venv/bin/python -m src.save_models

# Run inference on Apple Health export
venv/bin/python -m src.infer --input data/apple_health_export/export.xml


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ianrowan/thyroidEarlyDetection](https://github.com/ianrowan/thyroidEarlyDetection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
