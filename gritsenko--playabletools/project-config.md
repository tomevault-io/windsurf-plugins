---
trigger: always_on
description: <!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->
---

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# PlayableTools Project Architecture Guide

## Tech Stack
- **Build Tool**: Vite 7.x with TypeScript 5.8
- **Frontend Framework**: Lit 3.x web components
- **CSS Framework**: Pico CSS 2.x for styling with custom theme overrides
- **PWA**: Vite PWA plugin for service worker and offline support
- **Dependencies**: JSZip for archive handling, Pako for compression, Marked for markdown parsing, D3.js for visualizations

## Project Structure

### Core Framework (`src/fw/`)
This project uses a custom lightweight framework built on Lit:
- **ComponentBase**: Light DOM Lit components (no shadow DOM by default)
- **LayoutComponentBase**: Base class for layout components  
- **Dependency Injection**: Custom DI container with `@injectable` and `@inject` decorators
- **Router**: Clean URL routing with `@route` decorator, History API navigation helpers, and metadata support
- **Service Lifetimes**: Singleton (default), Scoped, Transient
- **Update Notification**: Version checking and update prompts

### Import Pattern
Always import from the `fw` alias:
```typescript
import { ComponentBase, customElement, html, route, inject, injectable, ServiceLifetime } from "fw";
```

### Component Conventions

#### Pages (`src/pages/`)
- Location: `src/pages/` folder
- Naming: kebab-case files (e.g., `home-page.ts`)
- Structure:
```typescript
import { ComponentBase, customElement, html, route } from "fw";

@customElement("page-name")
@route("/path", {
  title: "Page Title for SEO",
  description: "Page description for SEO"
})
export class PageName extends ComponentBase {
  render() {
    return html`<div>Content</div>`;
  }
}
```

#### Reusable Components
- Co-located CSS files: `component-name.ts.css`
- Light DOM by default (no shadow DOM)
- Use `@property` for public component APIs (data passed from parent) and `@state` for internal component state (data managed by the component itself)

#### Layout Components (`src/Layout/`)
- Use `LayoutComponentBase` for layouts
- Main layout: `main-layout.ts` with sidebar, navigation
- Components: `nav-menu.ts`, `site-logo.ts`

### Services (`src/services/`)
Use dependency injection for all services:

```typescript
import { injectable, ServiceLifetime, inject } from "fw";

@injectable(ServiceLifetime.Singleton) // or omit for default Singleton
export class MyService {
  constructor() {}
  
  async doSomething(): Promise<void> {
    // Service logic
  }
}

// In components/other services:
export class MyComponent extends ComponentBase {
  @inject()
  private myService!: MyService;
}
```

**Available Services:**
- `PlayablePublishService` - Publishing to 10+ ad networks with platform-specific transformations
- `ImbaPackerService` - HTML compression with Pako
- `Base64ConverterService` - File to base64 conversion
- `PreviewService` - Playable preview functionality with ZIP support
- `PortfolioService` - GitHub portfolio integration
- `VersionService` - App version checking and PWA updates
- `MetadataService` - SEO metadata management
- `Video2SpriteService` - Video to PNG sprite sequence conversion
- **Validators** (`PreviewServiceValidators/`) - Platform-specific validation tools
  - `FacebookValidator` - Facebook ad requirements validation
  - `GeneralValidator` - General platform validation
  - `MraidValidator` - MRAID compliance validation

### Routing & Navigation
- **Clean URL routing** based on `window.location.pathname`
- **History API helpers** are exported from `fw`: `navigate()`, `getCurrentPath()`, `getCurrentSearch()`
- Routes defined with `@route("/path", { title, description })`
- Automatic page loading via `import.meta.glob("./pages/**/*.ts", { eager: true })`
- Navigation handled by `nav-menu.ts`
- Legacy `/#/...` links are migrated at bootstrap time in `index.html`
- Public SEO landing pages are declared in `src/seo/route-manifest.json`

### File Organization
```
src/
├── fw/                    # Custom framework
├── Layout/               # Layout components  
├── pages/               # Page components
│   ├── publish/         # Publishing tools (playable-publisher, publish-page)
│   ├── preview/         # Preview tools (playable-previewer, preview-page)
│   ├── portfolio/       # Portfolio management (portfolio-page)
│   ├── folder-size/     # Folder visualization (multiple view components)
│   ├── spritesheet-maker/ # Sprite sheet creation
│   ├── video2sprite/    # Video to sprite conversion
│   ├── base64-converter.ts # File encoding tool
│   ├── compress-assets-page.ts # Asset compression
│   ├── cta-sdk-page.ts # CTA SDK documentation
│   ├── home-page.ts # Main landing page
│   ├── imba-packer-page.ts # HTML compression tool
│   └── validate-page.ts # Platform validation
├── services/            # Business logic services
│   └── PreviewServiceValidators/ # Validation services
├── utils/               # Utility functions (framework-agnostic helpers)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gritsenko/PlayableTools](https://github.com/gritsenko/PlayableTools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
