---
trigger: always_on
description: command line tools that can be used in this project
---

<tools>
# Internal Tools: Core capabilities available directly.
run_terminal_cmd
codebase_search
read_file
list_dir
grep_search
edit_file
file_search
delete_file
reapply
web_search
mcp_puppeteer_puppeteer_navigate
mcp_puppeteer_puppeteer_screenshot
mcp_puppeteer_puppeteer_click
mcp_puppeteer_puppeteer_fill
mcp_puppeteer_puppeteer_select
mcp_puppeteer_puppeteer_hover
mcp_puppeteer_puppeteer_evaluate
</tools>

<command_line_tools>
# Project-Specific Tools: Execute these using `run_terminal_cmd`.
# 🔧 Setup: Run `npm install` and configure API keys in `.env.local` (see `.env.example`)
{
  "tools": {
    "image-optimizer": {
      "description": "Optimizes images with background removal, resizing, and format conversion using Sharp and Replicate's remove-bg model",
      "tool": "run_terminal_cmd",
      "command": "npm run optimize-image",
      "status": "✅ Tested and working",
      "options": {
        "input": "Path to input image",
        "output": "Path to output image",
        "remove-bg": "(Optional) Remove image background using AI",
        "resize": "(Optional) Resize image (format: WIDTHxHEIGHT, e.g. 800x600)",
        "format": "(Optional) Convert to format (png, jpeg, or webp)",
        "quality": "(Optional) Set output quality (1-100, default: 80)"
      },
      "requires": [
        "REPLICATE_API_TOKEN in .env.local (for background removal)",
      ],
      "example": "npm run optimize-image -- -i input.png -o output.webp --resize 512x512 --format webp --quality 90",
      "note": "Use -i and -o flags for input/output. Background removal requires Replicate API token."
    },
    "html-to-md": {
      "description": "Scrapes a webpage and converts its HTML content to Markdown format using Turndown service",
      "tool": "run_terminal_cmd",
      "command": "npm run html-to-md",
      "status": "✅ Tested and working",
      "options": {
        "url": "URL of the webpage to scrape",
        "output": "(Optional) Output file path for the markdown (default: output.md)",
        "selector": "(Optional) CSS selector to target specific content"
      },
      "requires": [
      ],
      "example": "npm run html-to-md -- --url https://example.com --output docs/scraped.md --selector main"
    },
    "gemini": {
      "description": "Interacts with Google's Gemini API for text generation, chat, multimodal tasks, document analysis, and grounded search",
      "tool": "run_terminal_cmd",
      "command": "npm run gemini",
      "status": "✅ Tested and working",
      "options": {
        "prompt": "Text prompt or question for the model",
        "model": "(Optional) Model to use: 'gemini-2.0-flash' (default), 'gemini-2.5-pro-exp-03-25'",
        "temperature": "(Optional) Sampling temperature between 0.0 and 1.0 (default: 0.7)",
        "max-tokens": "(Optional) Maximum tokens to generate (default: 2048)",
        "image": "(Optional) Path to image file for vision tasks",
        "file": "(Optional) Path to local file (PDF, DOCX, TXT, etc.) for document analysis",
        "url": "(Optional) URL to a document to analyze (PDF, DOCX, TXT, etc.)",
        "mime-type": "(Optional) MIME type of the file (e.g., application/pdf, default: auto-detected)",
        "chat-history": "(Optional) Path to JSON file containing chat history",
        "stream": "(Optional) Stream the response (default: false)",
        "safety-settings": "(Optional) JSON string of safety threshold configurations",
        "schema": "(Optional) JSON schema for structured output",
        "json": "(Optional) Return structured JSON data. Available types: recipes, tasks, products, custom",
        "ground": "(Optional) Enable Google Search grounding for up-to-date information (default: false)",
        "show-search-data": "(Optional) Show the search entries used for grounding (default: false)"
      },
      "requires": [
        "GOOGLE_AI_STUDIO_KEY or GEMINI_API_KEY in .env.local",
        "@google/generative-ai package (auto-installed with npm install)",
        "node-fetch package (auto-installed with npm install)"
      ],
      "example": "npm run gemini -- --prompt \"What is the capital of France?\" --model gemini-2.0-flash --temperature 0.7",
      "advanced_examples": [
        "# Process a PDF document from a URL",
        "npm run gemini -- --prompt \"Summarize this document in 5 key points\" --url \"https://discovery.ucl.ac.uk/id/eprint/10089234/1/343019_3_art_0_py4t4l_convrt.pdf\"",
        "",
        "# Process a local PDF file",
        "npm run gemini -- --prompt \"What is this document about?\" --file test/data/sample.pdf",
        "",
        "# Process a text file with specific MIME type",
        "npm run gemini -- --prompt \"Expand on this information\" --file test/data/sample.txt --mime-type text/plain",
        "",
        "# Get grounded search results with real-time information",
        "npm run gemini -- --prompt \"When is the next total solar eclipse in North America?\" --ground --show-search-data",
        "",
        "# Generate structured JSON data with predefined schema (recipes)",
        "npm run gemini -- --prompt \"List 3 popular cookie recipes\" --json recipes",
        "",
        "# Generate structured JSON data with predefined schema (tasks)",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LastBotInc/nextjs-supabase-ai-webapp](https://github.com/LastBotInc/nextjs-supabase-ai-webapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
