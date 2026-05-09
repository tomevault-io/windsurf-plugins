---
trigger: always_on
description: This file, `safari.mdc`, serves as a repository for detailed working notes, observations, and learnings acquired during the process of automating Safari interactions, particularly for the MCP Inspector UI. It's intended to capture the nuances of trial-and-error, debugging steps, and insights into what worked, what didn't, and why.
---

### Meta Note

This file, `safari.mdc`, serves as a repository for detailed working notes, observations, and learnings acquired during the process of automating Safari interactions, particularly for the MCP Inspector UI. It's intended to capture the nuances of trial-and-error, debugging steps, and insights into what worked, what didn't, and why.

This contrasts with `mcp-inspector.mdc`, which is designed to be the concise, polished, and operational ruleset for future automated runs once a specific automation flow (like connecting to the MCP Inspector) has been stabilized and proven reliable. `mcp-inspector.mdc` should contain the 'final' working scripts and minimal necessary commentary, while `safari.mdc` is the space for the extended antechamber of discovery.

---

### Key Learnings and Observations from Safari Automation (MCP Inspector)

#### 1. Managing Safari Windows and Tabs for the Inspector

*   **Objective:** Reliably direct Safari to the MCP Inspector URL (`http://127.0.0.1:6274`) in a predictable way, preferably using a single, consistent browser window and tab to avoid disrupting the user's workspace or losing context.
*   **Initial Challenges & Evolution:
    *   Simply using `make new document with properties {URL:"..."}` could lead to multiple windows/tabs if not managed.
    *   Attempts to close all existing Inspector tabs first (`repeat with w in windows... close t...`) were functional but could be overly aggressive if the user had other work in Safari.
    *   Identifying and reusing an *existing specific tab* for the Inspector requires careful targeting (e.g., `first tab whose URL starts with "..."`). If this tab was from a previous, unconfigured session, just switching to it wasn't enough; it needed to be reloaded/reset.
*   **Refined & Recommended Approach (as implemented in `mcp-inspector.mdc`):
    ```applescript
    tell application "Safari"
      activate
      delay 0.2 -- Allow Safari to become the frontmost application
      if (count of windows) is 0 then
        -- No Safari windows are open, so create a new one.
        make new document with properties {URL:"http://127.0.0.1:6274"}
      else
        -- Safari has windows open; use the frontmost one.
        tell front window
          set inspectorTab to missing value
          try
            -- Check if a tab for the Inspector is already open in this window.
            set inspectorTab to (first tab whose URL starts with "http://127.0.0.1:6274")
          end try
          
          if inspectorTab is not missing value then
            -- An Inspector tab exists: set its URL again (to refresh/reset) and make it active.
            set URL of inspectorTab to "http://127.0.0.1:6274"
            set current tab to inspectorTab
          else
            -- No specific Inspector tab found: set the URL of the *current active tab*.
            set URL of current tab to "http://127.0.0.1:6274"
          end if
        end tell
      end if
      delay 1 -- Pause to allow the page to begin loading.
    end tell
    ```
    This logic aims to use the existing front window and either reuse/refresh an Inspector tab or repurpose the current active tab, falling back to creating a new window only if Safari isn't open.

#### 2. Clicking Elements Programmatically (The "Connect" Button Saga)

*   **The Core Challenge:** Programmatically clicking the "Connect" button in the MCP Inspector UI to initiate the server connection.
*   **Strategies Explored & Lessons:
    *   **CSS Selectors (`querySelector`):**
        *   Simple selectors like `[data-testid='env-vars-button']` worked for some buttons but required escaping single quotes in AppleScript: `do JavaScript "document.querySelector('[data-testid=\\\'env-vars-button\\']').click();"`.
        *   A complex `querySelector` for the "Connect" button (e.g., `'button[data-testid*=connect-button], button:not([disabled])... > span:contains(Connect)...'.click()`) ran without JS error but didn't reliably establish the connection, suggesting it might not have found the exact interactable element or the click wasn't registering correctly.
    *   **XPath (`document.evaluate`):**
        *   **Highly Specific XPaths:** An initial XPath based on the rule (`//button[contains(., 'Connect') and .//svg[.//polygon[@points='6 3 20 12 6 21 6 3']]]`) was very difficult to embed correctly in AppleScript due to nested single quotes requiring complex escaping (`\'`). This often led to AppleScript parsing errors (`-2741`).
        *   **`character id 39` for AppleScript String Construction:** To combat escaping issues, building the JavaScript string in AppleScript using `set sQuote to character id 39` for internal single quotes was effective for getting the AppleScript parser to accept the command. Example:
            ```applescript
            set sQuote to character id 39
            set jsConnectText to "Connect"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [steipete/macos-automator-mcp](https://github.com/steipete/macos-automator-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
