---
trigger: always_on
description: During you interaction with the user, if you find anything reusable in this project (e.g. version of a library, model name), especially about a fix to a mistake you made or a correction you received, you should take note in the `Lessons` section in the `.cursorrules` file so you will not make the same mistake again.
---

# Instructions

During you interaction with the user, if you find anything reusable in this project (e.g. version of a library, model name), especially about a fix to a mistake you made or a correction you received, you should take note in the `Lessons` section in the `.cursorrules` file so you will not make the same mistake again. 

You should also use the `.cursorrules` file as a scratchpad to organize your thoughts. Especially when you receive a new task, you should first review the content of the scratchpad, clear old different task if necessary, first explain the task, and plan the steps you need to take to complete the task. You can use todo markers to indicate the progress, e.g.
[X] Task 1
[ ] Task 2
Also update the progress of the task in the Scratchpad when you finish a subtask.
Especially when you finished a milestone, it will help to improve your depth of task accomplishment to use the scratchpad to reflect and plan.
The goal is to help you maintain a big picture as well as the progress of the task. Always refer to the Scratchpad when you plan the next step.

# Tools

Note all the tools are in python. So in the case you need to do batch processing, you can always consult the python files and write your own script.

## LLM

You always have an LLM at your side to help you with the task. For simple tasks, you could invoke the LLM by running the following command:
```
py310/bin/python ./tools/llm_api.py --prompt "What is the capital of France?"
```

But usually it's a better idea to check the content of the file and use the APIs in the `tools/llm_api.py` file to invoke the LLM if needed.

## Web browser

You could use the `tools/web_scraper.py` file to scrape the web.
```
py310/bin/python ./tools/web_scraper.py --max-concurrent 3 URL1 URL2 URL3
```
This will output the content of the web pages.

## Search engine

You could use the `tools/search_engine.py` file to search the web.
```
py310/bin/python ./tools/search_engine.py "your search keywords"
```
This will output the search results in the following format:
```
URL: https://example.com
Title: This is the title of the search result
Snippet: This is a snippet of the search result
```
If needed, you can further use the `web_scraper.py` file to scrape the web page content.

# Lessons

## User Specified Lessons

- You have a python venv in ./py310.
- Include info useful for debugging in the program output.
- Read the file before you try to edit it.
- Use LLM to perform flexible text understanding tasks. First test on a few files. After success, make it parallel.
- Keep article and image processing together in the same thread for better efficiency

## Cursor learned

- For website image paths, always use the correct relative path (e.g., 'images/filename.png') and ensure the images directory exists
- For search results, ensure proper handling of different character encodings (UTF-8) for international queries
- Add debug information to stderr while keeping the main output clean in stdout for better pipeline integration
- When using seaborn styles in matplotlib, use 'seaborn-v0_8' instead of 'seaborn' as the style name due to recent seaborn version changes

## Web Scraping Improvements

### Directory Structure
- Create flat directory structure for articles instead of nested ones
- Each article should have its own directory with the article name
- Store article content in a markdown file with the same name as the directory
- Keep images in an 'images' subdirectory within each article directory

### Content Processing
- Skip favicon.png when downloading images
- Remove Google notification content from articles
- Convert HTML to Markdown while preserving formatting
- Use relative paths for all internal links
- Handle Chinese characters and special symbols in filenames properly

### Performance & Reliability
- Implement rate limiting with random delays between requests
- Add retry mechanism for failed requests
- Handle HTTP 429 (Too Many Requests) with exponential backoff
- Process one section at a time for testing and validation

### Code Organization
- Use separate methods for different responsibilities (downloading, processing, saving)
- Maintain a set of processed URLs to avoid duplicates
- Implement proper error handling and logging
- Keep track of progress with informative log messages

# Version History

## Version 1.0 - Basic Scraping Functionality
- [X] Basic website structure scraping
- [X] Content organization in a clean directory structure
- [X] HTML to Markdown conversion with formatting preserved
- [X] Image downloading and proper path handling
- [X] Rate limiting and retry mechanism
- [X] Error handling and logging
- [X] Clean file naming and sanitization
- [X] Proper handling of nested content
- [X] Progress tracking and duplicate URL prevention

## Version 2.0 - Performance Improvements (Planning)
[ ] Multi-threading implementation
  - Use ThreadPoolExecutor to process articles in parallel
  - Each thread handles one complete article (including its images)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ly-GGboy/AI-Library](https://github.com/Ly-GGboy/AI-Library) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
