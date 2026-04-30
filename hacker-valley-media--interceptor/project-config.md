---
trigger: always_on
description: Browser control CLI for AI agents. No CDP, no MCP, no API keys. You call `interceptor`, read the output, decide what's next.
---

# interceptor

Browser control CLI for AI agents. No CDP, no MCP, no API keys. You call `interceptor`, read the output, decide what's next.

**Binary:** `dist/interceptor`

## Start Here

```bash
interceptor open "https://example.com"        # Open, wait, return tree + text (1 command)
interceptor act e1                            # Click element, return updated tree + diff
interceptor act e2 "hello"                    # Type into field, return updated tree
interceptor read                              # Re-read current page (tree + text)
interceptor inspect                           # Tree + text + network log + headers
```

The daemon auto-starts. When working inside this repo, prefer `./dist/interceptor ...` if `interceptor` is not on `PATH`.

### Legacy (still works, but prefer compound commands above)

```bash
interceptor tab new "https://example.com"    # Open managed tab
sleep 2                                # Wait for load
interceptor tree                              # See interactive elements
interceptor click e1                          # Click by ref
interceptor type e2 "hello"                   # Type into field
interceptor text                              # Read visible text
```

## Use Cases

The [`use-cases/`](use-cases/) folder is where proven browser workflows get turned into reusable documentation. When you figure out how to do something reliably on a live site, add a compact use-case there so later agents can follow the path instead of burning tokens rediscovering it.

## Compound Commands (Agent-Optimized)

These collapse multi-step patterns into single CLI invocations. Prefer these over the individual commands below.

```bash
interceptor open "https://example.com"        # tab new + wait + tree + text in one call
interceptor open "https://example.com" --tree-only   # Skip text
interceptor open "https://example.com" --text-only   # Skip tree
interceptor open "https://example.com" --full        # Full text (no 2000-char limit)
interceptor open "https://example.com" --timeout 10000  # Custom wait timeout
interceptor open "https://example.com" --no-wait     # Don't wait for load
interceptor read                              # Tree + text for current page
interceptor read e5                           # Tree + text for element subtree
interceptor read --tree-only                  # Just tree
interceptor read --text-only                  # Just text
interceptor act e5                            # Click + wait + return updated tree + diff
interceptor act e3 "hello"                    # Type + wait + return updated tree
interceptor act e5 --os                       # OS-level trusted click
interceptor act e5 --keys "Enter"             # Send keyboard shortcut instead
interceptor act e5 --no-read                  # Skip post-action tree read
interceptor inspect                           # Tree + text + net log + headers
interceptor inspect --net-only                # Just network data
interceptor inspect --filter api              # Filter network entries
```

## Reading Pages

```bash
interceptor tree                              # Interactive elements with refs (e1, e2...)
interceptor tree --filter all                 # Include headings + landmarks
interceptor text                              # All visible text
interceptor text e7                           # Text from specific element
interceptor html e5                           # HTML of element
interceptor find "Submit"                     # Find elements by name
interceptor find "Submit" --role button       # Filter by ARIA role
interceptor diff                              # What changed since last tree
interceptor state                             # DOM tree + scroll + focus (verbose)
```

## Interacting With Pages

```bash
interceptor click e5                          # Click element
interceptor click e5 --os                     # OS-level trusted click (bypasses isTrusted)
interceptor type e3 "hello"                   # Type into field (clears first)
interceptor type e3 "more" --append           # Append without clearing
interceptor select e7 "option-value"          # Select dropdown option
interceptor hover e5                          # Hover over element
interceptor keys "Enter"                      # Keyboard shortcut
interceptor keys "Control+A" --os             # OS-level keyboard
interceptor scroll down                       # Scroll
```

When a synthetic click doesn't trigger anything (React/Angular sites), interceptor auto-escalates to OS-level input. You can also force it with `--os`.

## Navigating

```bash
interceptor tab new "https://example.com"     # New tab (joins interceptor group)
interceptor navigate "https://other.com"      # Navigate current tab
interceptor tabs                              # List all tabs (* = active)
interceptor tab switch 12345                  # Switch to tab by ID
interceptor tab close                         # Close current tab
interceptor back                              # History back
interceptor forward                           # History forward
interceptor wait 2000                         # Wait milliseconds
interceptor wait-stable                       # Wait for DOM to stop changing
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hacker-Valley-Media/Interceptor](https://github.com/Hacker-Valley-Media/Interceptor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
