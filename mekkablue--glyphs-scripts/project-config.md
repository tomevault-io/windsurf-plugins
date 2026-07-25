---
trigger: always_on
description: Python scripts for the [Glyphs font editor](http://glyphsapp.com/). Scripts run inside Glyphs.app via its Python runtime (Python 3, PyObjC).
---

# mekkablue Glyphs Scripts

Python scripts for the [Glyphs font editor](http://glyphsapp.com/). Scripts run inside Glyphs.app via its Python runtime (Python 3, PyObjC).

## Project Structure

- Each subfolder contains standalone `.py` scripts grouped by topic (~370 scripts total across 25+ categories).
- `__init__.py` — shared `mekkaObject` base class and utility functions (clipboard, wildcard matching, etc.).
- `geometry.py` — math/geometry helpers (transforms, italicization, intersections, etc.).
- `pyproject.toml` — black/flake8 config (line length: 120).
- `.style.yapf` — yapf formatting config (tabs, column limit 180).
- `.flake8` — flake8 ignore rules (W191, E501, E722, W503, E741, F841, E265, E225).

### Script Categories

| Folder | Topic |
|---|---|
| `Anchors/` | Anchor management & positioning (29 scripts) |
| `App/` | Application-level utilities, navigation (10 scripts) |
| `Build Glyphs/` | Font build utilities (3 scripts) |
| `Color Fonts/` | COLR/CBDT color font handling (4 scripts) |
| `Compare Frontmost Fonts/` | Multi-font comparison (4 scripts) |
| `Components/` | Component generation, alignment, flattening (13 scripts) |
| `Features/` | OpenType feature code generation (15 scripts) |
| `Font Info/` | Font metadata handling (8 scripts) |
| `Glyph Names, Notes and Unicode/` | Naming & Unicode assignment (12 scripts) |
| `Guides/` | Guide management (10 scripts) |
| `Hinting/` | TrueType hinting utilities (6 scripts) |
| `Images/` | Image/bitmap handling (7 scripts) |
| `Interpolation/` | Variable font, brace layers, axis manipulation (19 scripts) |
| `Kerning/` | Kerning analysis & manipulation (21 scripts) |
| `Paths/` | Path/contour operations (21 scripts) |
| `Pixelfonts/` | Bitmap font utilities (4 scripts) |
| `Post Production/` | Build finishing tasks (11 scripts) |
| `Smallcaps/` | Smallcaps generation (6 scripts) |
| `Spacing/` | Metric & spacing tools (10 scripts) |
| `Test/` | Testing & QA helpers (7 scripts) |

Some subfolders contain helper modules (not scripts), e.g., `Interpolation/axisMethods.py` for axis value lookups.

## Script File Header

Every script must begin in this exact order:

```python
# MenuTitle: My Script Name
# -*- coding: utf-8 -*-
from __future__ import division, print_function, unicode_literals
__doc__ = """
Short description of what the script does.
"""
```

`# MenuTitle:` **must be the very first line** of every script.

## Code Conventions

- **Tabs**, not spaces, for indentation (PyObjC convention — underscores have special meaning in PyObjC).
- **camelCase** for all variables and function names (not `under_score`).
- Descriptive names: `points` not `p`; `layers` not `layerList` or `listOfLayers`.
- Max line length: 120 (flake8/black), 180 (yapf).

## Import Order

```python
# Standard library
from itertools import product
from copy import copy

# Vanilla UI toolkit
import vanilla

# mekkablue shared module
from mekkablue import mekkaObject

# GlyphsApp core
from GlyphsApp import Glyphs, GSFont, GSLayer

# PyObjC frameworks
from AppKit import NSFont, NSAffineTransform
from Foundation import NSPoint
```

## Script Structures

### Simple scripts (no GUI)

```python
# MenuTitle: My Simple Script
# -*- coding: utf-8 -*-
from __future__ import division, print_function, unicode_literals
__doc__ = """Description."""

from GlyphsApp import Glyphs

font = Glyphs.font
if font:
	# do work directly at module level
	pass
```

### GUI scripts — subclass `mekkaObject`

```python
# MenuTitle: My GUI Script
# -*- coding: utf-8 -*-
from __future__ import division, print_function, unicode_literals
__doc__ = """Description."""

import vanilla
from mekkablue import mekkaObject


class MyScript(mekkaObject):
	prefDict = {
		"someOption": True,
		"someValue": 0,
	}

	def __init__(self):
		windowWidth = 330
		windowHeight = 240
		windowWidthResize = 0   # extra width the user can resize by
		windowHeightResize = 0  # extra height the user can resize by
		self.w = vanilla.FloatingWindow(
			(windowWidth, windowHeight),
			"My Script",
			minSize=(windowWidth, windowHeight),
			maxSize=(windowWidth + windowWidthResize, windowHeight + windowHeightResize),
			autosaveName=self.domain("mainwindow"),  # persists window position/size
		)

		# UI elements:
		linePos, inset, lineHeight = 12, 15, 22

		self.w.myCheckbox = vanilla.CheckBox((inset, linePos, -inset, 20), "Do the thing", value=True, callback=self.SavePreferences, sizeStyle="small")
		self.w.myCheckbox.setToolTip("Tooltip explaining what this does.")
		linePos += lineHeight

		self.w.runButton = vanilla.Button((-80 - inset, -20 - inset, -inset, -inset), "Run", callback=self.run, sizeStyle="small")
		self.w.setDefaultButton(self.w.runButton)

		self.LoadPreferences()
		self.w.open()
		self.w.makeKey()

	def updateUI(self, sender=None):
		# enable/disable dependent elements based on current pref values
		self.w.myCheckbox.enable(onOff=True)

	def SavePreferences(self, sender=None):
		super().SavePreferences(sender)

	def run(self, sender):
		# main action
		font = Glyphs.font
		if not font:
			return
		print("Report for My Script\n")
		# ... do work ...
		Glyphs.showNotification("My Script", "Done! Details in Macro Window.")


MyScript()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mekkablue/Glyphs-Scripts](https://github.com/mekkablue/Glyphs-Scripts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
