---
trigger: always_on
description: - **ALWAYS** run `npm run build` after any code modification to verify that the project still compiles correctly.
---

# Workspace Instructions

## Build and Verification

- **ALWAYS** run `npm run build` after any code modification to verify that the project still compiles correctly.
- If the build fails, you must diagnose and fix the errors before concluding the task.
- Verification is only complete when `npm run build` exits with code 0.

## Basic Instructions

SilkStack Image Browser is a desktop application (Electron + React + TypeScript) for browsing, searching, and organizing AI-generated images locally.

The development is done in Windows, so use Windows-specific tools and commands when invoking commands for development. Always ensure that the code is compatible with Windows, MacOS and Linux. Always check if code builds without errors, even for small changes. Use only tmp folder for temporary files.

When working on this codebase:

- Always make sure the build works
- Always read existing code before modifying
- Maintain backward compatibility with cached data
- Consider performance impact on large collections
- Keep privacy-first approach (no external connections)
- Follow TypeScript best practices
- Write tests for new functionality

### Code Style

- Use TypeScript strict mode
- Follow existing naming conventions
- Use functional React components with hooks
- Prefer explicit typing over `any`
- Keep components focused and single-responsibility

**Key Dependencies & Technologies:**

- **Runtime & Build:** `electron` (^38.1.2), `vite` (^8.0.9), `typescript` (^5.2.2), `eslint` (^9.37.0) (Note: package scope is `"type": "module"`)
- **Frontend Framework:** `react` & `react-dom` (^18.2.0)
- **State Management:** `zustand` (^5.0.8)
- **Styling & UI:** `tailwindcss` (^3.4.18), `postcss` (^8.5.6), `autoprefixer` (^10.4.21)
- **Icons & Animations:** `lucide-react` (^0.544.0), `framer-motion` (^12.23.24)
- **UI Components & Virtualization:** `react-window` (^1.8.10), `react-virtualized-auto-sizer` (^1.0.26), `recharts` (^3.4.1), `react-zoom-pan-pinch` (^3.6.1)
- **Testing:** `vitest` (^4.1.4), `@testing-library/react` (^16.3.0), `playwright` (^1.59.1)
- **Metadata & Files:** `exifr` (^7.1.3), `pngjs` (^7.0.0), `cbor-js` (^0.1.0), `chokidar` (^5.0.0)
- **Storage:** IndexedDB for caching

**Key NPM Scripts (from package.json):**

- `npm run dev:app` - Run the full Electron + React dev environment
- `npm run dev` - Run the browser-only dev server
- `npm run build` - Compile TypeScript and build the React output
- `npm test` - Run tests via vitest
- `npm run lint` - Run ESLint
- `npm run electron-dist` - Build distributable without publishing
- `npm run auto-release <version>` - Execute the complete automated CI/CD pipeline
- `npm run cli:parse` / `npm run cli:index` - Execute CLI interface commands

## Project Structure

```
/
├── electron/              # Electron application code
│   ├── main.mjs           # Electron main process
│   ├── preload.js         # Electron preload script
│   └── fileWatcher.mjs    # File system watcher
├── scripts/               # Maintenance, release, and CLI scripts (cli.ts)
├── src/                   # Main React application source
│   ├── components/        # React components
│   ├── services/          # Business logic services
│   ├── store/             # Zustand state management
│   ├── utils/             # Utility functions
│   ├── hooks/             # Custom React hooks
│   ├── App.tsx            # Main React application component
│   ├── types.ts           # TypeScript type definitions
│   └── __tests__/         # Test files
├── docs/                  # Documentation and guides
└── public/                # Static assets
```

- **README.md**: [README.md](README.md) (User-facing documentation and features)
- **ARCHITECTURE.md**: [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) (Technical architecture and design decisions)
- **CHANGELOG.md**: [docs/CHANGELOG.md](docs/CHANGELOG.md) (Version history and changes)
- **RELEASE-GUIDE.md**: [docs/RELEASE-GUIDE.md](docs/RELEASE-GUIDE.md) (Release workflow for maintainers)
- **CLI-README.md**: [docs/CLI-README.md](docs/CLI-README.md) (CLI tool documentation)

## Key Features to Maintain

1. **Privacy**: All processing is local, no external connections
2. **Performance**: Optimized for 18,000+ images with smart caching
3. **Metadata Search**: Full-text search across all metadata fields
4. **Multi-Format Support**: Handle various AI generator formats
5. **File Operations**: Rename, move, delete (desktop only)
6. **Video & GIF Support**: Indexing, playback, and thumbnail support for MP4, WEBM, and GIF files
7. **Shadow Metadata**: View original metadata and revert changes (non-destructive editing)
8. **Subfolder Management**: Ability to exclude specific subfolders from indexing
9. **Auto-Watch**: Real-time monitoring of output folders during generation

## Smart Library & Auto-Tags

- **Clustering & Stacks**: `services/clusteringEngine.ts`, `services/workers/clusteringWorker.ts`, `components/SmartLibrary.tsx`, `components/StackCard.tsx`, `components/StackExpandedView.tsx`
- **Auto-Tags (TF-IDF)**: `services/autoTaggingEngine.ts`, `services/workers/autoTaggingWorker.ts`, `components/TagsAndFavorites.tsx`, `components/ImageModal.tsx`, `components/ImagePreviewSidebar.tsx`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skkut/SilkStack-Image-Browser](https://github.com/skkut/SilkStack-Image-Browser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
