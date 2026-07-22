---
trigger: always_on
description: <!-- mdformat-toc start --slug=github --no-anchors --maxlevel=3 --minlevel=2 -->
---

# AGENTS.md

<!-- mdformat-toc start --slug=github --no-anchors --maxlevel=3 --minlevel=2 -->

- [Project overview](#project-overview)
- [Directory structure](#directory-structure)
  - [Specs](#specs)
  - [Tests](#tests)
- [Important commands](#important-commands)
  - [Linting](#linting)
  - [Running tests](#running-tests)
  - [Cleaning](#cleaning)
- [Writing tests](#writing-tests)
  - [Reference tests vs unittests](#reference-tests-vs-unittests)
  - [Reference test formats](#reference-test-formats)
  - [Test decorators](#test-decorators)
  - [Test pattern](#test-pattern)
- [Common tasks](#common-tasks)
  - [Adding a new helper function](#adding-a-new-helper-function)
  - [Modifying an existing function](#modifying-an-existing-function)
  - [Adding a new container field](#adding-a-new-container-field)
  - [Adding a new fork or feature](#adding-a-new-fork-or-feature)
- [Important notes](#important-notes)
  - [Verify current spec behavior](#verify-current-spec-behavior)
  - [Fork inheritance](#fork-inheritance)

<!-- mdformat-toc end -->

## Project overview

This repository contains the **Ethereum Proof-of-Stake Consensus
Specifications**. It serves as:

- **Formal specifications** in human-readable markdown with embedded Python
- **Executable reference implementation** (Python code generated from markdown)
- **Reference test generator** for client implementations
- **Protocol development platform** organized by network upgrades (forks)

The specifications define how Ethereum's consensus layer (beacon chain)
operates.

## Directory structure

### Specs

```
/specs/
  phase0/     # The genesis specs
  altair/     # The 1st upgrade (starts with A)
  bellatrix/  # The 2nd upgrade (starts with B)
  capella/    # The 3rd upgrade (starts with C)
  ...
  _features/  # Features which have not been scheduled for inclusion
```

### Tests

```
/tests/
  core/pyspec/eth_consensus_specs/
    <fork>/              # Assembled pyspec (do not edit)
    test/<fork>/         # Test cases organized by fork
      block_processing/
      epoch_processing/
      sanity/
      ...
    test/helpers/        # Shared test helpers
      <fork>/            # Fork-specific test helpers
  generators/            # Reference test generators
  formats/               # Test format specifications
```

## Important commands

Everything is done through the Makefile. Run `make help verbose=true` for full
documentation.

### Linting

```bash
make lint
```

This command runs all linters, formatters, and checks for the repository. It
covers Python code style, markdown formatting, table of contents validation, and
spec-specific checks. Always run this before committing to ensure changes meet
the project standards.

### Running tests

See [`.claude/skills/run-tests/SKILL.md`](.claude/skills/run-tests/SKILL.md).

### Cleaning

```bash
make clean
```

This command deletes all untracked files in the repository. Any untracked files
that should be preserved must be staged with `git add` before running this
command.

## Writing tests

### Reference tests vs unittests

**Reference tests** generate test vectors that client implementations use.
Prefer reference tests when possible since they benefit the entire ecosystem.

**Unittests** are internal-only tests that don't produce reference files. Use
these when reference tests are not feasible (e.g., testing internal helpers or
edge cases that do not map to client behavior). Unittests are located in
`unittests` directories.

### Reference test formats

Reference test format specifications are located in `tests/formats/`. These
define the directory and file structure for generated reference tests,
documenting the expected inputs, outputs, and file organization for each test
category (e.g., `operations/`, `sanity/`, `epoch_processing/`). Client
implementations use these specifications to parse and run the reference tests.

### Test decorators

When writing tests, use these decorators:

- `@with_all_phases` - Run on all forks
- `@with_phases([DENEB, FULU])` - Run on specific forks
- `@with_deneb_and_later` - Run on Deneb and all subsequent forks
- `@with_electra_and_later` - Run on Electra and all subsequent forks
- `@spec_state_test` - State transition test
- `@spec_test` - General spec test
- `@always_bls` - Always enable BLS verification

### Test pattern

Tests yield their outputs for reference test generation:

```python
@with_all_phases
@spec_state_test
def test_example(spec, state):
    # Setup
    yield "pre", state

    # Execute
    block = build_empty_block_for_next_slot(spec, state)
    signed_block = state_transition_and_sign_block(spec, state, block)

    yield "blocks", [signed_block]
    yield "post", state
```

## Common tasks

### Adding a new helper function

1. Add the Python function to the appropriate spec markdown file
2. Add tests in `tests/core/pyspec/eth_consensus_specs/test/`
3. Run `make lint` to run checks

### Modifying an existing function

1. Find the function in the spec markdown
2. Make the necessary changes, adding "fork comments" above changed lines
3. Run `make lint` to run checks

### Adding a new container field

1. Add field to container definition in spec markdown
2. Update any functions that construct or use the container

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ethereum/consensus-specs](https://github.com/ethereum/consensus-specs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
