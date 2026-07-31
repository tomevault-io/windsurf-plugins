---
trigger: always_on
description: - [ ] Follow test suite categorization and test matrix in
---

# Instructions for AI agents

## PyThaiNLP specific

- [ ] Follow test suite categorization and test matrix in
      <https://github.com/PyThaiNLP/pythainlp/blob/dev/tests/README.md>.
      The document list test categories, their dependency sets,
      and test naming conventions.
- [ ] Use reStructuredText for docstring (PEP 287), targeting Sphinx.
- [ ] When possible, follow NLTK established convention of submodule
      name (tend to be a verb or a generic noun), function name, and
      configuration. Communicate this to the users during code review.
      See <https://www.nltk.org/py-modindex.html>.
- [ ] The type information analyzer at
      <https://github.com/PyThaiNLP/pythainlp/blob/dev/build_tools/analysis/type-analyzer.py>
      can generate information about annotation completeness of
      variables, functions, methods, type aliases, decorators, and
      classes in the PyThaiNLP repo.
      Use it to assist the maintenance of 100% type completeness in
      the repo. Read its usage and information it generates at
      <https://github.com/PyThaiNLP/pythainlp/blob/dev/build_tools/analysis/README.md>.
      Mind that the analyzer can create false positives,
      please refer to Python type specification when in doubt.
- [ ] Complete type annotations for function, method, class, variable, etc.
      Maintain near-100% type annotation coverage.
- [ ] Add tests for new functionality or behavior.
      New PR must not drop the test coverage more than 0.1%.
- [ ] Keep the test coverage high. Aim at least 70% test coverage.
- [ ] Add test cases to cover all code branches and capture edge cases.
- [ ] `# type: ignore[arg-type]` comment can be used in the test code,
      only if that specific code want to explicitly test type handling
      or TypeError raising.
- [ ] Docstring and doctest must reflect the latest code.
- [ ] All error messages and warning messages should be clear, concise,
      and consistent in style. They should be parseable.
- [ ] API documentation is in docs/api/.
      There must be an .rst file for each module, so that the generated
      module API documentation is visible publicly.
- [ ] Major changes should be logged in the change log at
      <https://github.com/PyThaiNLP/pythainlp/blob/dev/CHANGELOG.md>.
      Provide issue number or PR number if available.
- [ ] Do not use os.path.join();
      always use pythainlp.tools.safe_path_join() instead,
      to prevent path traversal vulnerabilities (CWE-22).
- [ ] Naming conventions: Follows PEP 8. Concise. Use US spelling.
      Align new modules, classes, public APIs, and environment
      variables with NLTK conventions as the primary standard,
      provided they suit the component's behavior.
      If NLTK offers no clear precedent, defer to established NLP
      frameworks in the following order of preference:
      spaCy, CoreNLP/Stanza, LangPipe, and Hugging Face.
- [ ] Noun number consistency: Maintain strict intentionality
      regarding singular vs. plural forms. Use singular names for
      classes representing a single entity and reserve plural
      names only for collections, utility modules, or clear aggregates.

## Project contribution guidelines

- [ ] Follow the project's established coding style and conventions.
- [ ] Run Ruff and fix errors before committing code.
  - [ ] New code should be written to pass all Ruff checks.
  - [ ] McCabe complexity should be kept low;
        refactor the new code that exceeds 10.
  - [ ] Cognitive complexity should be kept low;
        refactor the new code that exceeds 15.
  - [ ] Existing code should be gradually improved to pass Ruff checks
        when making changes.
- [ ] Write clear and concise commit messages that accurately describe
      the changes made.
- [ ] For significant changes, update the CHANGELOG.md file
      to document the changes.
  - [ ] Follow "Keep a Changelog" style guide
        <https://keepachangelog.com/en/1.1.0/>
  - [ ] Use semantic versioning for version numbers
        <https://semver.org/>
  - [ ] If it is a breaking change, indicate it clearly in the changelog.
    - [ ] Provide migration instructions if necessary.
- [ ] Do not leave trailing whitespaces in the code or documentation files,
      unless such a whitespace is explicitly necessary.
- [ ] Metadata in pyproject.toml, codemeta.json, CITATION.cff, and other
      project metadata files should be consistent and up-to-date.
  - [ ] Project name
  - [ ] Project version
  - [ ] Author/contributor names
  - [ ] License information
  - [ ] Project description
  - [ ] Repository URL
  - [ ] Keywords/tags (in the same order if possible)

## General language use

- [ ] Write short and simple comments. Do not state the obvious.
- [ ] Prefer clear, concise, and unambiguous sentences.
- [ ] Do not use jargon, slang, or idiomatic expressions
      that may not be universally understood.
- [ ] Use active voice whenever possible.
- [ ] Use consistent terminology throughout the code and documentation.
- [ ] Define acronyms and abbreviations on their first use.
- [ ] Use technical terms accurately and appropriately.
- [ ] Avoid unnecessary complexity and verbosity.
- [ ] Use proper grammar, punctuation, and spelling.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PyThaiNLP/pythainlp](https://github.com/PyThaiNLP/pythainlp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
