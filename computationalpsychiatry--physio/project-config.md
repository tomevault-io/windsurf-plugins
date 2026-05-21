---
trigger: always_on
description: This file provides context and instructions for AI coding agents (Claude Code,
---

# AGENTS.md — AI Agent Guide for the PhysIO Toolbox

This file provides context and instructions for AI coding agents (Claude Code,
GitHub Copilot, Cursor, etc.) working in this repository. Read it before making
any changes.

---

## What is PhysIO?

PhysIO is a **MATLAB toolbox** for model-based physiological noise correction of
fMRI data. It takes peripheral physiological recordings (cardiac pulsation via
ECG or pulse oximetry; respiration via breathing belt) as input and produces
nuisance regressor text files as output. These regressors can be fed into any
major fMRI analysis package (SPM, FSL, AFNI, BrainVoyager, etc.) as columns of
a General Linear Model (GLM).

**Do not assume this is a Python project.** All core code is MATLAB (`.m` files).
There is no `setup.py`, no `pyproject.toml`, no `package.json`.

**Part of TAPAS.** PhysIO is the physiological noise correction component of the
[TAPAS software collection](https://www.translationalneuromodeling.org/tapas)
(Translational Algorithms for Psychiatry-Advancing Science) by the
Translational Neuromodeling Unit (TNU), University of Zurich and ETH Zurich, now hosted
under the [ComputationalPsychiatry GitHub organization](https://github.com/ComputationalPsychiatry).

---

## Repository Structure

```
PhysIO/
├── code/                     # All MATLAB source code
│   ├── tapas_physio_new.m    # *** MASTER PARAMETER DEFINITION FILE ***
│   │                         # Defines the `physio` struct with inline comments
│   │                         # on every parameter. Read this first to understand
│   │                         # the data model.
│   ├── tapas_physio_main_create_regressors.m  # Top-level pipeline entry point
│   ├── tapas_physio_init.m   # Sets up MATLAB path and SPM toolbox link
│   ├── tapas_physio_review.m # Recreates QA figures from saved physio.mat
│   ├── tapas_physio_report_contrasts.m  # SPM F-test/tSNR QA reporting
│   ├── readin/               # Format-specific log-file readers
│   │   ├── tapas_physio_read_physlogfiles_bids.m
│   │   ├── tapas_physio_read_physlogfiles_biopac_mat.m
│   │   ├── tapas_physio_read_physlogfiles_ge.m
│   │   ├── tapas_physio_read_physlogfiles_philips.m
│   │   ├── tapas_physio_read_physlogfiles_siemens.m  (VB format, ideaCmdTool)
│   │   ├── tapas_physio_read_physlogfiles_siemens_tics.m  (VD/VE/Tics format, XA60, CMRR)
│   │   └── tapas_physio_read_physlogfiles_custom.m
│   ├── preproc/              # Preprocessing: peak detection, filtering
│   ├── model/                # Noise model creation (RETROICOR, HRV, RVT, etc.)
│   ├── assess/               # Model performance assessment
│   └── utils/                # Utility functions
├── examples/                 # Example scripts and input logfiles from different vendors, not part of this repository (see note below)
│   ├── BIDS/
│   ├── GE/
│   ├── Philips/
│   ├── Siemens_VB/
│   ├── Siemens_VD/
│   └── HCP/
├── tests/
│   ├── unit/                 # Unit tests (MATLAB testing framework)
│   └── integration/          # Integration tests for all examples
└── test-reference-results/   # Golden reference output files, not part of this repository (see note below)
├── docs/                     # Additional documentation
├── README.md
├── CHANGELOG.md
└── AGENTS.md                 # This file
```

> **Note:** Example data (logfiles) in examples/ subfolder are **not** stored in this repository. They
> are downloaded separately by calling `tapas_physio_download_example_data()` from
> the MATLAB command line after installation or can be cloned directly from
>  https://github.com/ComputationalPsychiatry/PhysIO-Examples

> **Note:**  Test reference results in test-reference-resuts/ subfolder are required for running unit 
> and integration tests in the tests/ subfolder, but **not** stored in this repository. They
> are downloaded separately by calling `tapas_physio_download_test_reference_results()` from
> the MATLAB command line after installation or can be cloned directly from 
> https://github.com/ComputationalPsychiatry/PhysIO-Test-Reference-Results

---

## The Central Data Structure: `physio`

Everything in PhysIO revolves around a single MATLAB struct called `physio`.
It is constructed by `tapas_physio_new()` and populated/passed through the whole
pipeline.

**Before modifying any pipeline code, open and read `code/tapas_physio_new.m`.**
Every parameter is defined there with inline comments explaining its purpose and
valid values. This is the authoritative technical reference — it supersedes the
scientific papers, which may use older variable names.

Key top-level substructures of `physio`:

| Substructure | Purpose |
|---|---|
| `physio.log_files` | Input file paths and vendor/format selection |
| `physio.scan_timing` | fMRI acquisition parameters (TR, nSlices, nVols, etc.) |
| `physio.preproc` | Preprocessing options (cardiac/respiratory filtering, peak detection) |
| `physio.model` | Noise model selection and order (RETROICOR, HRV, RVT, noise ROIs, motion) |
| `physio.model.retroicor` | RETROICOR Fourier expansion orders |
| `physio.model.hrv` | Heart rate variability model and delays |
| `physio.model.rvt` | Respiratory volume per time model and delays |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ComputationalPsychiatry/PhysIO](https://github.com/ComputationalPsychiatry/PhysIO) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
