---
trigger: always_on
description: This is a VS Code extension for editing, previewing, and validating URDF (Unified Robot Description Format) and Xacro files. The extension provides 3D visualization using BabylonJS and supports WebXR preview for robotics development.
---

# URDF Editor VS Code Extension - AI Coding Instructions

## Project Overview
This is a VS Code extension for editing, previewing, and validating URDF (Unified Robot Description Format) and Xacro files. The extension provides 3D visualization using BabylonJS and supports WebXR preview for robotics development.

## Architecture
- **Extension Entry**: `src/extension.ts` - Main activation point, registers commands and providers
- **Preview System**: `src/previewManager.ts` + `src/preview.ts` - Manages URDF/Xacro/OpenSCAD 3D preview webviews
- **OpenSCAD Processing**: `src/openscad.ts` - OpenSCAD conversion utilities and library management
- **3D Viewer**: `src/3DViewerProvider.ts` + `src/3DViewerDocument.ts` - Custom editor for STL/DAE mesh files
- **Webview Frontend**: `src/webview/webview.ts` - BabylonJS-based 3D rendering in webview
- **URDF Processing**: `src/utils.ts` - Core Xacro parsing and package resolution logic

## Key Patterns

### Xacro Processing Pipeline
The extension uses a sophisticated pipeline for processing Xacro files:
1. `XacroParser` from `xacro-parser` npm package handles macro expansion
2. Custom `getFileContents` function resolves `$(find package_name)` idioms during parsing
3. Post-processing converts `package://` URIs to webview URIs for mesh loading
4. Missing packages are tracked and reported to user

Example from `utils.ts`:
```typescript
// Convert $(find package) to package:// format before parsing
export function convertFindToPackageUri(text: string): string {
  const findPattern = /\$\(find\s+([a-zA-Z0-9_-]+)\)/g;
  return text.replace(findPattern, (match, packageName) => {
    return `package://${packageName}`;
  });
}
```

### Package Resolution
Packages are discovered by scanning workspace for `package.xml` files. The `getPackages()` function in `utils.ts` builds a map of package names to filesystem paths, enabling resolution of ROS package references.

### OpenSCAD Processing
OpenSCAD (.scad) files are processed using dedicated utilities in `src/openscad.ts`:
1. File is detected as OpenSCAD by extension
2. OpenSCAD libraries are loaded from OS-specific and user-configured paths
3. `openscad-wasm-prebuilt` module converts .scad code to STL format with library support
4. Generated STL is written to the same directory as the .scad file
5. Existing STL viewer renders the converted file through webview
6. File watching triggers re-conversion on .scad file changes

#### Library Support
OpenSCAD library loading supports:
- **SCAD file directory**: Automatically included with highest priority (the directory containing the SCAD file being processed). **NOTE**: Only the SCAD file's directory is included from the workspace - the workspace root is no longer automatically added to avoid copying large directories like `.git`, `node_modules`, or virtual environments.
- **OS-specific default paths**:
  - Windows: `%USERPROFILE%\Documents\OpenSCAD\libraries`
  - Linux: `$HOME/.local/share/OpenSCAD/libraries`
  - macOS: `$HOME/Documents/OpenSCAD/libraries`
- **User-configured paths**: Via `urdf-editor.OpenSCADLibraryPaths` setting
- **Workspace variables**: `${workspaceFolder}` resolves to workspace root
- **Automatic discovery**: Only existing directories are included
- **Recursive loading**: Subdirectories and files (.scad, .stl, .dxf) loaded into virtual filesystem
- **Directory exclusion**: Automatically excludes common directories like `.git`, `node_modules`, `venv`, `__pycache__`, `dist`, `build`, etc. to improve performance

#### Library Documentation
The extension can generate comprehensive documentation of available OpenSCAD libraries:
- **Command**: `urdf-editor.generateOpenSCADDocs` - Generate markdown documentation
- **MCP Tool**: `get_openscad_libraries` - Expose library info to AI assistants
- **Extraction**: Header comments, module signatures, function parameters, and inline documentation
- **Format**: Structured markdown suitable for AI consumption

Example from `src/openscad.ts`:
```typescript
// Load libraries and convert OpenSCAD to STL
const libraryPaths = await getAllOpenSCADLibraryPaths(workspaceRoot);
await loadLibraryFiles(instance, libraryPaths, this._trace);
instance.FS.writeFile('/input.scad', scadText);
instance.callMain(['-L', '/libraries', '-o', '/output.stl', '/input.scad']);

// Generate documentation for MCP
const documentation = await generateOpenSCADLibrariesDocumentation(workspaceRoot);
const markdown = convertLibrariesDocumentationToMarkdown(documentation);
````
```

### OpenSCAD Processing
OpenSCAD (.scad) files are processed in `3DViewerDocument.ts`:
1. File content is read from the filesystem
2. `openscad-wasm` module converts .scad code to STL format
3. Generated STL is written to a temporary file
4. Existing STL viewer renders the converted file
5. File watching triggers re-conversion on .scad file changes

### Webview Communication
Bidirectional communication between extension and webview uses message passing:
- Extension → Webview: URDF content, color settings, file paths
- Webview → Extension: Ready state, errors, trace messages

Commands sent to webview:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ranch-Hand-Robotics/rde-urdf](https://github.com/Ranch-Hand-Robotics/rde-urdf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
