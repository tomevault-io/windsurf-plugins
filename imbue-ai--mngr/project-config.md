---
trigger: always_on
description: IT IS CRITICAL TO FOLLOW ALL INSTRUCTIONS IN THIS FILE DURING YOUR WORK ON THIS PROJECT.
---

IT IS CRITICAL TO FOLLOW ALL INSTRUCTIONS IN THIS FILE DURING YOUR WORK ON THIS PROJECT.

IF YOU FAIL TO FOLLOW ONE, YOU MUST EXPLICITLY CALL THAT OUT IN YOUR RESPONSE.

# Important things to know:

- This is a monorepo.
- ALWAYS run commands by calling "uv run" from the root of the git checkout (ex: "uv run mngr create ..."). Do NOT call "mngr" directly (it will refer to the wrong version).
- NEVER amend commits or rebase--always create new commits.
- If you ever need to work with another *git* repo that is *outside* of this monorepo, you should do so by creating a worktree for that repo and putting it within a ".external_worktrees/" directory *within* this root folder (ie, at the same level as the ".git/" folder).  Use the same branch name in the worktree as the branch you are working on in this repo, and be sure to commit inside there as well.

# How to get started on any task:

Always begin your session by reading the relevant READMEs and any other related documentation in the docs/ directory of the project(s) you are working on. These represent *user-facing* documentation and are the most important to understand.

Once you've read these once during a session, there's no need to re-read them unless explicitly instructed to do so.

If you will be writing code, be sure to read the style_guide.md for the project. Then read all README.md files in the relevant project directories, as well as all `.py` files at the root of the project you are working on (ex: `primitives.py`, etc.). Also read everything in data_types, interfaces, and utils to ensure you understand the core abstractions.

Then take a look at the other code directories, and based on the task, determine which files are most relevant to read in depth. Be sure to read the full contents from those files.

Do NOT read files that end with "_test.py" during this first pass as they contain unit tests (unless you are explicitly instructed to read the unit tests).

Do NOT read files that start with "test_" either, as they contain integration, acceptance, and release tests (again, unless you are explicitly instructed to read the existing tests).

Only after doing all of the above should you begin writing code.

# Important commands and conventions:

- Never run `uv sync`, always run `uv sync --all-packages` instead
- Never pipe the output of a non-instant command through `grep`, `jq`, `awk`, `head`, `tail`, or similar without preserving the full output. If you later need something that was discarded, you have to rerun the entire command. Instead, write the output to a file first and then filter the file, or use `tee` to capture and filter simultaneously (e.g. `some_command | tee /tmp/some_command_output.txt | grep pattern`).

# Always remember these guidelines:

- Never misrepresent your progress. It is far better to say "I made some progress but didn't finish" than to say "I finished" when you did not.
- Always finish your response by reflecting on your work and identify any potential issues.
- If I ask for something that seems misguided, flag that immediately. Then attempt to do whatever makes the most sense given the request, and in your final reflection, be sure to flag that you had to diverge from the request and explain why.
- During your final reflection, if you see a potentially better way to do something (e.g. by using an existing library or reusing existing code), flag that as a potential task for future improvement.
- Never use emojis. Remove any emojis you see in the code or docs whenever you are modifying that code or those docs.
- Be concise in your communications. Don't hype up your results, say "perfect!", or use emojis. Be serious and professional.

# When coding, follow these guidelines:

- Only make the changes that are necessary for the current task.
- Before implementing something, check if there is something in the codebase or look for a library
- Reuse code and use external dependencies heavily. Before implementing something, make sure that it doesn't already exist in the codebase, and consider if there's a library that can be imported instead of implementing it yourself. We want to be able to maintain the minimum amount of code that gets the job done, even if that means introducing dependencies. If you don't know of a library but think one might be plausible, search the web. (I'm even open to using random GitHub projects, but run anything that's not a well-established library by me first so I can check if it's likely to be reliable.)
- Code quality is extremely important. Do not compromise on quality to deliver a result--if you don't know a good way to do something, ask.
- Follow the style guide!
- Use the power of the type system to constrain your code and provide some assurance of correctness. If some required property can't be guaranteed by the type system, it should be runtime checked (i.e. explode if it fails).
- Avoid using the `TYPE_CHECKING` guard. Do not add it to files that do not already contain it, and never put imports inside of it yourself--you MUST ask for explicit permission to do this (it's generally a sign of bad architecture that should be fixed some other way).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imbue-ai/mngr](https://github.com/imbue-ai/mngr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
