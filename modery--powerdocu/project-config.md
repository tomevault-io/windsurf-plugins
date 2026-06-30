---
trigger: always_on
description: PowerDocu is a .NET 10 Windows application that auto-generates technical documentation for Microsoft Power Platform components (Cloud Flows, Canvas Apps, Model-Driven Apps, Copilot Studio Agents, AI Models, Business Process Flows, Desktop Flows, and Solutions). Output formats are **Word (.docx)**, **Markdown (.md)**, and **HTML**.
---

# CLAUDE.md — PowerDocu Codebase Guide

## Project Overview

PowerDocu is a .NET 10 Windows application that auto-generates technical documentation for Microsoft Power Platform components (Cloud Flows, Canvas Apps, Model-Driven Apps, Copilot Studio Agents, AI Models, Business Process Flows, Desktop Flows, and Solutions). Output formats are **Word (.docx)**, **Markdown (.md)**, and **HTML**.

The application ships as a **Windows GUI** (`PowerDocu.exe`) and also supports a **CLI** interface via command-line flags.

---

## Repository Structure

```
PowerDocu/
├── PowerDocu.sln                      # Visual Studio solution (all projects)
├── modules/
│   └── PowerDocu.Common/              # Git submodule (https://github.com/modery/PowerDocu.Common)
│       └── PowerDocu.Common/          # The actual library project
├── PowerDocu.GUI/                     # Entry point: WinForms GUI + CLI runner
├── PowerDocu.SolutionDocumenter/      # Orchestrator for .zip solution packages
├── PowerDocu.FlowDocumenter/          # Cloud Flow documentation
├── PowerDocu.AppDocumenter/           # Canvas App documentation
├── PowerDocu.AgentDocumenter/         # Copilot Studio Agent documentation
├── PowerDocu.AIModelDocumenter/       # AI Model documentation
├── PowerDocu.AppModuleDocumenter/     # Model-Driven App documentation
├── PowerDocu.BPFDocumenter/           # Business Process Flow documentation
├── PowerDocu.DesktopFlowDocumenter/   # Desktop Flow (Power Automate Desktop) documentation
├── Images/                            # Screenshots used in README/docs
├── examples/                          # Sample generated output
├── .vscode/                           # VS Code build/launch tasks
├── .github/ISSUE_TEMPLATE/            # GitHub issue templates
├── README.md
├── compile.md                         # How to build from source
├── installation.md                    # Usage instructions
├── settings.md                        # All configuration options
├── roadmap.md                         # Planned features
└── softwarereferences.md              # Third-party library credits
```

---

## Submodule: PowerDocu.Common

**Location:** `modules/PowerDocu.Common/PowerDocu.Common/`  
**Source:** https://github.com/modery/PowerDocu.Common  
**Target:** `net10.0`

After cloning, initialize the submodule:
```bash
git submodule init
git submodule update
```

### Key contents of PowerDocu.Common

**Entity classes** (data models):
- `FlowEntity`, `AppEntity`, `AgentEntity`, `AppModuleEntity`, `AIModel`, `BPFEntity`, `DesktopFlowEntity`
- `SolutionEntity`, `CustomizationsEntity`, `TableEntity`, `ControlEntity`
- `EnvironmentVariableEntity`, `WebResourceEntity`, `OptionSetEntity`, `FormulaDefinitionEntity`

**Parser classes** (extract entities from Power Platform packages):
- `FlowParser`, `AppParser`, `AgentParser`, `SolutionParser`, `CustomizationsParser`
- `BPFXamlParser`, `RobinScriptParser`, `AppActionParser`, `SettingDefinitionParser`, `EnvironmentVariableParser`

**Base builder classes** (shared output generation):
- `WordDocBuilder` — base for all Word document builders
- `MarkdownBuilder` — base for all Markdown builders
- `HtmlBuilder` — base for all HTML builders

**Helper/utility classes**:
- `ConfigHelper` — configuration model (maps to `powerdocu.config.json`)
- `DocumentationContext` — shared state passed across the two-phase pipeline
- `NotificationHelper` — event-driven notification/logging system
- `ConnectorHelper` — connector icon management
- `CharsetHelper` — safe filename/path generation
- `OutputFormatHelper` — output format constants (`Word`, `Markdown`, `Html`, `All`)
- `ProgressTracker` — tracks documentation progress across parallel tasks
- `CrossDocLinkHelper` — cross-document hyperlink resolution
- `ZipHelper`, `JsonUtil`, `YamlDotNet` — file parsing utilities

**NuGet dependencies** (all managed in `PowerDocu.Common.csproj`):
| Package | Version | Purpose |
|---|---|---|
| `DocumentFormat.OpenXml` | 3.5.1 | Word document generation |
| `Grynwald.MarkdownGenerator` | 3.0.106 | Markdown file creation |
| `Newtonsoft.Json` | 13.0.4 | JSON parsing |
| `Rubjerg.Graphviz` | 3.0.4 | Flow/app diagram generation (requires Graphviz) |
| `Svg` | 3.4.7 | SVG to PNG conversion |
| `HtmlAgilityPack` | 12.4 | HTML parsing |
| `System.Drawing.Common` | 10.0.5 | Drawing utilities |
| `Microsoft.PowerFx.Core` | 1.8.1 | Power Fx expression parsing |
| `YamlDotNet` | 16.3.0 | YAML parsing (agent definitions) |

---

## Project Dependency Graph

```
PowerDocu.GUI (WinExe, net10.0-windows)
  └── PowerDocu.FlowDocumenter
  └── PowerDocu.AppDocumenter
  └── PowerDocu.SolutionDocumenter
        └── PowerDocu.FlowDocumenter
        └── PowerDocu.AppDocumenter
        └── PowerDocu.AgentDocumenter
        └── PowerDocu.AppModuleDocumenter
        └── PowerDocu.AIModelDocumenter
        └── PowerDocu.BPFDocumenter
        └── PowerDocu.DesktopFlowDocumenter

All documenter projects → PowerDocu.Common (submodule)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [modery/PowerDocu](https://github.com/modery/PowerDocu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
