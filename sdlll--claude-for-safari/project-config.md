---
trigger: always_on
description: Control the user's real Safari browser on macOS using AppleScript and screencapture. This skill should be used when the user asks to interact with Safari, browse websites, read web pages, automate browser tasks, take screenshots of web content, or when any task would benefit from seeing or interacting with what's in their browser. Triggers on keywords like "safari", "browser", "web page", "open tab", "screenshot the page", "read this site", "browse", "click on", "fill in the form".
---


# Claude for Safari

Operate the user's real Safari browser on macOS via AppleScript (`osascript`) and `screencapture`. This provides full access to the user's actual browser session — including login state, cookies, and open tabs — without any extensions or additional software.

## Prerequisites

Before first use, verify two settings are enabled. Run this check at the start of every session:

```bash
osascript -e 'tell application "Safari" to get name of front window' 2>&1
```

If this fails, instruct the user to enable:
1. **System Settings > Privacy & Security > Automation** — grant terminal app permission to control Safari
2. **Safari > Settings > Advanced** — enable "Show features for web developers", then **Develop menu > Allow JavaScript from Apple Events**

## Core Capabilities

### 1. List All Open Tabs

```bash
osascript -e '
tell application "Safari"
  set output to ""
  repeat with w from 1 to (count of windows)
    repeat with t from 1 to (count of tabs of window w)
      set tabName to name of tab t of window w
      set tabURL to URL of tab t of window w
      set output to output & "W" & w & "T" & t & " | " & tabName & " | " & tabURL & linefeed
    end repeat
  end repeat
  return output
end tell'
```

### 2. Read Page Content

Read the full text content of the current tab:

```bash
osascript -e '
tell application "Safari"
  do JavaScript "document.body.innerText" in current tab of front window
end tell'
```

Read structured content (title, URL, meta description, headings):

```bash
osascript -e '
tell application "Safari"
  do JavaScript "JSON.stringify({
    title: document.title,
    url: location.href,
    description: document.querySelector(\"meta[name=description]\")?.content || \"\",
    h1: [...document.querySelectorAll(\"h1\")].map(e => e.textContent).join(\" | \"),
    h2: [...document.querySelectorAll(\"h2\")].map(e => e.textContent).join(\" | \")
  })" in current tab of front window
end tell'
```

Read a simplified DOM (similar to Chrome ACP's `browser_read`):

```bash
osascript -e '
tell application "Safari"
  do JavaScript "
    (function() {
      const walk = (node, depth) => {
        let result = \"\";
        for (const child of node.childNodes) {
          if (child.nodeType === 3) {
            const text = child.textContent.trim();
            if (text) result += text + \"\\n\";
          } else if (child.nodeType === 1) {
            const tag = child.tagName.toLowerCase();
            if ([\"script\",\"style\",\"noscript\",\"svg\"].includes(tag)) continue;
            const style = getComputedStyle(child);
            if (style.display === \"none\" || style.visibility === \"hidden\") continue;
            if ([\"h1\",\"h2\",\"h3\",\"h4\",\"h5\",\"h6\"].includes(tag))
              result += \"#\".repeat(parseInt(tag[1])) + \" \";
            if (tag === \"a\") result += \"[\";
            if (tag === \"img\") result += \"[Image: \" + (child.alt || \"\") + \"]\\n\";
            else if (tag === \"input\") result += \"[Input \" + child.type + \": \" + (child.value || child.placeholder || \"\") + \"]\\n\";
            else if (tag === \"button\") result += \"[Button: \" + child.textContent.trim() + \"]\\n\";
            else result += walk(child, depth + 1);
            if (tag === \"a\") result += \"](\" + child.href + \")\\n\";
            if ([\"p\",\"div\",\"li\",\"tr\",\"br\",\"h1\",\"h2\",\"h3\",\"h4\",\"h5\",\"h6\"].includes(tag))
              result += \"\\n\";
          }
        }
        return result;
      };
      return walk(document.body, 0).substring(0, 50000);
    })()
  " in current tab of front window
end tell'
```

### 3. Execute JavaScript

Run arbitrary JavaScript in the page context and get the return value:

```bash
osascript -e '
tell application "Safari"
  do JavaScript "YOUR_JS_CODE_HERE" in current tab of front window
end tell'
```

For multi-line scripts, use a heredoc:

```bash
osascript << 'APPLESCRIPT'
tell application "Safari"
  do JavaScript "
    (function() {
      // Multi-line JS here
      return 'result';
    })()
  " in current tab of front window
end tell
APPLESCRIPT
```

### 4. Screenshot

Two approaches are available. Auto-detect which to use at session start:

```bash
# Test if Screen Recording permission is granted (background screenshot available)
/tmp/safari_wid 2>/dev/null && echo "BACKGROUND_SCREENSHOT=true" || echo "BACKGROUND_SCREENSHOT=false"
```

#### Background Screenshot (requires Screen Recording permission)

If the user has granted Screen Recording permission to the terminal app, use `screencapture -l` to capture Safari **without activating it**:

```bash
# Compile the helper once per session (if not already compiled)
if [ ! -f /tmp/safari_wid ]; then
cat > /tmp/safari_wid.swift << 'SWIFT'
import CoreGraphics
import Foundation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SDLLL/claude-for-safari](https://github.com/SDLLL/claude-for-safari) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
