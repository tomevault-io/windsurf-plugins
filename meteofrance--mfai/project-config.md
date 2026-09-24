---
trigger: always_on
description: This is the `mfai` (Météo-France AI) librarie, dedicated to sharing bricks of code to people working on meteorology and AI.
---

# AGENTS.md

## Context

This is the `mfai` (Météo-France AI) librarie, dedicated to sharing bricks of code to people working on meteorology and AI.

Our main features are:
- Multiple neural networks architectures in pure Pytorch for vision, segmentation and LLMs.
- Pytorch Lightning modules
- NamedTensors : an extension of torch Tensors with named feature fields

## Stack
 - Python >= 3.10 
 - Pytorch
 - Pytorch Lightning
 

## Chat answer guidelines

These instructions apply to LLM agents answering to the user's chat prompts.

- Be respectful.
- Be concise, do not give examples unless prompted to. Short answers.
- Do not over explain, assume user expertise and give precise explanations when prompted.
- If a code example is requested, answer only with the code, no explanations.
- When making asumptions about the project's environment, check the `pyproject.toml` file.

## Python coding guidelines

These instructions apply to LLM agents writing or editing Python code in this repository.

### Keep it simple

- Prefer the simplest solution that works. Do not over-engineer.
- Do not add abstractions, base classes, or indirection unless the code actually needs them.
- Solve the problem at hand; avoid speculative generality and unused features.

### Follow best practices

- Follow PEP 8 style and the conventions already used in the surrounding code.
- Use type hints on all function signatures and public variables.
- Prefer the standard library and Python 3 idioms (e.g. f-strings, `pathlib`, dataclasses) over third-party utilities where the stdlib suffices.
- Write clear, descriptive names for variables, functions, and classes.
- Keep functions short and focused on a single responsibility.
- Do not add emojis.
- Accept no compromise with security vulnerabilities, prefer not writing code than code with a vulnerability.
- If you propose the usage of third party package, check their cybersecurity status (known vulnerabilities, use appropriate versions, etc)
- Warn the user if you think there is a cybersecurity risk with either the code you propose or the problem you are asked to resolve 
- Keep line length shorter or equal to 88 characters.
- Write code following the structure double line break, comment, code pragraph. Like so:
```py

# Comment explaining what
my = python_code()
can = be_multiple_lines()  # Comment explaining why

# Second paragraph separated by a double line break
_with = the_next_python_code_paragraph()
```

### Respect dev context
You are writting code in the mfai library, wich should be compatible for all versions of python >= 3.10. Keep it easy to maintain. This libairy is unit tested with a coverage > 85 %. When introducing new features, ensure it is tested.


### Post contribution rules
- After any modification to a Python file, load the `python-code-formating` skill and
  run `uvx ruff@0.15.20 format` then `uvx ruff@0.15.20 check --fix` on the changed files.
- Load the `type-checking` skill and run `uv run pyright`.
- Fix any errors the check raises, then re-run both until no errors are raised.
- Load the `unit-test` skill and run the relevant tests to validate the changed code.

### Python writing rules

#### 1. Docstrings (Google style)

- Add a docstring to every public module, class, and function.
- Format docstrings following Google's syntax:

```
"""Summary, that can
be multiline.

Args:
    arg_name: Description of the argument.

Returns:
    return_type: Description of the return value.
"""
```

- Start with a summary that can be multiline ending in a period.
- Add a blank line after the summary when there is more content.
- Use an `Args:` section with one indented `name: description.` line per parameter. Describe what each parameter is or does; do not repeat its type in the description (types come from annotations).
- Add a `Returns:` section describing the return value. Use `Yields:` for generators.
- Omit `Args`/`Returns` sections only when there are no parameters or return value.
- Do not use `:data:` / `:func:` / `:class:` or other Sphinx references.

#### 2. Prefer explicit checks over exceptions (LBYL)

Check conditions proactively before acting. Do not use exceptions for normal control flow
(Look Before You Leap, LBYL). Reserve exceptions for error boundaries, third-party APIs
that offer no alternative, or re-raising with added context.

```py
# WRONG: exception as control flow
try:
    value = mapping[key]
    process(value)
except KeyError:
    pass

# CORRECT: check first
if key in mapping:
    value = mapping[key]
    process(value)
```

#### 3. Never swallow exceptions

Let failures propagate instead of catching and ignoring them. Never use bare `except:`
or catch broad exceptions and discard them silently, as hidden failures are hard to
debug and surface far from their root cause.

```py
# WRONG: silent exception swallowing
try:
    risky_operation()
except:
    pass

# CORRECT: let exceptions bubble up
risky_operation()
```

#### 4. Keep magic methods O(1)

`__len__`, `__bool__`, `__contains__`, and properties are called frequently and
implicitly. Implement them in constant time and avoid I/O or expensive computation.

```py
# WRONG: O(n) __len__
def __len__(self) -> int:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meteofrance/mfai](https://github.com/meteofrance/mfai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
