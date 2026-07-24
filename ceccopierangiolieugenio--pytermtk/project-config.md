---
trigger: always_on
description: pyTermTk is a Text-based User Interface (TUI) library for Python inspired by Qt5, GTK, and tkinter APIs. It creates cross-platform terminal applications with rich widgets, layouts, and an event-driven architecture.
---

# pyTermTk Copilot Instructions

## Project Overview

pyTermTk is a Text-based User Interface (TUI) library for Python inspired by Qt5, GTK, and tkinter APIs. It creates cross-platform terminal applications with rich widgets, layouts, and an event-driven architecture.

## Architecture

### Core Structure
- **`libs/pyTermTk/TermTk/`** - Main library code organized into logical modules:
  - `TTkCore/` - Core functionality (signals, colors, canvas, configuration)
  - `TTkWidgets/` - All UI widgets inherit from `TTkWidget` base class
  - `TTkLayouts/` - Layout managers (GridLayout, HBoxLayout, VBoxLayout)
  - `TTkGui/` - GUI components (drag & drop, application management)
  - `TTkTemplates/` - Mixin classes for event handling (`TKeyEvents`, `TMouseEvents`, `TDragEvents`)

### Widget Inheritance Pattern
All widgets follow this pattern:
```python
class TTkMyWidget(TTkWidget):  # Or TTkContainer for composite widgets
    # Class-level styling
    classStyle = {
        'default':  {'color': TTkColor.fg("#dddd88"), 'borderColor': TTkColor.RST},
        'hover':    {'color': TTkColor.fg("#ffffff"), 'borderColor': TTkColor.BOLD},
        'focus':    {'borderColor': TTkColor.fg("#ffff00")},
        'disabled': {'color': TTkColor.fg('#888888')}
    }

    # Signal declarations
    mySignal: pyTTkSignal

    __slots__ = ('_private_var', 'mySignal')  # Private fields use leading '_'; exposed signals can be public

    _private_var: int
    mySignal: pyTTkSignal

    def __init__(self, **kwargs):
        self.mySignal = pyTTkSignal(int)  # Define signals in __init__
        super().__init__(**kwargs|{'size': (w, h)})  # Merge kwargs, set default size
```

Slot naming/type rules for widgets:
- Use `__slots__` in every widget and helper state class.
- Every non-signal slot name must start with `_`.
- Signals are the only slot entries allowed without `_` because they are part of the widget API.
- Add class-level type annotations for every slot entry directly under `__slots__`.

### Signal-Slot System (Qt-inspired)
Use type-safe signal-slot patterns:
```python
# Define signals with types
signal = pyTTkSignal(int)
# Define slots with decorators
@pyTTkSlot(int)
def my_slot(value: int):
    pass
# Connect them
signal.connect(my_slot)
```

### Event Handling
Widgets handle events by overriding template methods:
- `keyEvent()`, `mousePressEvent()`, `paintEvent()` - Core events
- `focusInEvent()`, `focusOutEvent()` - Focus management
- `dropEvent()`, `dragEnterEvent()` - Drag & drop
- For `keyEvent()`, `mousePressEvent()`, `focusInEvent()`, `focusOutEvent()`, `dropEvent()`, and `dragEnterEvent()`: return `True` only when the event is handled by the widget; otherwise return `False` so it can propagate.

## Development Workflows

### Testing
- **Unit tests**: `pytest tests/pytest/` (run via Makefile: `make test`)
- **Performance tests**: `tests/timeit/` - Contains signal/slot benchmarks and optimization tests
- **Manual tests**: `tests/t.*/` - Interactive UI tests
- **CI**: Tests run on Python 3.10-3.14 with flake8 linting

### Build & Deploy
- **Local build**: `pip install -e libs/pyTermTk` (uses `pip`)
- **Documentation**: `make doc` (Sphinx-based, outputs to `docs/source/_build/html/`)
- **Apps deployment**: Individual apps in `apps/` have their own `pyproject.toml`

### Running Examples
- **Demo**: `python demo/demo.py -f`
- **Designer**: `pip install -e apps/ttkDesigner ; ttkDesigner`
- **Individual tests**: `python tests/t.ui/test.ui.036.datetime.01.py`

## Project-Specific Patterns

### Widget State Management
Many widgets use internal state classes (see `datetime_date_form.py`):
```python
class _TTkWidgetState:
    __slots__ = ('_data', 'signal_name')
    def __init__(self):
        self.signal_name = pyTTkSignal()
```

### File Organization
- One widget per file in `TTkWidgets/`
- Use `__all__ = ['ClassName']` exports
- Import from `TermTk.TTkCore`, `TermTk.TTkWidgets` etc (not relative imports)
- Apps in `apps/` are self-contained with `pyproject.toml`

### Color & Theming
Use `TTkColor` constants and theme system:
```python
TTkColor.fg("#ffffff") + TTkColor.bg("#000044") + TTkColor.BOLD
style = self.currentStyle()  # Get theme-aware colors
```

### Cross-Platform Considerations
- Platform-specific code in `TTkCore/drivers/`
- Terminal compatibility testing in `tests/ansi.images.json`
- HTML5 export capabilities via `tools/webExporter/`

### Documentation & Docstrings
Use **Sphinx-compatible docstrings** with Epytext-style field lists:
```python
# In any sphinx reference
# i.e. ':py:class:' or ':py:meth:'
# the full path is not required but just che class name,
# the link will be resolved in one of the sphynx custom plugins.
def ttkStringData(self, row:int, col:int) -> TTkString:
    '''
    Returns the :py:class:`TTkString` reprsents the data stored in the row/column.

    :param row: the row position of the data
    :type row: int
    :param col: the column position of the data
    :type col: int

    :return: the formatted string
    :rtype: :py:class:`TTkString`
    '''
    data = self.data(row,col)
    return TTkAbstractTableModel._dataToTTkString(data)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ceccopierangiolieugenio/pyTermTk](https://github.com/ceccopierangiolieugenio/pyTermTk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
