---
trigger: always_on
description: *ODF Scan for Zotero* converts plain-text citation markers in word-processor documents into active Zotero citations, and vice-versa. This enables Zotero citation support in editors without a native Zotero plugin (e.g. Scrivener).
---

# CLAUDE.md — ODF Scan for Zotero

## What this plugin does

*ODF Scan for Zotero* converts plain-text citation markers in word-processor documents into active Zotero citations, and vice-versa. This enables Zotero citation support in editors without a native Zotero plugin (e.g. Scrivener).

Two marker formats are supported:
- **Scannable Cite**: `{See | Smith, (2012) |p. 45 | for an example |zu:2433:WQVBH98K}`
- **Pandoc**: `[@citekey]` (uses Zotero's Citation Key field)

Two file formats are supported:
- **ODF** (.odt, .fodt) — downstream requires LibreOffice + Zotero LibreOffice plugin
- **DOCX** (.docx) — downstream requires Word + Zotero Word plugin

Four conversion directions exist:
| Mode value | Direction |
|---|---|
| `tocitations` | Markers → Zotero citations (default) |
| `tomarkers` | Zotero citations → markers |
| `pandoctocitations` | Pandoc → Zotero citations |
| `topandoc` | Zotero citations → Pandoc |

---

## Plugin architecture

This is a **Zotero 7+ bootstrap-style plugin** (no install.rdf, no chrome.manifest).

### Entry point: `bootstrap.js`

Lifecycle hooks called by Zotero:
- `install()` — installs the Scannable Cite translator via `Zotero.Translators.save()`
- `startup()` — registers chrome:// package programmatically via `aomStartup.registerChrome`, sets default prefs, loads `odfScanMenu.js`, initializes the menu object
- `onMainWindowLoad()` / `onMainWindowUnload()` — adds/removes the menu item per window
- `shutdown()` — removes menu items, unregisters chrome package
- `uninstall()` — no-op

Chrome packages registered:
- `chrome://odf-scan/content/` → `chrome/content/`
- `chrome://odf-scan/locale/en-US/` → `chrome/locale/en-US/`

Default preferences (all under `extensions.zotero.`):
- `ODFScan.fileType` — `"odf"`
- `ODFScan.outputMode` — `"tocitations"`
- `ODFScan.odf.lastInputFile{mode}` / `ODFScan.odf.lastOutputFile{mode}` — remembered paths

---

### Source files in `chrome/content/`

#### `odfScanMenu.js` — menu integration
- Defines `Zotero_ODFScan` as a plain object (not a function constructor)
- `addToWindow(win)` inserts a `<menuitem id="menu_odfScan">` after the existing `menu_rtfScan` item in the Tools menu
- `openDialog(win)` opens `chrome://odf-scan/content/odfScan.xhtml` as a `chrome,centerscreen,resizable=yes` dialog

#### `odfScan.xhtml` — dialog UI
- XUL `<window>` (not a wizard — the wizard structure from the original rtfScan has been replaced)
- Loads Zotero core scripts: `chrome://zotero/content/include.js` and `customElements.js`
- Loads `odfScan.js` for the dialog controller
- Four radio buttons for conversion direction
- Input/output file pickers
- Status box (hidden by default, shown on completion/error)

#### `odfScan.js` — dialog controller (`Zotero_ODFScanDialog`)
- `init()` attaches event listeners and restores last-used paths from prefs
- `processDocument()` detects file type from extension, routes to `processDOCX()` or `processODF()`
- **Important**: Creates stub `canAdvance`/`advance`/`rewind` on `document.documentElement` because `odfConverter.js` still uses them
- **Important**: Communicates with converters via globals: `window.inputFile`, `window.outputFile`, `window.DOCXScanComplete`, `window.DOCXScanFailed`
- Scripts are loaded lazily via `Services.scriptloader.loadSubScript()` — always loads `citationUtils.js` first
- ODF completion is detected with `setTimeout(200ms)` polling of flags — fragile but inherited from original design

#### `citationUtils.js` — shared format-agnostic utilities (`CitationUtils`)
Loaded by both `odfScan.js` paths before either converter is loaded. Contains everything not tied to a specific XML format:
- Pandoc syntax parsing: `parsePandocCitationGroup`, `parsePandocLocator`, `labelToPandocLocator`
- Zotero item lookup: `findItemByCitationKey`, `getItemByURI`
- URI construction: `buildItemURI`, `buildItemURIShort`, `httpURIToShort`
- XML escaping: `escapeXml`
- Text-based marker↔pandoc converters used by the ODF pandoc flows:
  - `pandocToMarkersText(content)` — pandoc plain text → scannable cite markers (ODF mode)
  - `markersToPandoc(content)` — scannable cite markers → pandoc plain text

#### `odfConverter.js` — ODF scan logic (`_ODFScanImpl` + `ODFConverter`)
Merges the former `rtfScan.js` (core ODF processing) and `odfConvert.js` (wrapper) into one file.

`_ODFScanImpl` (internal function constructor):
- Contains `_scanODF(outputMode)` — the main ODF processing function using heavy regex-based XML manipulation
- Zotero citation XML template: `<text:reference-mark-start text:name="ZOTERO_ITEM {...} RND{id}"/>...`
- Scannable Cite marker template: `{ prefix | readable | locator | suffix | uri }`
- `runODFScan(outputMode)` — public entry point used by `ODFConverter`
- **Known issue**: Attempts to load stringbundle from `chrome://rtf-odf-scan-for-zotero/locale/zotero.properties` (old plugin ID) — falls back to hardcoded English strings when this fails

`ODFConverter` (public object, `window.ODFConverter`):
- `scanODF(outputMode)` — wraps `_ODFScanImpl.runODFScan(outputMode)`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Juris-M) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
