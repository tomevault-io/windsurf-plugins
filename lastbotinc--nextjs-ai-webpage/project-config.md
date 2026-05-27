---
trigger: always_on
description: This is a NextJS 15 / React 19 / Typescript / Tailwind CSS application with Jest and Cypress test frameworks.
---

<project>
This is a NextJS 15 / React 19 / Typescript / Tailwind CSS application with Jest and Cypress test frameworks.
</project>

<tools>
#internal Cursor tools used
codebase_search - For semantic search in the codebase
read_file - For reading file contents
run_terminal_cmd - For running terminal commands
list_dir - For listing directory contents
grep_search - For text-based regex search
file_search - For fuzzy file path search
delete_file - For deleting files
</tools>

<project_tools>
#Project specific tools, please run as commands with tool defined in tool-attribute
{
  "tools": {
    "recraft": {
      "description": "Generates images using Recraft V3 API (SOTA text-to-image model)",
      "tool": "run_terminal_cmd",
      "command": "npm run recraft",
      "options": {
        "prompt": "Text description of the desired image",
        "style": "(Optional) Image style to use (default: digital_illustration). Available styles:\n
        - realistic_image (and variants: b_and_w, hard_flash, hdr, natural_light, studio_portrait, enterprise, motion_blur)\n
        - digital_illustration (and variants: pixel_art, hand_drawn, grain, infantile_sketch, 2d_art_poster, handmade_3d, hand_drawn_outline, engraving_color, 2d_art_poster_2)",
        "negative_prompt": "(Optional) Things to avoid in the image",
        "width": "(Optional) Image width in pixels. Available sizes: 1024, 1365, 1536, 1820, 2048, 1434, 1280, 1707 (default: 1024)",
        "height": "(Optional) Image height in pixels. Available sizes: 1024, 1365, 1536, 1820, 2048, 1434, 1280, 1707 (default: 1024)",
        "num_outputs": "(Optional) Number of images to generate (default: 1)",
        "scheduler": "(Optional) Sampling method to use",
        "num_inference_steps": "(Optional) Number of denoising steps (default: 50)",
        "guidance_scale": "(Optional) How closely to follow the prompt (default: 7.5)",
        "seed": "(Optional) Random seed for reproducibility",
        "folder": "(Optional) Output folder path",
        "filename": "(Optional) Output filename"
      },
      "requires": ["REPLICATE_API_TOKEN in .env"],
      "example": "npm run recraft -- --prompt \"A modern logo with blue background\" --style digital_illustration --folder public/images --filename logo.png"
    },
    "image-optimizer": {
      "description": "Optimizes images with background removal, resizing, and format conversion",
      "tool": "run_terminal_cmd",
      "command": "npm run optimize-image",
      "options": {
        "input": "Path to input image",
        "output": "Path to output image",
        "remove-bg": "(Optional) Remove image background using AI",
        "resize": "(Optional) Resize image (format: WIDTHxHEIGHT, e.g. 800x600)",
        "format": "(Optional) Convert to format (png, jpeg, or webp)",
        "quality": "(Optional) Set output quality (1-100, default: 80)"
      },
      "requires": [
        "REPLICATE_API_TOKEN in .env",
        "sharp package (npm install sharp)"
      ],
      "example": "npm run optimize-image -- input.png output.webp --resize 512x512 --format webp --quality 90"
    },
    "read-url": {
      "description": "Scrapes a webpage and converts its HTML content to Markdown format",
      "tool": "run_terminal_cmd",
      "command": "npm run html-to-md",
      "options": {
        "url": "URL of the webpage to scrape",
        "output": "(Optional) Output file path for the markdown (default: output.md)",
        "selector": "(Optional) CSS selector to target specific content"
      },
      "requires": [
        "Node.js >= 14",
        "Internet connection for scraping"
      ],
      "example": "npm run html-to-md -- --url https://example.com --output docs/scraped.md --selector main"
    },
    "tavily-search": {
      "description": "Executes AI-powered web search using Tavily API with options for search type, depth, and domain filtering",
       "tool": "run_terminal_cmd",
      "command": "npm run tavily-search",
      "options": {
        "query": "Search query text",
        "type": "(Optional) Search type: 'search' (default), 'context', or 'qna' for question and answer",
        "depth": "(Optional) Search depth: 'basic' (default) or 'advanced'",
        "max-results": "(Optional) Maximum number of results (default: 5)",
        "include": "(Optional) Comma-separated list of domains to include",
        "exclude": "(Optional) Comma-separated list of domains to exclude"
      },
      "requires": [
        "TAVILY_API_KEY in .env",
        "@tavily/core package"
      ],
      "example": "npm run tavily-search -- --query \"Next.js best practices\" --type context --depth advanced --max-results 10"
    },
    "download-file": {
      "description": "Downloads files (especially images) from URLs with progress tracking",
      "tool": "run_terminal_cmd",
      "command": "npm run download-file",
      "options": {
        "url": "URL of the file to download",
        "output": "(Optional) Complete output path including filename",
        "folder": "(Optional) Output folder path (default: downloads)",
        "filename": "(Optional) Output filename (if not provided, derived from URL or content)"
      },
      "requires": [
        "axios package",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LastBotInc/nextjs-ai-webpage](https://github.com/LastBotInc/nextjs-ai-webpage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
