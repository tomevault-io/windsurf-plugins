---
trigger: always_on
description: This repository contains the source files for the Windows Driver Kit (WDK) Device Driver Interface (DDI) reference documentation. The content is published to [learn.microsoft.com/windows-hardware/drivers/ddi](/windows-hardware/drivers/ddi).
---

# Windows Driver Documentation DDI - Copilot Instructions

## Repository Overview

This repository contains the source files for the Windows Driver Kit (WDK) Device Driver Interface (DDI) reference documentation. The content is published to [learn.microsoft.com/windows-hardware/drivers/ddi](/windows-hardware/drivers/ddi).

### Purpose
- Provides comprehensive API reference documentation for Windows device drivers
- Covers functions, structures, enumerations, IOCTLs, callbacks, interfaces, and classes
- Serves as the authoritative source for driver development APIs across hundreds of header files organized by technology areas

## Repository Structure

```
windows-driver-docs-ddi/
├── wdk-ddi-src/content/         # Main content directory
│   ├── {header}/                # Header file directories (e.g., ntddk, wificx, etc.)
│   │   ├── na-{header}-*.md     # Header file entry page, should be renamed to index.md
│   │   ├── nf-{header}-*.md     # Function documentation
│   │   ├── ns-{header}-*.md     # Structure documentation  
│   │   ├── ne-{header}-*.md     # Enumeration documentation
│   │   ├── ni-{header}-*.md     # IOCTL documentation
│   │   ├── nc-{header}-*.md     # Callback documentation
│   │   ├── nn-{header}-*.md     # Interface documentation
│   │   ├── nl-{header}-*.md     # Class documentation
│   │   └── index.yml            # Header index
│   ├── _{technology}/           # Technology area directories (e.g., _audio, _kernel, etc.)
│   │   ├── config.json          # Lists headers included in this technology
│   │   └── index.md             # Technology area overview
│   └── docfx.json               # Build configuration
├── CONTRIBUTING.md              # Contribution guidelines
└── README.md                    # Repository information
```

### Folder Organization
- **Header directories** (no underscore): Folders like `ntddk`, `wificx`, `acxcircuit` correspond to header file names and contain the API documentation files
- **Technology directories** (with underscore): Folders like `_audio`, `_kernel`, `_display` represent technology areas and contain a `config.json` that lists all headers belonging to that technology
- Headers can appear in multiple technology areas but must appear in at least one
- Each technology area groups related headers for organizational purposes

## File Naming Conventions

All documentation files follow a strict naming pattern using prefixes to indicate the type of API element:

### Filename Prefix Key

| Prefix | Type | Description | Example |
|--------|------|-------------|---------|
| `na` | Header | Header file entry page, should be renamed to `index.md` | `na-wificx.md` |
| `ne` | Enum | Enumerations and enumerated types | `ne-wificx-wifi_adapter_type.md` |
| `nc` | Callback | Callback functions and function pointers | `nc-wificx-evt_wifi_device_create_adapter.md` |
| `ni` | IOCTL | Input/Output Control codes | `ni-charging-ioctl_cad_disable_charging.md` |
| `ns` | Structure | Data structures and unions | `ns-wificx-wifi_device_config.md` |
| `nl` | Class | Classes (primarily for C++ APIs) | `nl-engextcpp-extremotedata.md` |
| `nn` | Interface | COM interfaces and abstract base classes | `nn-{header}-{interface_name}.md` |
| `nf` | Function | Functions and methods | `nf-wificx-wifideviceinitialize.md` |

### Naming Pattern
Files should follow the pattern: `{prefix}-{header}-{api_name}.md`

Where:
- `{prefix}` is one of the prefixes above
- `{header}` matches the directory name
- `{api_name}` is the lowercase, hyphenated version of the API name

## Content Structure

Each documentation file contains:

### YAML Frontmatter (Required)
Every file must start with YAML frontmatter containing metadata:

```yaml
---
UID: {Unique identifier}
title: {API Name} ({header file})
description: {Brief description of the API}
tech.root: {Technology area}
ms.date: {Date in MM/DD/YYYY format}
targetos: Windows
keywords: ["{API Name}"]
ms.keywords: {Comma-separated keywords}
req.header: {Header file name}
req.include-header: {Include header if different}
req.target-type: {Target type}
req.target-min-winverclnt: {Minimum Windows client version}
req.target-min-winversvr: {Minimum Windows server version}
req.lib: {Required library}
req.dll: {Required DLL}
req.irql: {IRQL level}
req.ddi-compliance: {DDI compliance info}
topic_type:
 - apiref
api_type:
 - {HeaderDef|DllExport|LibDef}
api_location:
 - {header file or library}
api_name:
 - {API name}
f1_keywords:
 - {API name}
 - {header}/{API name}
dev_langs:
 - c++
---
```

#### Required AI Usage Metadata

**IMPORTANT**: When Copilot creates or modifies content, ensure that `ai-usage: ai-assisted` is included as a metadata attribute in the YAML frontmatter. If this attribute is not present, add it.

### Common Metadata Patterns

#### API Types by Prefix:
- **Enumerations (ne)**: `req.construct-type: enumeration`
- **Callbacks (nc)**: `req.construct-type: function` 
- **IOCTLs (ni)**: `api_type: HeaderDef`
- **Structures (ns)**: `req.construct-type: structure`
- **Classes (nl)**: `api_type: HeaderDef` or `LibDef`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MicrosoftDocs/windows-driver-docs-ddi](https://github.com/MicrosoftDocs/windows-driver-docs-ddi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
