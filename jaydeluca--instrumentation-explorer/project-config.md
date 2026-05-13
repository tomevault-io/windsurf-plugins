---
trigger: always_on
description: Instrumentation Explorer is a web-based tool for exploring Java instrumentation libraries. It consists of a Python data processing pipeline that enriches instrumentation data and a React frontend that displays the data with search, filter, and comparison capabilities.
---

# Instrumentation Explorer

Instrumentation Explorer is a web-based tool for exploring Java instrumentation libraries. It consists of a Python data processing pipeline that enriches instrumentation data and a React frontend that displays the data with search, filter, and comparison capabilities.

## Primary Documentation

**For complete development guidance, refer to `AGENTS.md` in the repository root.**

The AGENTS.md file contains comprehensive instructions for:
- Project architecture and setup
- Development environment configuration
- Build and test commands
- Code style guidelines
- Pull request requirements
- Common troubleshooting

## Additional References
- `TESTING.md` - Detailed testing procedures including GitHub Pages routing

## Working Effectively

### Bootstrap, Build, and Test the Repository

**CRITICAL: NEVER CANCEL long-running commands. Set timeouts appropriately:**

1. **Install Python dependencies:**
   ```bash
   cd data-processing
   pip install requests pyyaml
   ```
   - Takes ~5 seconds. Use timeout of 60+ seconds.

2. **Run data processing pipeline:**
   ```bash
   cd data-processing
   python3 main.py
   ```
   - Takes ~10 seconds. Use timeout of 120+ seconds. NEVER CANCEL.
   - GitHub API rate limiting causes warnings but does NOT prevent successful completion.
   - Creates `frontend/public/instrumentation-list-enriched.json` (~500KB file).

3. **Install frontend dependencies:**
   ```bash
   # From repository root
   npm install
   ```
   - Takes ~45 seconds. Use timeout of 300+ seconds. NEVER CANCEL.
   - Uses npm workspaces - frontend dependencies are installed automatically.

4. **Build the frontend:**
   ```bash
   cd frontend
   npm run build
   ```
   - Takes ~6 seconds. Use timeout of 120+ seconds.
   - Runs prebuild script that converts YAML to JSON, then TypeScript compilation and Vite build.

5. **Lint the frontend:**
   ```bash
   cd frontend
   npm run lint
   ```
   - Takes ~5 seconds. Use timeout of 60+ seconds.

6. **Run tests:**
   ```bash
   cd frontend
   npm run test:run    # Unit tests
   npm run test:e2e    # E2E tests (requires build first)
   npm run test:all    # All tests (from root)
   ```
   - Unit tests take ~5 seconds. Use timeout of 60+ seconds.
   - E2E tests take ~30 seconds. Use timeout of 180+ seconds.
   - Requires Playwright browsers: `npx playwright install chromium`

### Running the Application

**ALWAYS run the bootstrapping steps first.**

#### Development Mode:
```bash
cd frontend
npm run dev
```
- Application available at `http://localhost:5173/instrumentation-explorer/`
- Hot reload enabled for development.

#### Production Mode:
Use the production build artifacts in `frontend/dist/` with any static file server.

## Validation

**CRITICAL: ALWAYS run through complete end-to-end scenarios after making changes.**

### Required Validation Steps:
1. **Data Pipeline Validation:**
   - Run `python3 main.py` from data-processing directory.
   - Verify `frontend/public/instrumentation-list-enriched.json` is created and contains data.

2. **Frontend Build Validation:**
   - Run `npm run build` from frontend directory.
   - Verify no TypeScript compilation errors.
   - Verify build artifacts are created in `frontend/dist/`.

3. **Application Functionality Validation:**
   - Start dev server with `npm run dev`.
   - Navigate to application in browser.
   - Test key user scenarios:
     - Browse library list on home page.
     - Search/filter libraries by name or semantic convention.
     - Click on a library to view detailed information.
     - Use version comparison tool to compare telemetry differences.
     - Switch between themes (default/grafana).

### CI Pipeline Validation:
**Always run these commands before committing changes:**
```bash
cd frontend
npm run lint
npm run test:run
npm run build
```
- The CI build (.github/workflows/ci-build.yml) will fail if linting, testing, or building fails.
- All PRs automatically run: linting, unit tests, build verification, and E2E tests.

### Testing:
- **Unit Tests (Vitest):** Test utility functions and component logic
- **E2E Tests (Playwright):** Test complete user workflows including GitHub Pages routing
- **GitHub Pages Routing Fix:** The app uses client-side routing with 404.html redirect handling
- Screenshot automation exists but requires Playwright browser installation: `npx playwright install chromium`.
- May fail in some environments due to download limitations - document in instructions as "may not work in all environments due to network restrictions."
- Screenshots are automatically generated in GitHub Actions for PRs.

## Common Tasks

### Repository Structure:
```
.
├── data-processing/          # Python data enrichment pipeline
│   ├── main.py              # Main data processing script
│   ├── requirements.txt     # Python dependencies
│   └── README.md           
├── frontend/                # React/TypeScript frontend application
│   ├── package.json        # Frontend dependencies and scripts
│   ├── src/                # Frontend source code

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jaydeluca/instrumentation-explorer](https://github.com/jaydeluca/instrumentation-explorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
