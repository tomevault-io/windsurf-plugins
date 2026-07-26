---
trigger: always_on
description: **CRITICAL: Always follow these instructions first and only fallback to search or bash commands when the information here is incomplete or found to be in error.**
---

# ioBroker.admin Development Instructions

**CRITICAL: Always follow these instructions first and only fallback to search or bash commands when the information here is incomplete or found to be in error.**

## 🚨 CRITICAL ISSUE MANAGEMENT POLICY

**NEVER automatically close issues with PR merges. All issues must be manually validated before closing.**

### Issue Handling Rules
1. **NEVER** use keywords that auto-close issues in PR titles, descriptions, or commit messages:
   - Prohibited: "fix", "fixes", "fixed", "close", "closes", "closed", "resolve", "resolves", "resolved"
   - Use instead: "address", "implement", "handle", "improve", "related to", "regarding"

2. **Reference issues neutrally**: Use "Related to #123" instead of "Fixes #123"

3. **Add 'fixed' label**: When a PR fully addresses an issue, manually add the "fixed" label to the issue (if you have permissions)

4. **Manual validation required**: Issues should only be closed manually by maintainers after proper review and validation, not automatically by PR merges

---

ioBroker.admin is a complex monorepo that provides the web-based administration interface for ioBroker home automation systems. It consists of a React/TypeScript frontend, Express.js backend, and multiple shared component packages.

## Working Effectively

### Required Node.js Version
- **ALWAYS use Node.js 20** - specified in `.nvmrc`
- Check version: `node --version` should show v20.x.x

### Bootstrap and Install Dependencies
- **Environment Setup**: Set `PUPPETEER_SKIP_DOWNLOAD=true` to avoid network issues with Chrome downloads
- **Install Command**: `PUPPETEER_SKIP_DOWNLOAD=true npm run install-monorepo`
- **Install Time**: Takes 1-2 minutes. NEVER CANCEL.
- **Expected Output**: Installs dependencies for root + 4 workspace packages (adapter-react-v5, dm-gui-components, jsonConfig, admin)

### Build Process
- **Build Command**: `npm run build`
- **Build Time**: Takes 1-2 minutes. NEVER CANCEL. Set timeout to 5+ minutes.
- **What it does**: 
  - Cleans previous builds with `npm run clean` 
  - Runs Lerna to build all 4 packages in dependency order
  - Frontend builds with Vite + TypeScript
  - Backend compiles TypeScript to `build-backend/`
  - Frontend builds to `adminWww/`

### Development Server
- **Start Command**: `npm run start`
- **URL**: http://localhost:3000/ (Vite dev server)
- **Startup Time**: 10-15 seconds. Development server will show "VITE ready" message.
- **What it runs**: Vite development server with hot reload for React frontend
- **Backend**: For full functionality, you need a running ioBroker instance (typically on port 8081)

## Validation and Testing

### Linting
- **Command**: `npm run lint`
- **Time**: 30-60 seconds. NEVER CANCEL.
- **What it checks**: Backend and frontend TypeScript/JavaScript code across all packages
- **Expected**: May show some linting errors in development - this is normal

### Testing
- **Unit Tests**: `npm test` (in packages/admin/) - Takes <5 seconds
- **Package Tests**: Validates package.json and io-package.json structure
- **GUI Tests**: `npm run test:gui` - **REQUIRES Chrome/Puppeteer** (will fail in restricted environments)

### Manual Validation Steps
**ALWAYS perform these validation steps after making changes:**
1. **Build Validation**: Run `npm run build` and ensure it completes without errors
2. **Development Server**: Start `npm run start` and verify it loads at http://localhost:3000/
3. **Linting**: Run `npm run lint` to check code style (some errors are acceptable in development)
4. **Basic UI Test**: Open http://localhost:3000/ and verify the admin interface loads (shows ioBroker admin panel)

## Project Structure and Key Locations

### Monorepo Layout
```
/
├── packages/
│   ├── admin/                    # Main adapter package
│   │   ├── src/                  # Backend TypeScript source
│   │   ├── src-admin/            # React frontend source
│   │   ├── adminWww/            # Built frontend (generated)
│   │   ├── build-backend/       # Built backend (generated)
│   │   └── test/                # Test files
│   ├── adapter-react-v5/        # React components for ioBroker adapters
│   ├── dm-gui-components/       # Device management UI components  
│   └── jsonConfig/              # JSON-based configuration system
└── package.json                 # Root package with workspace scripts
```

### Important Files
- **Frontend Entry**: `packages/admin/src-admin/src/index.tsx`
- **Backend Entry**: `packages/admin/src/main.ts`
- **Build Scripts**: `packages/admin/tasks.js`
- **Frontend Config**: `packages/admin/src-admin/vite.config.ts`
- **JSON Config Schema**: `packages/jsonConfig/SCHEMA.md`

### Key Commands Reference
```bash
# Quick setup (run once)
PUPPETEER_SKIP_DOWNLOAD=true npm run npm

# Development workflow
npm run build              # Build all packages (1-2 min)
npm run start             # Start dev server on :3000
npm run lint              # Check code style
npm test                  # Run tests (in packages/admin/)

# Individual package operations
npm run build:backend -w packages/admin    # Build only backend
npm run start -w packages/admin           # Start only admin dev server
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ioBroker/ioBroker.admin](https://github.com/ioBroker/ioBroker.admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
