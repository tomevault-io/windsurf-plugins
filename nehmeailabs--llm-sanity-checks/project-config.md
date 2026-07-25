---
trigger: always_on
description: When to use different agent architectures. Most tasks need simpler tools than you think.
---

# Agent Patterns

When to use different agent architectures. Most tasks need simpler tools than you think.

## The Agent Decision Tree

```
                    You need an agent.
                           │
                           ▼
           ┌───────────────────────────────┐
           │  Does it need real-time web   │
           │  information?                 │──── NO ───► Stuff docs in context.
           └───────────────────────────────┘              No agent needed.
                           │ YES
                           ▼
           ┌───────────────────────────────┐
           │  Can you get it via API or    │──── YES ───► Search API + Fetch URL.
           │  public URL?                  │               Convert HTML to markdown.
           └───────────────────────────────┘               Simple tool agent.
                           │ NO
                           ▼
           ┌───────────────────────────────┐
           │  Does it require UI           │──── NO ───► You probably can get it
           │  interaction? (clicks, forms, │              via API. Look harder.
           │  authentication flows)        │
           └───────────────────────────────┘
                           │ YES
                           ▼
                  Browser automation.
                  (Computer Use, Playwright)
```

---

## Anti-Pattern: Browser Automation for Research

**The mistake:** Using computer use / browser automation to gather information from the web.

**Why it's wrong:**
- 10x slower (rendering, screenshots, visual processing)
- 10x more expensive (vision tokens, multiple screenshots per page)
- More brittle (UI changes break your agent)
- Overkill for reading public content

**The right approach:**

```
Research Task
     │
     ▼
┌──────────┐     ┌───────────┐     ┌──────────────┐     ┌──────────┐
│ Search   │────►│ Fetch URL │────►│ HTML → MD    │────►│ LLM      │
│ API      │     │           │     │ conversion   │     │ synthesis│
└──────────┘     └───────────┘     └──────────────┘     └──────────┘
```

Three tools. No browser. No screenshots. No vision model.

### Example: Deep Research Agent

```python
TOOLS = [
    {
        "name": "web_search",
        "description": "Search the web. Returns list of URLs and snippets.",
    },
    {
        "name": "fetch_url",
        "description": "Fetch a URL and return content as markdown.",
    },
]

# That's it. Two tools.
# The agent can search, read pages, synthesize.
# No Puppeteer. No computer use. No screenshots.
```

### Smart Tool Implementation

Your tools should do the heavy lifting, not the LLM.

**Search tool:**
- Use [SearXNG](https://github.com/searxng/searxng) (self-hosted meta search) instead of paid APIs
- Aggregates Google, Bing, DuckDuckGo — no per-query costs
- Return just: title, URL, snippet. Not raw HTML.

**Fetch tool:**
- Convert HTML → clean markdown
- Strip noise: nav, ads, footers, scripts, sidebars
- Extract main content only (use readability algorithms)
- Truncate if too long (first N chars with "...truncated")
- Return structured: `{title, content, url}`

```python
def fetch_url(url: str) -> dict:
    html = requests.get(url, timeout=10).text
    
    # Extract main content (like Readability)
    article = extract_article(html)
    
    # Convert to markdown
    markdown = html_to_markdown(article.content)
    
    # Truncate on a character boundary that won't split a token mid-way,
    # and never hang the agent on a slow host.
    MAX_CHARS = 15000
    if len(markdown) > MAX_CHARS:
        markdown = markdown[:MAX_CHARS].rsplit(" ", 1)[0] + "\n\n[...truncated]"
    
    return {
        "title": article.title,
        "content": markdown,
        "url": url
    }
```

**Why this matters:**
- Raw HTML is 10x more tokens than clean markdown
- Noise confuses the model
- Preprocessing is cheap, LLM tokens are expensive

### When you actually need browser automation

- **Login walls:** Content behind authentication you can't API
- **Dynamic forms:** Multi-step wizards, dropdowns that load content
- **Actions:** Booking, purchasing, form submission
- **Scraping SPAs:** Content rendered only by JavaScript with no API

If you're just reading and synthesizing public information, you don't need it.

---

## Skip the Framework

Use native LLM tool use instead of LangChain/LlamaIndex abstractions.

**Why:**
- Direct control over prompts, retries, error handling
- No hidden token overhead from framework wrappers
- Easier to debug (you see exactly what's sent/received)
- Fewer dependencies, less breakage

**Native tool use is simple:**

```python
# OpenAI / OpenRouter
response = client.chat.completions.create(
    model="gpt-5",  # or any model with tool support
    messages=messages,
    tools=[{
        "type": "function",
        "function": {
            "name": "web_search",
            "description": "Search the web",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {"type": "string"}
                }
            }
        }
    }]
)

# Check if model wants to call a tool
if response.choices[0].message.tool_calls:
    tool_call = response.choices[0].message.tool_calls[0]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NehmeAILabs/llm-sanity-checks](https://github.com/NehmeAILabs/llm-sanity-checks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
