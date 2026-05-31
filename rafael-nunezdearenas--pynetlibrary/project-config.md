---
trigger: always_on
description: PyNET supports two hosts: **Autodesk Navisworks** and **Autodesk Revit**. Both execute Python scripts via **Python.NET** (CPython 3.10+ with `pythonnet` — not IronPython). Full Python 3 syntax is supported, along with the `clr` bridge to access .NET and Autodesk APIs.
---

# Project Context — PyNET Platform (Navisworks & Revit)

## 1. Execution Environment

PyNET supports two hosts: **Autodesk Navisworks** and **Autodesk Revit**. Both execute Python scripts via **Python.NET** (CPython 3.10+ with `pythonnet` — not IronPython). Full Python 3 syntax is supported, along with the `clr` bridge to access .NET and Autodesk APIs.

Scripts are sent to the plugin through the MCP bridge and executed locally inside the host process. Always check `list_active_instances` first to identify which host is running (Navisworks or Revit) and its PID — the boilerplate and available APIs differ between hosts.

> **Timeout rule:** Always use a minimum timeout of **60 seconds** when calling `send_command`. Scripts can take longer than expected depending on model size.

> **MCP bridge version:** Run `pip show pynet-mcp-bridge` (NOT `pynet-bridge`) using the Python 3.10 pip at `C:\Users\34655\AppData\Local\Programs\Python\Python310\Scripts\pip.exe`. Current installed version: **1.4.7**.

---

## 2. Execution Responses

### Scenario A: Success

```json
{
  "ScriptName": "SayHi_AI_Running",
  "ExecutionDate": "2026-03-30T15:42:12.8640716+02:00",
  "Status": "Success",
  "Message": "Script executed successfully",
  "PrintMessages": [
    "Hello Pablo"
  ],
  "Duration": "00:00:00.0312442",
  "Data": []
}
```

### Scenario B: Error (PythonException with StackTrace)
This is the format received when Python code fails. Analyze `Message` to auto-correct:

```json
{
  "ScriptName": "SayHi_AI_Running",
  "ExecutionDate": "2026-03-30T15:52:56.1204577+02:00",
  "Status": "Error",
  "Message": "Python.Runtime.PythonException: '(' was never closed (<string>, line 2)...",
  "PrintMessages": [],
  "Duration": "00:00:00",
  "Data": []
}
```

---

## 3. Script Output

Scripts can return structured data via a global variable that the plugin collects automatically.

### Output Variable

The variable name must be:

```
ia_Result
```

The system will look for this variable after script execution. If it does not exist, no data output will be generated.

### Print vs ia_Result

The Navisworks plugin has a visible **Output Window** where every `print()` is displayed to the user in real time. This means:

- **During iterative development** (scripts sent via `send_command`): use `ia_Result` as the primary channel to return structured data back to the AI. Keep `print` usage minimal — only for brief status messages (e.g. `"Found 41 wall types"`). Do NOT flood the Output Window with per-element or per-iteration prints.
- **When saving a script for the user** (deployed to a button or saved to source): add meaningful `print` statements so the user gets clear feedback when running the script manually (progress, results summary, confirmations). In this case, `ia_Result` is optional since the user reads the Output Window directly.

In short: `ia_Result` is for AI consumption, `print` is for user consumption. During development keep prints quiet; in final saved scripts make them informative.

Do not abbreviate output values or apply any transformation unless explicitly requested.

### Data Format

`ia_Result` must contain a JSON-serializable structure, typically:

- a list of dicts
- or a single dict

Recommended example (list of items):

```python
ia_Result = [
    {
        "type": "Wall",
        "id": 1,
        "name": "Wall A",
        "height": 3.2
    }
]
```

### Important Rules

- Data must be serializable (numbers, strings, lists, dicts)
- Never return complex Python objects (classes, API references, etc.)
- Always convert objects to `dict` before returning them
- Always include a `"type"` field in each object for easy interpretation
- Maintain a consistent structure across scripts

---

## 4. Security & Execution Restrictions

All scripts are validated by a static analyzer before execution. The scope is strictly limited to **Autodesk Navisworks automation** — no file system access, no network operations, no system-level actions.

### Allowed CLR Assemblies
Only these .NET references are permitted via `clr.AddReference`:
- `Autodesk.Navisworks.Api`, `.ComApi`, `.Interop.ComApi`, `.Clash`
- `System`, `System.Windows.Forms`, `System.Drawing`, `System.Collections.Generic`
- `Raen.Core.Pynet.*`, `Raen.Navisworks.Pynet.*` (any version)

Any other assembly will be rejected.

### Allowed Python Imports
`clr`, `sys`, `json`, `re`, `time`, `datetime`, `pathlib`, `typing`, `threading`, `collections`, `xml`, `pandas`, `plotly`, `matplotlib`, `dash`, `webbrowser`, `psutil`, `openpyxl`

> **Note (Revit):** `openpyxl` requires bridge **≥ 1.4.7** — it was not whitelisted in 1.4.6.

### Blocked Python Imports
`os`, `subprocess`, `shutil`, `socket`, `ctypes`, `pickle`, `importlib`, `http`, `urllib`, `signal`, `multiprocessing`, `tempfile`, `glob`, `inspect`, `code`, `codeop`

### Blocked Calls
`eval`, `exec`, `compile`, `__import__`, `getattr`, `setattr`, `delattr`, `globals`, `locals`, `vars`, `breakpoint`

### Blocked Attribute Access
`__builtins__`, `__subclasses__`, `__globals__`, `__code__`

### Execution Confirmation Policy

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rafael-NunezDeArenas/PyNetLibrary](https://github.com/Rafael-NunezDeArenas/PyNetLibrary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
