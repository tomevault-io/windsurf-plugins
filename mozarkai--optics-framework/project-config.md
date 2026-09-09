---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Concrete map of the optics-framework runtime for Claude Code and similar tools. All line numbers below are anchors at the time of writing — if a `path:line` no longer matches the named symbol, fix this file instead of trusting it.

## Execute journey (CLI → driver action)

The full chain when a user types `optics execute <folder>`:

1. `optics_framework/helper/cli.py:390` — `main()` builds argparse subparsers, dispatches via `ExecuteCommand.execute` (`cli.py:319`) → `execute_main(folder_path, runner, use_printer)` in `optics_framework/helper/execute.py:672`.
2. `execute_main` constructs `ExecuteRunner(args)` (`execute.py:660`, subclass of `BaseRunner` at `execute.py:492`) and wraps `asyncio.run(...)`.
3. `BaseRunner.__init__` (`execute.py:495`) runs discovery + loading synchronously:
   - `find_files(folder_path, validate=True)` (`execute.py:54`) walks the dir, sniffs CSV headers / YAML top-level keys via `identify_file_content` (`execute.py:210`) and `_categorize_file_by_content` (`execute.py:140`), routes paths to `test_case` / `module` / `element` / `api` / `error_definitions` buckets. A YAML with both `driver_sources` and `element[s]_sources` is recognised as project config (`_is_config_file`, `execute.py:121`).
   - `_load_test_cases` / `_load_modules` / `_load_elements` / `_load_api_data` (`execute.py:540`–`590`) read each file via `CSVDataReader` / `YAMLDataReader` (`common/runner/data_reader.py:124` / `:249`) and populate `ModuleData` / `ElementData` / `ApiData` (`common/models.py:139`, `:154`, `:278`). `_load_error_definitions` (`execute.py:596`) similarly reads an auto-discovered `error_definitions.csv` via `read_error_definitions` (`data_reader.py:209`) into an `ErrorDefinitions` model (`models.py:308`) — see "On-screen error detection" below.
   - `_load_templates` → `discover_templates` (`execute.py:31`) collects every `.png/.jpg/...` into `TemplateData` (`models.py:293`).
   - `_filter_and_build_execution_queue` → `filter_test_cases` (`execute.py:286`, honours `include`/`exclude`, always keeps setup/teardown) + `build_linked_list` (`execute.py:440`) which threads `TestCaseNode → ModuleNode → KeywordNode` (`models.py:69` / `:34` / `:28`).
   - `_setup_session` → `SessionManager.create_session` (`common/session_manager.py:154`) which builds a `Session` (`session_manager.py:100`): instantiates `EventSDK`, `OpticsBuilder` (`common/optics_builder.py:31`), and via `_get_enabled_config_list` (`session_manager.py:29`) keeps only `DependencyConfig.enabled == True` entries. `Session.__init__` calls `add_driver` / `add_element_source` / `add_text_detection` / `add_image_detection` then `self.optics.get_driver()` to fail fast — `OpticsBuilder.instantiate_driver`/`instantiate_element_source` (`optics_builder.py:115`/`:126`) raise `E0501` when no driver/element-source config is set.
4. `BaseRunner.run("batch")` (`execute.py:633`) constructs `ExecutionParams` and awaits `ExecutionEngine.execute` (`common/execution.py:365`).
5. `ExecutionEngine.execute`:
   - Pulls session-scoped `EventManager` from `get_event_manager(session_id)` (`common/events.py:206`, registry at `:174`) and `.start()`s its dispatch loop.
   - `RunnerFactory.create_runner` (`execution.py:214`) constructs `KeywordRegistry` (`common/runner/keyword_register.py:5`), then via `session.optics.build(cls)` (`optics_builder.py:192`) instantiates `ActionKeyword` (`api/action_keyword.py:206`), `AppManagement` (`api/app_management.py:7`), `Verifier` (`api/verifier.py:11`), `FlowControl` (`api/flow_control.py:41`), and registers each. `KeywordRegistry.register` (`keyword_register.py:22`) walks `dir(instance)` and maps every non-underscore callable into `keyword_map`. Then picks `TestRunner` (`runner/test_runnner.py:98`), `PytestRunner` (`:838`), or `KeywordRunner` (`:1293`) based on `runner_type`.
   - `BatchExecutor.execute` (`execution.py:50`) → `TestRunner.run_all` (`test_runnner.py:807`).
6. `TestRunner.run_all` walks the `TestCaseNode` chain → `_process_test_case` (`:617`) → `_process_module` (`:557`) → `_execute_keyword` (`:319`):
   - Resolves `func_name = "_".join(name.split()).lower()` (`:350`) and looks it up in `keyword_map`.
   - `_build_param_candidates` (`:392`) expands every `${var}` to `ElementData.get_element(var)` — the **fallback list** — and other params to single-element lists.
   - `_try_execute_with_fallback` (`:424`) iterates `itertools.product(*param_candidates)` (capped at `MAX_ATTEMPTS = 20`, `:428`). On each combination it calls the bound method. The ladder advances **only** when the raised `OpticsError.code` starts with `E02` (element-not-found family) or equals `Code.X0201` (`:451`); any other exception is fatal via `_handle_keyword_exception` (`:462`). This is **fallback level 1** (param-axis).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mozarkai/optics-framework](https://github.com/mozarkai/optics-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
