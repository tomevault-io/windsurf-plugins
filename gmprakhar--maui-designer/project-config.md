---
trigger: always_on
description: MAUI Designer is an Angular 18.2.0 web application for designing XAML-based user interfaces with drag-and-drop functionality. It provides a visual XAML editor, element toolbox, properties panel, and real-time preview capabilities for creating MAUI UI layouts in a web browser.
---

# MAUI Designer - Angular
MAUI Designer is an Angular 18.2.0 web application for designing XAML-based user interfaces with drag-and-drop functionality. It provides a visual XAML editor, element toolbox, properties panel, and real-time preview capabilities for creating MAUI UI layouts in a web browser.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Platform Requirements
- **Cross-Platform**: This Angular application runs on any platform with Node.js support (Windows, macOS, Linux)
- **Modern Web Browser**: Requires a modern browser with JavaScript support
- **Internet Connection**: Initial setup requires network access for package downloads

### Prerequisites and Setup
- Install **Node.js** (version 16.x or higher): Download from https://nodejs.org/
- Install **npm** (version 8.x or higher): Usually comes with Node.js
- Install **Angular CLI** globally: `npm install -g @angular/cli` (version 18.x)
- Verify installation: `ng version` should show Angular CLI and core versions

### Building the Application
- **Bootstrap and build**:
  - `cd /path/to/MAUI-Designer`
  - `npm install` -- takes 30-90 seconds for package installation
  - `npm run build` or `ng build` -- takes 1-3 minutes. Set timeout to 5+ minutes.
- **Build for specific configuration**:
  - Development: `ng build` or `ng build --configuration development`
  - Production: `ng build --configuration production` or `npm run build`

### Running the Application
- **Development server**: `npm start` or `ng serve` -- starts dev server on http://localhost:4200
- **Production build**: `ng build` then serve the `dist/` folder with any web server
- The application will launch in a web browser with the XAML designer interface

## Validation
- **Manual Testing Required**: After making changes, always test these core scenarios:
  1. **XAML Editor**: Write XAML in the text editor and click "Apply" to see the preview
  2. **Drag and Drop**: Select elements from the toolbox and drag them to the design canvas
  3. **Element Scaling**: Select an element and use handles to resize it  
  4. **Properties Panel**: Select an element and modify properties like color, margin, padding, text
  5. **XAML Generation**: Use the "Download" feature to export the designed layout
  6. **Load Design**: Import existing XAML to recreate the visual design
- **Layout Support**: Supports multiple layout types including StackLayout, Grid, and AbsoluteLayout
- **Limited Automated Tests**: This repository has minimal test infrastructure - validation is primarily manual
- **Post-Build Validation**: Always run `ng serve` after building to ensure UI components load correctly in browser
- **Designer Workflow Test**: Create a simple layout with Label, Button, and Entry controls to verify basic functionality

### Running Tests
- **Unit Tests**: `npm test` or `ng test` -- runs Jasmine/Karma tests in watch mode
- **Build Tests**: `ng build` -- verifies TypeScript compilation and bundling
- **E2E Tests**: Currently no E2E test infrastructure configured
- **Linting**: Use `ng lint` if ESLint is configured, or standard TypeScript compiler checks

### Browser Compatibility
- **Modern Browsers**: Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- **JavaScript Required**: Application requires JavaScript enabled
- **Local Storage**: Uses browser local storage for user preferences
- **Responsive Design**: Designed primarily for desktop/tablet screen sizes

### Build Time Expectations
- **First npm install**: 30-90 seconds (includes package downloads). Set timeout to 3+ minutes.
- **Incremental builds**: 10-30 seconds for development builds
- **Production builds**: 1-3 minutes. Set timeout to 5+ minutes.
- **Development server startup**: 5-15 seconds

## Repository Structure

### Key Projects and Files
- **package.json**: NPM package configuration with Angular dependencies
- **angular.json**: Angular CLI workspace configuration
- **src/app/app.ts**: Main application component with resizable panels
- **src/app/components/**: UI component modules
- **src/app/services/**: Business logic and data services
- **src/app/models/**: TypeScript interfaces and data models

### Important Directories
- **src/app/components/**: Angular components for the designer interface
  - **designer-canvas/**: Main design surface component
  - **hierarchy-panel/**: Element tree view component
  - **properties-panel/**: Property editor component
  - **toolbox/**: Element selection toolbox component
  - **xaml-editor/**: XAML code editor component
- **src/app/services/**: Business logic services
  - **drag-drop.ts**: Drag-and-drop functionality
  - **element.ts**: Element management service
  - **layout-designer.ts**: Layout calculation service
  - **xaml-generator.ts**: XAML code generation
  - **xaml-parser.ts**: XAML parsing service
- **src/app/models/**: Data models and interfaces
  - **maui-element.ts**: MAUI element definitions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GMPrakhar/MAUI-Designer](https://github.com/GMPrakhar/MAUI-Designer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
