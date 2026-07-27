---
trigger: always_on
description: 1. **Long-term memory**: If you learn something during a session that would save time in future sessions, add it to `.github/copilot-instructions.md`
---

# copilot-instructions.md - Agent Long-Term Memory

## META-INSTRUCTIONS (IMMUTABLE)

1. **Long-term memory**: If you learn something during a session that would save time in future sessions, add it to `.github/copilot-instructions.md`
2. **Paper knowledge must go**: If something is no longer true, update or remove it immediately
3. **Evaluate after every session**: Consider whether the instructions need updates based on what you learned
4. **Concise and useful**: All information must be actionable, current, and concise

## Code Formatting and Linting

**Before committing code, always run:**
- `yarn lint:prettier:fix` - Format all TypeScript files with Prettier
- `yarn lint:fix` - Fix ESLint and Prettier issues

**Check code quality:**
- `yarn lint` - Check Prettier, ESLint, and spell checking
- `yarn lint:prettier` - Check Prettier formatting only
- `yarn lint:eslint` - Check ESLint only

## Test Commands

**Unit tests:**
- `yarn test` - All unit tests (app + backend)
- `yarn test:app` - Frontend tests only
- `yarn test:backend` - Backend tests only

**LLM integration tests:**
- Requires API key (OpenAI or Gemini)
- **Setup**: Run `./scripts/setup-llm-env.sh` to create `.env.llm-tests` from injected secrets
- **Usage**: `source .env.llm-tests && ./scripts/run-llm-tests.sh`
- **Note**: The `.env.llm-tests` file must be sourced to get the LLM access token before running tests
- Tests make real API calls and cost ~$0.01-$0.05 per run
- See `app/src/services/spec/README.md` for details

**Integration tests:**
- `yarn test:ui` - Browser tests (requires `yarn build` first)
- `yarn test:demo-video` - UI recording (requires Xvfb, mosquitto, tmux, ffmpeg)
- `yarn test:mcp` - Model Context Protocol tests
- `yarn test:all` - All tests (unit + demo-video)
- `./scripts/runBrowserTests.sh` - Browser mode UI tests (requires mosquitto service)
- `./scripts/uiTests.sh` - Demo video tests with Electron (requires Xvfb, mosquitto)

**CI jobs:** `test`, `ui-tests`, `demo-video`, `test-browser`, `browser-ui-tests`

**Important:** 
- Browser UI tests require MQTT broker. In CI, GitHub Actions health checks ensure the mosquitto service is ready before tests run.
- Demo video tests use the same test scenarios as browser tests - if browser tests pass, demo video tests should pass too (they use identical selectors in `src/spec/scenarios/`)

## Debugging Demo Video / UI Tests

When demo video tests fail in CI but you need to debug locally:

**Prerequisites:**
```bash
# Install required packages
sudo apt-get install xvfb mosquitto tmux ffmpeg

# Ensure mosquitto is stopped (script will start its own instance)
sudo systemctl stop mosquitto
```

**Steps to debug:**
1. Build the project: `yarn build`
2. Run the demo video tests: `ELECTRON_DISABLE_SANDBOX=1 ./scripts/uiTests.sh`
3. If tests fail, check the error messages for:
   - Missing `data-test` or `data-test-type` attributes
   - Elements not visible (hidden, outside viewport, or covered by overlays)
   - Click interception (tooltips, dialogs blocking clicks)
   - XPath selector issues (check the data-test value format)

**Common issues:**
- **"locator not visible"**: Element exists but is hidden or outside viewport
- **"locator.click intercepted"**: Another element (tooltip, dialog) is covering the click target
- **Empty `data-test` attribute**: For simple numeric values, ensure you're using the topic name, not `props.literal.path` (which is empty for non-JSON values)
- **"Process failed to launch"**: Electron can't start - ensure DISPLAY is set and Xvfb is running

**Environment-specific notes:**
- Demo video tests use Electron with Xvfb (virtual display)
- Browser tests use Chromium without Electron (easier to debug locally)
- CI environment has proper Electron setup - if local tests are flaky, trust CI results
- Both test types use the same scenario files in `src/spec/scenarios/`

**Material-UI Tooltip considerations:**
- Tooltips wrap their children and create overlay divs
- Test attributes (`data-test-type`, `data-test`) must be on the inner clickable element that's passed as a child to the Tooltip
- Mouse event handlers (onMouseEnter, onMouseLeave, ref) go on outer wrapper
- onClick handler and test attributes go on the span inside the Tooltip
- The clickable child inside the Tooltip is what Playwright should target

**Example:**
```tsx
// ❌ WRONG - attributes on outer wrapper, Tooltip wraps and hides them
<Tooltip title="...">
  <span onClick={...} data-test-type="Button" data-test="example">
    <Icon />
  </span>
</Tooltip>

// ✅ CORRECT - attributes on the actual clickable child inside Tooltip
<span ref={ref} onMouseEnter={...} onMouseLeave={...}>
  <Tooltip title="...">
    <span onClick={...} data-test-type="Button" data-test="example">
      <Icon />
    </span>
  </Tooltip>
</span>
```

**data-test value format:**
- ShowChart: Use last segment of topic path (e.g., "heater" for "kitchen/coffee_maker/heater")
- ChartSettings: Use full topic + dotPath (e.g., "kitchen/coffee_maker/heater-" with trailing dash when dotPath is empty)
- Test XPaths use `contains(@data-test, "substring")` so partial matches work

## Running Browser Tests Locally

**Prerequisites:**
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thomasnordquist/MQTT-Explorer](https://github.com/thomasnordquist/MQTT-Explorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
