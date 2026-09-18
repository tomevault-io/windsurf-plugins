---
trigger: always_on
description: <!-- All comments in this format should be kept -->
---

# FatFishFairy

<!-- All comments in this format should be kept -->

The goal of this project is to create a desktop fairy with realtime response to user actions:
- The app keeps taking snapshots of all monitors in a reasonable rate.
- A model with stateless session (the vision agent) describe what could be seen from snapshots, another model with memory (the fairy agent) react to the description.
- The fairy tries to remember any interesting stuff about the user.
- Run each vision-fairy round sequentially; the desktop app uses an owned background worker so model requests do not block the UI.

This application requires Windows 10 or later, no need to worry about cross platform stuff. The existing capture implementation uses per-monitor DPI awareness v2; Windows 7 is not supported.

Note that agent expects a OpenAI v1 chat completion protocol, it could use any official or self-hosted LLM that supports it.

## Maintenance

At the end of any coding task, update AGENTS.md and README.md to fix stale fact or add important information, commit and push local changes to the main branch, rebase if conflict.

DO NOT repeat information of actual characters here, including information in `themes/job.updateThemes.prompt.md`, always expect to read files in the `themes` folder when needed.

DO NOT maintain README.md as the Chinese translation of AGENTS.md. Instead it should only has the following topic:
- How to prepare, build and test this repo.
- How to use `FatFishCli` and `FatFishFairy` and what do they do from user's perspective.
- README is not the specification of the software, focus more on what users can do and what users can get, instead of describing too much details.
Maintain README_EN.md as the English translation of README.md and leave the `中文 | English` link in both of them for easy switching.

If the source code is not touched in a request, you are not required to run the verification.
If `Agents` folder is changed, you need to run full verification as well as `FatFishFairy`, otherwise only run verification on test apps that is affected in:
- `UnitTest`
- `FatFishCli`
- `FatFishFairy`

## Supported Tools

- Tools available to all agents
  - HTTP/HTTPS querying, for learning knowledges.
  - File reading.
  - File writing, unexisting folders and the target file will be created recursively.
  - File deleting, folders without any file will be deleted:
    - From the direct folder of the target file to the root folder, if a folder is empty, delete that folder.
    - The `ROOT-REPO/memory` folder should never be deleted.
    - The `ROOT-REPO/Index.md` file should never be deleted.
  - File searching.
  - Speak:
    - For the vision agent: description from snapshots.
    - For the fairy agent: anything want to say to the user.

### Restrictions

- File access should only limit to everything in the`REPO-ROOT/memory` folder.
- Path given to file tools should not contain anything like `.` or `..` that could escape from the folder. Later the path should be first expanded to an absolute path, verify and deny if the target file is not in the memory folder.
- All files will be loaded into a `Dictionary<WString, Ptr<List<WString>>>` data structure:
  - The key is a normalized path to the memory folder.
  - The value is all lines of the file.
  - Reading and searching could be performed without accessing the file system, writing will be submitted to the dictionary and the file system.

### Specification

Maintain tool spec in `REPO-ROOT/env/Tools.md` in this format:
```markdown
## Specification (TOOL-NAME)
Request and response format, behavior, other details
```

## Authorization

`REPO-ROOT/env/apikey.json` has anything needed for the authorization. This file contains sensitive information:
- It cannot be committed to git, always make sure `REPO-ROOT/env/.gitignore` already excludes it.
- All agents should use its own dedicated model.
- Follow OpenAI chat competion protocol but it connects to a non-official server.

## Agents

In `REPO-ROOT/env` these shared prompt files are submitted to agents accordingly, in each request submission:
- `Tools.md`, describe specification of all tools to all agents.
- `Guidance.md`, guidance about how to maintain memories based on the file system, including that `Index.md` should be used to index all other files, offering efficient advices.
- `Request_Vision.md`, fixed request to the vision agent.
- `Request_Fairy.md`, fixed request to the fairy agent.

The fairy's character prompt is stored in `REPO-ROOT/themes/<theme>/Character.md`; `env/Character.md` is no longer used. Read the selected theme's files for its personality, background and design rationale. Any `Character.research.md` is documentation and must not be loaded as a character prompt.

**IMPORTANT**: All prompt files listed here should be in Chinese. Except `Character.md`, all files could be modified during development.

### the Vision

- Requests to the agent should combine these prompt files in `REPO-ROOT/env` in this order:
  - `Tools.md`.
  - `Guidance.md`
  - `Request_Vision.md`
  - Embed snapshots of all monitors.
- Expect very detailed description from the snapshot.
- Every round starts a new session, nothing from the last round is needed.

### The Fairy

- Requests to the agent should combine these prompts in this order:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vczh/FatFishFairy](https://github.com/vczh/FatFishFairy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
