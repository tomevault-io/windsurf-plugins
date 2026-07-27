---
trigger: always_on
description: This is an **Angular library** (not an app) that exports HTML/table elements to multiple file formats (PDF, PNG, Excel, Word, CSV, JSON, XML). The project uses a **monorepo structure** with the library in `projects/ngx-export-as/` and a demo app in `src/` for testing.
---

# ngx-export-as - AI Agent Instructions

## Project Overview

This is an **Angular library** (not an app) that exports HTML/table elements to multiple file formats (PDF, PNG, Excel, Word, CSV, JSON, XML). The project uses a **monorepo structure** with the library in `projects/ngx-export-as/` and a demo app in `src/` for testing.

**As of v1.21.0:** The library is fully standalone - no NgModule required. Users provide `ExportAsService` directly in components or app.config.ts.

## Architecture

### Library Structure (projects/ngx-export-as/)
- **Public API**: `src/public_api.ts` - Single entry point exporting: `ExportAsService`, `ExportAsConfig`
- **Service**: `export-as.service.ts` - Core service with private methods per format (`getPDF()`, `getPNG()`, `getXLS()`, etc.)
- **Configuration**: `export-as-config.model.ts` - Type-safe config interface with `SupportedExtensions` union type

> **Note:** As of v1.21.0, the library is fully standalone. No NgModule required - provide `ExportAsService` directly in components or app.config.ts

### Key Design Patterns

1. **Dynamic Method Resolution**: The `get()` method uses string manipulation to call format-specific methods:
   ```typescript
   const func = 'get' + config.type.toUpperCase(); // e.g., 'getPDF'
   if (this[func]) return this[func](config);
   ```

2. **Observable-Based API**: All methods return `Observable<string | null>` (except JSON which returns `Observable<any[] | null>`)
   - `save()`: Downloads file, returns `Observable` that completes when download starts
   - `get()`: Returns base64 content (except JSON returns actual objects)

3. **UTF-8 Base64 Encoding**: Custom `btoa()` wrapper handles special characters:
   ```typescript
   private btoa(content: string) {
     return btoa(unescape(encodeURIComponent(content)));
   }
   ```

4. **SSR Compatibility**: Platform checks prevent browser-only code from running server-side:
   ```typescript
   if (isPlatformBrowser(this.platformId)) {
     window['html2canvas'] = html2canvas;
   }
   ```

## Critical Build & Publishing Workflow

### Build Commands (package.json scripts)
```bash
npm run build_lib          # Build library only
npm run package            # Build + copy LICENSE/README to dist
npm run npm_pack           # Create .tgz package
npm run npm_publish        # Full publish workflow
```

### Important: TypeScript Target Requirements
- **Library tsconfig**: `target: "es2015"` (required for DOCX support via html-to-docx)
- **Demo app**: Can use any target
- See `projects/ngx-export-as/tsconfig.lib.json`

### Publishing Checklist
1. Build library: `ng build ngx-export-as --configuration=production`
2. Files automatically copied to `dist/ngx-export-as/`:
   - LICENSE (via `copy-license` script)
   - README.md (via `copy-readme` script)
3. Output location: `dist/ngx-export-as/` (configured in `ng-package.json`)

## Format-Specific Implementation Details

### Table-Required Formats
These **require** HTML `<table>` element (querySelector for `tr`, `td`, `th`):
- CSV, TXT, XLS/XLSX, JSON, XML

### PDF/PNG Accept Any Element
- Work with any `elementIdOrContent` (div, table, canvas, etc.)
- PDF also accepts: HTMLElement, Canvas, Image directly

### DOCX/DOC Status
- **Currently commented out** in service (see lines with `// private getDOCX()`)
- Requires `target: es2015` when enabled
- Issue reference: https://github.com/privateOmega/html-to-docx/issues/145

## Browser Polyfills (IE Support)

The demo app includes comprehensive IE11 polyfills:
1. **Core-JS polyfills**: All enabled in `src/polyfills.ts`
2. **TypedArray polyfill**: Custom implementation in `src/polyfills/typedarray.js`
3. **Buffer/Process shims**: Required for xlsx library compatibility

## Testing the Library Locally

### Demo App Development
```bash
ng build ngx-export-as    # Build library first
ng serve                   # Run demo at localhost:4200
```

**Important**: Always rebuild the library after changes to see them in demo app.

### Example Usage (see src/app/app.component.ts)
```typescript
import { Component, inject } from '@angular/core';
import { ExportAsService, ExportAsConfig } from 'ngx-export-as';

@Component({
  selector: 'app-root',
  standalone: true,
  providers: [ExportAsService]  // Provide service in component
})
export class AppComponent {
  private readonly exportAsService = inject(ExportAsService);
  
  config: ExportAsConfig = {
    type: 'pdf',
    elementIdOrContent: 'mytable',
    options: {
      jsPDF: { orientation: 'landscape' },
      pdfCallbackFn: this.pdfCallbackFn  // Add headers/footers
    }
  };

  export() {
    this.exportAsService.save(this.config, 'myFile').subscribe(() => {
      // Download started
    });
  }
}
```

## JSDoc Standards

All public methods have comprehensive JSDoc with:
- `@description` with detailed explanation
- `@param` with types and descriptions
- `@returns` with Observable type details
- `@example` code blocks showing real usage
- `@memberof ExportAsService`

Private methods use same standard but include `@private` tag.

## Dependencies & External Libraries


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wnabil/ngx-export-as](https://github.com/wnabil/ngx-export-as) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
