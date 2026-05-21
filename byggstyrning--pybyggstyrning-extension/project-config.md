---
trigger: always_on
description: Organizing your PyRevit extension code is crucial for maintainability. PyRevit provides an **extension** architecture that encourages modular code separation. Follow these guidelines:
---

# PyRevit Development Best Practices

## 1. Code Structure

Organizing your PyRevit extension code is crucial for maintainability. PyRevit provides an **extension** architecture that encourages modular code separation. Follow these guidelines:

- **Use Extensions (Don’t Modify Core)**  
  Keep your scripts in a separate `.extension` folder rather than altering the core pyRevit tools. This ensures your custom scripts remain safe from pyRevit updates and can be shared independently.

- **Bundle Commands Properly**  
  Structure each tool as a command bundle (a folder ending in `.pushbutton`). Inside that, include a `script.py` (and optionally a `config.py` for additional options). This bundle approach organizes commands logically and helps avoid a single giant script.

- **Shared Code in `lib` Folder**  
  For common functions or classes, use a `lib/` subfolder in your extension or panel bundle. PyRevit adds the bundle’s `lib` directory to the search path, so you can import modules easily. This promotes modularity and code reuse.

- **Follow PyRevit Script Conventions**  
  At the top of each script, define metadata like `__doc__`, `__title__`, and `__author__`. These become tool tooltips and labels in Revit. Use `__context__` if needed to control command availability.

- **Prefer PyRevit APIs and Utilities**  
  Use pyRevit’s provided modules (e.g. `pyrevit.script`, `pyrevit.forms`, etc.) for common tasks instead of directly accessing Revit APIs. Consistent use of these helpers enhances code clarity.

- **Version Control and Organization**  
  Keep your extension in a version-controlled repository (e.g., Git) with a clear hierarchy: one folder per extension, organized by tabs/panels and then command bundles. Document your tools with a README for setup and usage instructions.

## 2. Debugging Best Practices

Effective debugging in IronPython for PyRevit is essential since traditional debuggers are hard to attach. Adopt these strategies:

- **Use Logging for Insight**  
  Instrument your scripts with logging calls using `pyrevit.script.get_logger()`. Use `logger.debug()`, `info()`, `warning()`, etc., to trace execution and identify issues. Toggle debug mode when necessary.

- **Prints and PyRevit Output**  
  Supplement logging with `print()` statements when needed. This simple approach can help validate critical variables or workflow checkpoints, especially during early development.

- **Avoid Relying on Interactive Debuggers**  
  Since attaching Visual Studio or VS Code is not straightforward in this environment, use a test-driven approach: isolate complex logic outside Revit or use RevitPythonShell to run snippets interactively.

- **Granular Exception Handling**  
  Wrap only small code segments in `try/except` blocks. Catch specific exceptions (e.g., `KeyError`, Revit API exceptions) to provide meaningful error messages rather than a blanket `except:`.

- **Defensive Programming**  
  Proactively check conditions (e.g., user selections, list lengths) to prevent errors instead of solely relying on error handling. This makes your code more robust and predictable.

- **Fail Gracefully (but Not Silently)**  
  Handle exceptions in a way that informs the user (e.g., using TaskDialog or `forms.alert`) instead of just logging them. This improves the user experience by providing clear feedback.

- **Leverage PyRevit Transactions**  
  When modifying the Revit document, use the `pyrevit.revit.Transaction` context manager. This ensures that if an error occurs, the transaction is rolled back, leaving Revit in a stable state.

- **Iterative Testing**  
  Develop and test your scripts on small examples or dummy models to isolate issues early. This minimizes the risk of encountering basic logic errors in the full Revit environment.

## 3. WPF and IronPython Integration

Creating rich WPF UIs in PyRevit requires careful handling of the UI thread and event management. Follow these best practices:

- **Separate UI Layout with XAML**  
  Define your WPF UI in a XAML file and load it in IronPython. This keeps UI design declarative and your Python code focused on event handling.

- **Use PyRevit Forms Framework**  
  Leverage `pyrevit.forms` utilities (e.g., subclassing `forms.WPFWindow` or `forms.WPFPanel`) for common UI tasks. This framework handles XAML loading and event wiring, reducing boilerplate code.

- **Modal vs. Modeless Windows**  
  Decide if your window should be modal (blocking Revit until closed) or modeless (allowing interaction with Revit). For modeless windows, do not call Revit API methods directly from the UI; use the ExternalEvent pattern instead.

- **Use ExternalEvent for Revit Actions**  
  In modeless scenarios, any Revit API action should be executed via an ExternalEvent and IExternalEventHandler. This pattern safely delegates actions from the UI thread to Revit’s main thread.

- **UI Performance Considerations**  
  Avoid overly complex WPF visuals and heavy data bindings in IronPython. Offload heavy computations or data loading (e.g., using background threads with `Dispatcher.Invoke` for UI updates) to maintain a responsive interface.

- **Closing and Cleanup**  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [byggstyrning/pyByggstyrning.extension](https://github.com/byggstyrning/pyByggstyrning.extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
