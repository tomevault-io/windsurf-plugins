---
trigger: always_on
description: |
---


# X Topic Intelligence

Research what's trending and performing on X/Twitter for any topic. Two search backends available — pick based on the task complexity.

---

## Usage

```
/x-collect [topic]        # auto-select mode based on topic complexity
/x-collect                # interactive mode - asks for topic
```

When no topic is provided, use `AskUserQuestion`:
"What topic do you want to research on X/Twitter? (e.g., 'AI agents', 'vibe coding', 'indie hacking')"

---

## Search Mode Selection

| | Grok Mode (quick) | Actionbook Scrape Mode (full) |
|---|---|---|
| **Speed** | ~60s single query | ~5-10 min multi-round |
| **Best for** | Quick overview, trend check, "what's hot" | Systematic collection, precise metrics, dataset building |
| **Data quality** | AI-summarized, may miss some posts | Raw DOM data, exact engagement numbers, includes views |
| **Search operators** | No (natural language only) | Yes (`min_faves:`, `filter:blue_verified`, date ranges, etc.) |
| **Output** | 5-15 posts per query | 10-15 posts per round, 3-5 rounds |
| **Views count** | Not available | Available |

### Auto-selection rules

Use **Grok Mode** when:
- User asks a simple question ("what's trending about X?", "find popular posts about Y")
- Quick exploration / first look at a topic
- Single broad query is sufficient
- User explicitly says "quick" or "fast"

Use **Actionbook Scrape Mode** when:
- User wants systematic multi-round collection (viral + trending + KOL + hook study)
- Need precise engagement thresholds (`min_faves:100`, `filter:blue_verified`)
- Need views count
- Building a JSONL dataset for later analysis
- User explicitly says "detailed", "full", or "all rounds"

> When unsure, default to **Grok Mode** — it's faster and sufficient for most requests. Escalate to Actionbook Scrape if Grok results are insufficient or user asks for more.

---

## Prerequisites

| Dependency | Purpose | Required |
|------------|---------|----------|
| Claude Code | Runtime | Yes |
| Actionbook CLI | Browser automation (`actionbook` command) | Yes |
| Actionbook Extension | Chrome extension + bridge for controlling user's browser | Yes |
| Logged-in X/Twitter session | User's Chrome must have an active x.com session | Yes |
| X Premium (for Grok) | Grok search requires Premium subscription | Only for Grok Mode |

### Pre-flight Check

Before starting any round, verify the extension bridge is running:

```bash
actionbook extension ping
```

If not connected, inform the user:
> "Actionbook extension bridge is not running. Please run `actionbook extension serve` in a terminal and ensure the Chrome extension is connected."

---

## Grok Mode (Quick Search)

Ask Grok on x.com to search X and return structured results. One query, ~60 seconds.

### Grok Workflow

#### 1. Navigate to Grok

```bash
actionbook --extension browser open "https://x.com/i/grok"
```

Wait 4 seconds for page load, then verify textarea exists:

```bash
actionbook --extension browser eval "document.querySelector('textarea') ? 'ready' : 'not_ready'"
```

> Use `browser open` (new tab) instead of `browser goto` to avoid "Cannot access chrome:// URL" errors.

#### 2. Input the Question

Craft a Grok prompt that returns structured data. Template:

```
Search X for the most popular posts about [TOPIC] in the past [TIME_RANGE]. Show me the top [N] posts with the most engagement (likes + retweets). For each post, include: the author handle, post text, likes count, retweets count, and the post URL.
```

Set the value via JS (React state-safe):

```bash
actionbook --extension browser eval "
  var textarea = document.querySelector('textarea');
  if (textarea) {
    var setter = Object.getOwnPropertyDescriptor(window.HTMLTextAreaElement.prototype, 'value').set;
    setter.call(textarea, 'YOUR GROK PROMPT HERE');
    textarea.dispatchEvent(new Event('input', { bubbles: true }));
    'ok'
  } else { 'no_textarea' }
"
```

#### 3. Send the Question

```bash
actionbook --extension browser eval "
  var btns = document.querySelectorAll('button');
  var sent = false;
  for (var i = 0; i < btns.length; i++) {
    var svg = btns[i].querySelector('svg');
    var rect = btns[i].getBoundingClientRect();
    if (svg && rect.width > 20 && rect.width < 60 && rect.height > 20 && rect.height < 60 && rect.x > 600) {
      btns[i].click();
      sent = true;
      break;
    }
  }
  sent ? 'sent' : 'not_found'
"
```

If `not_found`, take a screenshot (`actionbook --extension browser screenshot`) and debug visually.

#### 4. Wait for Response

Grok takes 10-60 seconds (longer when searching X). Poll every 8 seconds:

```bash
actionbook --extension browser eval "document.querySelector('main') ? document.querySelector('main').innerText.length : 0"
```

Response is ready when length > 500. Timeout after 90 seconds and extract whatever is available.

#### 5. Extract Response

```bash
actionbook --extension browser eval "document.querySelector('main').innerText"
```

#### 6. Parse into JSONL Schema

Parse Grok's text response and map each post to the standard JSONL schema. For fields Grok doesn't provide:
- `views`: set to `"0"` (Grok doesn't return views)
- `replies`: set to `0` if not provided
- `posted_at`: set to `null` if not provided
- `angle` and `key_takeaway`: analyze and fill in yourself

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SamCuipogobongo/x-collect](https://github.com/SamCuipogobongo/x-collect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
