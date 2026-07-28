---
trigger: always_on
description: Miew is a high-performance web tool for advanced visualization and manipulation of molecular structures. It provides a full-featured set of tools for 3D visualization and editing of small molecules as well as large molecular complexes, including means to view, analyze, and modify the 3D structure of a molecule.
---

# Miew 3D Molecular Viewer - Repository Instructions

## Project Overview

Miew is a high-performance web tool for advanced visualization and manipulation of molecular structures. It provides a full-featured set of tools for 3D visualization and editing of small molecules as well as large molecular complexes, including means to view, analyze, and modify the 3D structure of a molecule.

The project works as a standalone web application or integrates as a component into web pages. It targets the latest versions of WebGL-enabled desktop (Chrome, Firefox, Safari, Edge) and mobile (iOS, Android) browsers.

## Monorepo Structure

This is a Yarn monorepo organized as follows:

- **Root level**: Contains monorepo configuration, shared tooling, and documentation
- **packages/miew**: Core JavaScript library with the 3D molecular viewer, including docs, examples, and old demo application
- **packages/miew-react**: React.js wrapper component for easy integration
- **packages/miew-app**: New demo application built with React with the goal to replace the old demo application (work in progress)

Note that the core library (`miew`) is framework-agnostic and does not depend on React or any other UI framework. It started in 2015 and can be considered as a legacy-style JavaScript library. It implements its own mutable state management. Other packages (`miew-react`, `miew-app`) are built with modern React best practices.

### Key Directories

- `/packages/miew/src/`: Core library source code
  - `/chem/`: Chemistry-related classes (Complex, Atom, Bond, etc.)
  - `/gfx/`: Graphics rendering (modes, colorers, materials, shaders)
  - `/io/`: File input/output (loaders, parsers for PDB, SDF, etc.)
  - `/ui/`: User interface components and controls
  - `/utils/`: Utility functions and helpers
- `/packages/miew/demo/`: Old demo application assets and scripts
- `/packages/miew/examples/`: Usage examples and integration samples
- `/packages/miew/test/`: Unit and E2E tests
- `/packages/miew/docs/`: Documentation and tutorials
- `/packages/miew-react/src/`: React component wrapper
- `/packages/miew-react/types/`: Type definitions for use by TypeScript projects
- `/packages/miew-app/src/`: New React-based demo application

## Technologies and Frameworks

### Core Technologies
- **JavaScript ES6+**: Primary language for the core library
- **Three.js 0.153.0**: 3D graphics rendering engine
- **WebGL**: Hardware-accelerated 3D graphics
- **Lodash**: Utility library

### Build Tools and Development
- **Webpack 5**: Module bundler for all packages
- **Babel**: JavaScript transpilation with preset-env and preset-react
- **Yarn 3**: Package manager with workspaces
- **Node.js 20-24**: Development environment

### React Ecosystem (miew-app, miew-react)
- **React 16/19**: UI framework
- **React Redux 7**: State management
- **React Bootstrap**: UI components
- **React Icons**: Icon library

### Testing and Quality (miew)
- **Mocha**: Test runner
- **Chai**: Assertion library
- **ESLint**: Linting with Airbnb base config
- **Stylelint**: CSS/SCSS linting
- **NYC**: Code coverage
- **Selenium WebDriver**: E2E testing

### Testing and Quality (miew-app, miew-react)
- **Jest**: Test runner and assertion library
- **React Testing Library**: React component testing

## Coding Standards and Conventions

### Code Quality and Change Management
- **Keep changes focused**: Each commit should address a single, well-defined issue or feature
- **Honor Single Responsibility Principle**: Classes and functions should have one clear purpose
- **Avoid irrelevant changes**: Don't mix formatting, refactoring, or unrelated fixes in feature commits
- **Split large changes**: Break substantial modifications into multiple commits and pull requests for easier review
- **One concept per PR**: Each pull request should implement one feature, fix one bug, or address one improvement

### JavaScript/ES6+
- Follow **Airbnb JavaScript Style Guide** via ESLint configuration
- Use ES6+ features: arrow functions, destructuring, template literals, modules
- Use `import`/`export` for modules (no CommonJS in source)
- Prefer `const` and `let` over `var`
- Use meaningful variable and function names
- Comment complex algorithms and mathematical computations

### React/JSX (for React packages)
- Use functional components with hooks when possible
- Follow React best practices for state management
- Use JSX for component rendering
- Implement proper prop validation
- Use SCSS modules for styling

### File Naming
- Use PascalCase for class files, React components, and SCSS modules: `ComplexVisual.js`, `AboutPanel.module.scss`
- Use camelCase for utility files and modules: `settings.js`, `getTopWindow.js`, `main.scss`

### Code Organization
- One class per file for major components
- Group related functionality in modules/directories
- Use barrel exports (`index.js`) for clean imports
- Keep test files adjacent to source files with `.test.js` suffix

## Development Workflow

### Git Commit Messages

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [epam/miew](https://github.com/epam/miew) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
