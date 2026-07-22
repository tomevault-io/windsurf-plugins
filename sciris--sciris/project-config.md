---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Publication Information

**Citation**: Kerr CC, Sanz-Leon P, Abeysuriya RG, et al. Sciris: Simplifying scientific software in Python. *Journal of Open Source Software*. 2023;8(88):5076. doi:10.21105/joss.05076

**Key Applications**: Used by Starsim (general-purpose disease modeling), Covasim, Optima HIV, Optima Nutrition, Atomica, and other scientific software tools adopted in 30+ countries.

## Quick Reference

### Essential Commands
- **Run tests:** `cd tests && ./run_tests`
- **Install dev:** `pip install -e .`
- **Build docs:** `cd docs && ./build_docs`
- **Standard import:** `import sciris as sc`

### Core Functions
- **Containers:** `sc.odict()`, `sc.dcp()` (deep copy)
- **I/O:** `sc.save()`, `sc.load()`, `sc.savejson()`, `sc.loadjson()`
- **Arrays:** `sc.toarray()`, `sc.findinds()`, `sc.findnearest()`
- **Parallel:** `sc.parallelize()`, `sc.loadbalancer()`
- **Plotting:** `sc.boxoff()`, `sc.commaticks()`, `sc.dateformatter()`
- **Timing:** `sc.tic()`, `sc.toc()`, `sc.profile()`

### Key Parameters
- **`die`**: Exception handling (`die=True` raises, `die=False` warns)
- **`verbose`**: Output control (`verbose=True/False/None`)
- **`copy`**: Data copying behavior for containers

## Development Commands

### Testing
- **Install test requirements first:** `pip install -r tests/requirements.txt` (adds pytest, pytest-cov, pytest-xdist)
- **Run all tests:** `cd tests && ./run_tests` (uses pytest with parallel execution via pytest-xdist)
- **Run specific test:** `cd tests && pytest test_<module>.py -v`
- **Check test coverage:** `cd tests && ./check_coverage` (generates HTML report at tests/htmlcov/index.html)
- **Test downstream packages:** `cd tests/regression && ./run_downstream` (tests covasim, hpvsim, fpsim, atomica)
- **Environment:** Tests run with `SCIRIS_BACKEND=agg` to prevent matplotlib windows

### Code Quality
- **Linting:** Uses pylint with custom .pylintrc; `cd tests && ./check_style` (generates text report at tests/pylint_report.txt)
- **Type checking:** Do not use type hints (Sciris prioritizes flexibility over strict typing)
- **Pre-commit hooks:** Not configured
- **Style guide:** See https://github.com/starsimhub/styleguide/blob/main/README.rst

### Documentation
- **Build docs:** `cd docs && ./build_docs` (builds Sphinx documentation)
- **Build docs without notebooks:** `cd docs && ./build_docs never`
- **Debug mode (serial):** `cd docs && ./build_docs debug`
- **Output:** Documentation built to `docs/_build/html/index.html`

### Development Setup
- **Install package:** `pip install -e .` (editable install from pyproject.toml)
- **Test requirements:** `pip install -r tests/requirements.txt`
- **Python version:** Requires Python >=3.9

### Git Workflow
- **Main branch:** `main` (use this as base for PRs)
- **Feature branches:** Create descriptive branch names (e.g., `update-profiler`)

### Continuous Integration
- **Main test workflow:** `.github/workflows/test_sciris.yaml` (runs on push/PR)
- **Scheduled tests:** `.github/workflows/schedule_tests.yaml` (periodic testing)
- **Downstream tests:** `.github/workflows/test_downstream.yaml` (tests dependent packages)
- **Release workflow:** `.github/workflows/pypi_release.yaml` (automated PyPI publishing)

## Claude Code Plugin

Sciris includes a Claude Code skills plugin in `claude_plugin/` that provides domain-specific skills for working with Sciris. The plugin is registered via `.claude-plugin/marketplace.json`.

### Installation
Users can install the plugin by adding `https://github.com/sciris/sciris` as a Claude Code marketplace (see `/install-plugin`).

### Available Skills
The plugin provides 10 skills covering all major Sciris domains:
- **sciris-intro** - Basic Sciris features and quick reference
- **sciris-arrays** - Array operations (`sc.findinds()`, `sc.findnearest()`, `sc.toarray()`, etc.)
- **sciris-dicts** - Containers (`sc.odict()`, `sc.objdict()`, `sc.counter()`, etc.)
- **sciris-files** - File I/O (`sc.save()`, `sc.load()`, `sc.savejson()`, etc.)
- **sciris-plotting** - Matplotlib extensions (`sc.boxoff()`, `sc.dateformatter()`, etc.)
- **sciris-parallel** - Parallelization (`sc.parallelize()`, `sc.loadbalancer()`)
- **sciris-dates** - Date/time handling (`sc.date()`, `sc.daterange()`, `sc.timer()`)
- **sciris-printing** - Printing and formatting (`sc.heading()`, `sc.sigfig()`, `sc.progressbar()`)
- **sciris-utils** - Utilities (`sc.mergedicts()`, `sc.tryexcept()`, `sc.checkmem()`, etc.)
- **sciris-advanced** - Advanced features (`sc.odict` subclassing, nested operations, versioning)

### MCP Servers
The plugin also configures Context7 and GitMCP servers for accessing up-to-date Sciris documentation.

### Plugin Structure
```
claude_plugin/
  .claude-plugin/plugin.json    # Plugin manifest
  skills/                       # Skill definitions (one SKILL.md per skill)
.claude-plugin/
  marketplace.json              # Marketplace registration
```

## Architecture Overview


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sciris/sciris](https://github.com/sciris/sciris) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
