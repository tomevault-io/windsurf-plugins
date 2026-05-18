---
trigger: always_on
description: Use it to get full context on fastmcp
---

├── .github
    ├── ai-labeler.yml
    ├── release.yml
    └── workflows
    │   ├── ai-labeler.yml
    │   ├── publish.yml
    │   ├── run-static.yml
    │   └── run-tests.yml
├── .gitignore
├── .pre-commit-config.yaml
├── .python-version
├── LICENSE
├── README.md
├── Windows_Notes.md
├── docs
    └── assets
    │   └── demo-inspector.png
├── examples
    ├── complex_inputs.py
    ├── desktop.py
    ├── echo.py
    ├── memory.py
    ├── readme-quickstart.py
    ├── screenshot.py
    ├── simple_echo.py
    └── text_me.py
├── pyproject.toml
├── src
    └── fastmcp
    │   ├── __init__.py
    │   ├── cli
    │       ├── __init__.py
    │       ├── claude.py
    │       └── cli.py
    │   ├── exceptions.py
    │   ├── prompts
    │       ├── __init__.py
    │       ├── base.py
    │       ├── manager.py
    │       └── prompt_manager.py
    │   ├── py.typed
    │   ├── resources
    │       ├── __init__.py
    │       ├── base.py
    │       ├── resource_manager.py
    │       ├── templates.py
    │       └── types.py
    │   ├── server.py
    │   ├── tools
    │       ├── __init__.py
    │       ├── base.py
    │       └── tool_manager.py
    │   └── utilities
    │       ├── __init__.py
    │       ├── func_metadata.py
    │       ├── logging.py
    │       └── types.py
├── tests
    ├── __init__.py
    ├── prompts
    │   ├── __init__.py
    │   ├── test_base.py
    │   └── test_manager.py
    ├── resources
    │   ├── __init__.py
    │   ├── test_file_resources.py
    │   ├── test_function_resources.py
    │   ├── test_resource_manager.py
    │   ├── test_resource_template.py
    │   └── test_resources.py
    ├── servers
    │   ├── __init__.py
    │   └── test_file_server.py
    ├── test_cli.py
    ├── test_func_metadata.py
    ├── test_server.py
    └── test_tool_manager.py
└── uv.lock


/.github/ai-labeler.yml:
--------------------------------------------------------------------------------
 1 | instructions: |
 2 |   Apply the minimal set of labels that accurately characterize the issue/PR:
 3 |   - Use at most 1-2 labels unless there's a compelling reason for more. It's ok to use no labels.
 4 |   - Prefer specific labels (bug, feature) over generic ones (question, help wanted)
 5 |   - For PRs that fix bugs, use 'bug' not 'enhancement'
 6 |   - Never combine: bug + enhancement, feature + enhancement. For these labels, only choose the most relevant one.
 7 |   - Reserve 'question' and 'help wanted' for when they're the primary characteristic
 8 | 
 9 | labels:
10 |   - bug:
11 |     description: "Something isn't working as expected"
12 |     instructions: |
13 |       Apply when describing or fixing unexpected behavior:
14 |       - Issues: Clear error messages or unexpected outcomes
15 |       - PRs: Standalone fixes for broken functionality or closing bug reports.
16 |       Don't apply bug unless the issue or PR is predominantly about a specific bug.
17 | 
18 |   - documentation:
19 |     description: "Improvements or additions to documentation"
20 |     instructions: |
21 |       Apply only when documentation is the primary focus:
22 |       - README updates
23 |       - Code comments and docstrings
24 |       - API documentation
25 |       - Usage examples
26 |       Don't apply for minor doc updates alongside code changes
27 | 
28 |   - enhancement:
29 |     description: "Improvements to existing features"
30 |     instructions: |
31 |       Apply only for improvements to existing functionality:
32 |       - Performance improvements
33 |       - UI/UX improvements
34 |       - Expanded capabilities of existing features
35 |       Don't apply to:
36 |       - Bug fixes
37 |       - New features
38 |       - Minor tweaks
39 | 
40 |   - feature:
41 |     description: "New functionality"
42 |     instructions: |
43 |       Apply only for net-new functionality:
44 |       - New API endpoints
45 |       - New commands or tools
46 |       - New user-facing capabilities
47 |       Don't apply to:
48 |       - Improvements to existing features (use enhancement)
49 |       - Bug fixes
50 | 
51 |   - good first issue:
52 |     description: "Good for newcomers"
53 |     instructions: |
54 |       Apply very selectively to issues that are:
55 |       - Small in scope
56 |       - Well-documented
57 |       - Require minimal context
58 |       - Have clear success criteria
59 |       Don't apply if the task requires significant background knowledge
60 | 
61 |   - help wanted:
62 |     description: "Extra attention is needed"
63 |     instructions: |
64 |       Apply only when it's the primary characteristic:
65 |       - Issue needs external expertise
66 |       - Current maintainers can't address it
67 |       - Additional contributors would be valuable
68 |       Don't apply just because an issue is open or needs work
69 | 
70 |   - question:
71 |     description: "Further information is requested"
72 |     instructions: |
73 |       Apply only when the primary purpose is seeking information:
74 |       - Clarification needed before work can begin
75 |       - Architectural discussions
76 |       - Implementation strategy questions
77 |       Don't apply to:
78 |       - Bug reports that need more details
79 |       - Feature requests that need refinement
80 | 

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cohnen/mcp-google-ads](https://github.com/cohnen/mcp-google-ads) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
