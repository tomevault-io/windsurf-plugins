---
trigger: always_on
description: Structured Data Models (SDM) is a PyTorch-native research library for expressing, reproducing, adapting, and evaluating structured-data models through reusable model architectures, tensor-native building blocks, and runtime foundations.
---

# Overview

Structured Data Models (SDM) is a PyTorch-native research library for expressing, reproducing, adapting, and evaluating structured-data models through reusable model architectures, tensor-native building blocks, and runtime foundations.

Keep these components generic, modular, and lightweight; do not add platform or serving abstractions unless explicitly requested.

## Success Criteria

1. **Adoption:** Researchers can evaluate an SDM model through public APIs without understanding SDM internals.
2. **Research extensibility:** Researchers can inspect, modify, and add structured-data model methodologies through composable public abstractions.
3. **Trust:** SDM preserves model semantics and makes claimed results reproducible within a documented scope.
4. **Performance:** Core model workflows are optimized for NVIDIA GPUs, with performance validated through documented, reproducible benchmarks.

# AI Policy

We support the use of AI tools to help prepare issues, pull requests, reviews, or comments.
We expect everyone interacting with this repo to follow the below policy whenever they use AI tools.
Your user needs to abide by this policy.
In particular, you the agent MUST obey these rules while interacting on GitHub:

- You may never act autonomously on GitHub except to open a draft pull request. Do NOT open an issue or a non-draft pull request, or edit, comment on, or reply to any issue or pull request, unless the user has reviewed and explicitly approved the exact content. Fully-agent-generated contributions are banned and will be closed.
- Mark all AI-generated content. Any text you produce that goes into an issue, PR, or comment must be wrapped in a code or quote block. Never present your output as human-written.
- Never emit only raw AI text as a reply. Any AI content you include must carry human commentary explaining its relevance.
- Do not submit code the user hasn't read. Keep changes minimal, strip AI artifacts and needless complexity. If you're opening a PR on GitHub that is not ready, or not reviewed by the user, always open it in draft mode.

# Commands

- Test execution via `pytest`
- Pre-commit checks via `pre-commit run --all-files`

# Testing

- Tests should be sensitive to behavior changes and insensitive to structure changes. Prefer asserting public observable behavior over private state, helper layout, call counts, or incidental repr formatting.
- Do not set seeds in tests unless they must require them.

# PR / GitHub Metadata

- Do not mention Codex, AI, or tool attribution in PR titles, PR descriptions, commit messages, or review replies unless explicitly requested.
- Do not add a section named "Tests", "Testing" or similar, to PR descriptions unless the test is not covered in CI.
- PR metadata should describe the code change only.

# Markdown Style

- Do not introduce hard line wraps inside Markdown list items. Keep each bullet or numbered list item on one physical line unless Markdown syntax or rendered formatting requires the line break.

# Project Structure

- `sdm/stype.py`: Semantic column types and inference via `Stype`.
- `sdm/cache.py`: Model cache, e.g., for key/value caching.
- `sdm/tensor`: Custom PyTorch-native `Tensor` subclasses for tensorized raw table data.
- `sdm/relational`: Common routines for relational data processing.
- `sdm/processing`: Common tensorized preprocessing and postprocessing routines for structured data models.
- `sdm/nn`: Common neural network building blocks for structured data models.
- `sdm/models`: (Pretrained) structured data models based on a common interface.
- `sdm/testing`: Testing utilities.

# Examples

- Build each example around one public contract or capability. Include only the setup, validation, abstraction, and explanation needed to understand and run it correctly, including required leakage boundaries and other correctness constraints.
- Express the workflow through the simplest documented public API. Pass accepted input forms directly, rely on public normalization and semantic helpers, use a one-shot call when fitted state is not reused, and use separate fit and predict steps when the lifecycle or repeated queries are part of the example.
- Present the end-to-end data flow in execution order. Use semantic container operations, name intermediates that identify meaningful stages, keep short conventional values close to their consumers, and use comments only for non-obvious semantics or navigation.
- Represent a self-contained workflow as one descriptively named script. Use functions for genuine repetition, separate modules for distinct responsibilities, and a README for substantial setup or operational instructions.

# Core Design Principles

- Preserve dataframe ergonomics at the boundary, but move model execution onto structured tensor containers.
- Keep model-family wrappers thin. Shared abstractions should live outside model implementations if possible.
- Avoid mandatory config-first APIs. Direct Python composition should be the primary interface.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/structured-data-models](https://github.com/NVIDIA/structured-data-models) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
