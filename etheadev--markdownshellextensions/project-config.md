---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Markdown Shell Extensions is a Delphi project providing Windows shell integration for markdown files:
- **MDShellExtensions.dll** (32/64-bit) - Windows shell extension library providing Preview Handler and Thumbnail Provider for markdown files in Windows Explorer
- **MDTextEditor.exe** - Standalone markdown text editor with live HTML preview

## Build Commands

### Prerequisites
- Delphi 12 or 13 (RAD Studio)
- Inno Setup 6 (for installer)

### Building with Delphi IDE
Open `Source\MDShellExtensionsGroup.groupproj` which contains:
- `MDShellExtensions32.dproj` - 32-bit shell extension DLL
- `MDShellExtensions.dproj` - 64-bit shell extension DLL
- `MDTextEditor.dproj` - Markdown editor application

### Building via Command Line
```batch
# Set up Delphi environment (adjust path to your installation)
call "C:\BDS\Studio\37.0\bin\rsvars.bat"

# Build individual projects
msbuild.exe "Source\MDShellExtensions.dproj" /target:Clean;Build /p:Platform=Win64 /p:config=release
msbuild.exe "Source\MDShellExtensions32.dproj" /target:Clean;Build /p:Platform=Win32 /p:config=release
msbuild.exe "Source\MDTextEditor.dproj" /target:Clean;Build /p:Platform=Win64 /p:config=release
msbuild.exe "Source\MDTextEditor.dproj" /target:Clean;Build /p:Platform=Win32 /p:config=release
```

### DLL Registration (requires Administrator)
```batch
# Register 32-bit DLL
%systemroot%\SysWoW64\regsvr32 "Bin32\MDShellExtensions32.dll"

# Register 64-bit DLL
%systemroot%\System32\regsvr32 "Bin64\MDShellExtensions.dll"

# Unregister uses same commands with /u flag
```

**Important**: Close all Windows Explorer instances before registering/unregistering as the DLL stays loaded in memory.

### Building Installer
```batch
"C:\Program Files (x86)\Inno Setup 6\iscc.exe" "Setup\MDShellExtensions.iss"
```

## Architecture

### Core Components

**Shell Extension DLL** (`MDShellExtensions.dpr`):
- `MDShellEx.PreviewHandler` - Implements Windows Preview Handler (IPreviewHandler) for Explorer preview pane
- `MDShellEx.ThumbnailHandler` - Implements thumbnail provider (IThumbnailProvider) for file icons
- `MDShellEx.ContextMenuHandler` - Right-click context menu integration
- `MDShellEx.PreviewForm` - VCL form rendered in Explorer preview pane
- Uses GUIDs for 32-bit and 64-bit variants (defined in `MDShellEx.PreviewHandler.pas`)

**Editor Application** (`MDTextEditor.dpr`):
- `MDTextEditor.EditorMainForm` - Main editor form with tabbed interface
- `MDShellEx.Settings` - User preferences and theme management
- `MDShellEx.Resources` - Shared resources (images, icons)

### Key Dependencies (in Ext folder)
- **MarkdownProcessor** - Markdown to HTML conversion
- **HTMLViewer** - HTML rendering component
- **SynEdit** - Syntax highlighting editor
- **SVGIconImageList** - SVG icon support
- **StyledComponents** - VCL styled components
- **Skia4Delphi** - Image format support (webp, wbmp)
- **Image32** - SVG rendering

### Supported File Extensions
Defined in `MDShellEx.Misc.pas`:
`.md`, `.mkd`, `.mdwn`, `.mdown`, `.mdtxt`, `.mdtext`, `.markdown`

### Output Directories
- `Bin32\` - 32-bit binaries
- `Bin64\` - 64-bit binaries
- `Setup\Output\` - Installer output

## Theme Support

The application supports Windows light/dark themes. Theme selection is managed via `TThemeSelection` enum in `MDShellEx.Settings.pas`:
- `tsAsWindows` - Follow Windows theme
- `tsDarkTheme` - Force dark theme
- `tsLightTheme` - Force light theme

## MDTextEditor - Detailed Architecture

### Main Form Structure (`MDTextEditor.EditorMainForm.pas`)

**TfrmMain** - Main application form implementing:
- `IDragDrop` interface for file drag-and-drop support
- Tabbed document interface via `TPageControlEx`
- Split view with collapsible menu (`TSplitView`)
- Markdown toolbar with formatting buttons (`TStyledToolbar`)

**TEditingFile** - Class representing each open document:
- `SynEditor: TSynEdit` - Syntax highlighted markdown editor
- `HTMLViewer: THTMLViewer` - Live HTML preview panel
- `FMarkDownFile: TMarkDownFile` - Parsed markdown content (from MarkdownProcessor)
- `TabSheet: TTabSheet` - Tab in the PageControl
- `Splitter: TSplitter` - Resizable splitter between editor and preview
- Key methods: `ReadFromFile`, `SaveToFile`, `ShowMarkDownAsHTML`, `UpdateRootPath`

### Actions and Features

**File Operations** (`ActionList`):
- `acNewFile`, `acOpenFile`, `acSave`, `acSaveAs`, `acSaveAll`
- `acClose`, `acCloseAll`, `acQuit`
- `OpenRecentAction` - Recent files menu
- `acSaveHTMLFile`, `acSavePDFFile` - Export functionality

**Edit Operations**:
- `acEditCut`, `acEditCopy`, `acEditPaste`, `acEditSelectAll`, `acEditUndo`
- `acSearch`, `acReplace`, `acSearchAgain`
- `acZoomIn`, `acZoomOut`, `acRefresh`

**Markdown Toolbar Actions** (`ToolbarActionList`):
- Headers: `acHeader1`, `acHeader2`, `acHeader3`
- Links/Media: `acLink`, `acImage`, `acTable`
- Text Formatting: `acBold`, `acItalic`, `acStrike`, `acUnderline`, `acCode`, `acMarker`
- Subscript/Superscript: `acSubscript`, `acSuperscript`
- Lists: `acUnorderedList`, `acOrderedList`
- Blocks: `acBlockquote`, `acHorizontalRule`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EtheaDev/MarkdownShellExtensions](https://github.com/EtheaDev/MarkdownShellExtensions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
