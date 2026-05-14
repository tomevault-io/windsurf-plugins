---
trigger: always_on
description: - mini_agent.py -- main entrypoint for agentic loop
---

# Mini_agent

## File structure and development

Files:
- mini_agent.py -- main entrypoint for agentic loop
- core_tools.py -- MCP server with tools, hooks and system prompts
- typedefs.py -- type definitions
- utils.py -- common helpers used by agentic loop
- test/*.py -- various unit tests
- test/sample_data -- directory with immutable sample data

Codebase:
- uses litellm library, to make calls to different LLMs in a uniform way
- uses watchdog, to be able to notify about file changes
- uses mcp, for some common tool definitions
- uses pyright, for typechecking in strict mode
- uses pytest, for testing

Running and testing:
- Set up venv and install dependencies:
```
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```
- On subsequent use, just `source venv/bin/activate`.
- Run the agentic loop:
```
OPENAI_API_KEY=redacted    ./mini_agent.py --model gpt-4.1
GEMINI_API_KEY=redacted    ./mini_agent.py --model gemini/gemini-2.5-pro
ANTHROPIC_API_KEY=redacted ./mini_agent.py --model anthropic/claude-sonnet-4-20250514
```
- Test the mcp server:
   - `npx @modelcontextprotocol/inspector --cli ./core_tools.py --method tools/list`
   - `npx @modelcontextprotocol/inspector --cli ./core_tools.py --method resources/templates/list`
- Unit tests: for example `python -m pytest test/test_edit.py`



## Codebase style and guidelines

All code MUST be written with a high degree of rigor:
- All functions are documented to say what they do, what side effects they have in any
- All state variables MUST be documented with INVARIANTS.
  Every function's comments MUST explain which invariants the function is assuming,
  and which ones it establishes/upholds, and how,
- When we do code review, we always review by checking it against invariants.
- When we write code, we add comments to explain whenever code relies upon a documented
  assumption, or ensures a documented guarantee.

IMPORTANT: The AI agent MUST ALWAYS evaluate code with skepticism and rigor.

- IMPORTANT: The AI agent MUST ALWAYS look for flaws, bugs, loopholes,
  in what the user writes and what the AI agent writes.
- IMPORTANT: instead of saying "that's right" to a user prompt, the AI agent
  must instead think to find flaws, loopholes, problems,
  and question what assumptions went into a question or solution.
- A good way to find flaws is to think through the code line by line with a
  worked example.

Example: instead of saying "That's right!" say "This approach seems to
avoid {problem-we-identified}, but has a flaw is that it is non-idiomatic,
and it hasn't considered the edge case of {counter-example}."

Coding style: All code must also be clean, documented and minimal. That means:
- If a helper is only called by a single callsite, then prefer to inline it into
  the caller
- If some code looks heavyweight, perhaps with lots of conditionals, then think
  harder for a more elegant way of achieving it.
- Code should have comments, and functions should have docstrings.
  The best comments are ones that introduce invariants, or prove that invariants
  are being upheld, or indicate which invariants the code relies upon.
- Prefer functional-style code, where variables are immutable "const" and there's
  less branching. Prefer to use ternary expressions "x if b else y" rather than
  separate lines and assignments, if doing so allows for immutable variables.

## Testing conventions

IMPORTANT: Tests must NEVER modify or delete files in the working directory.
- The test/sample_data directory contains permanent test fixtures that must NEVER be deleted
- Tests must NEVER alter the contents of any file in the working directory
- Tests can ONLY create temporary files in the system tmp directory (e.g., using Python's tempfile module)
- Any test that needs to create files should use tempfile.mkdtemp() or similar
- Any test cleanup should only remove files that the test itself created in tmp directories

Here is an example of rigorous quality code. It covers all the signs of rigor:
- was the code as simple as could be?
- does the code work all edge cases, and does it include proof/evidence
  that we identified all possible edge cases and that it handled them all?
- were the classes and data-structures the right fit?
- was logic abstracted out into functions at the right time, not too much,
  not too little?
- for mutable state, were the correct invariants identified, established,
  maintained, proved
- did we correctly identify the big-O issues and come up with suitable solutions?
- was async concurrency and re-entrancy handled correctly

```
/**
 * This function is like fetch(), but it deals with OAuth2 code-flow authentication:
 * - It sends a header "Authorization: Bearer <access_token>" using the access_token in localStorage
 * - If this fails with 401 Unauthorized, it attempts to refresh the access token and try again.
 * - If someone else is busy doing a refresh, it waits for the refresh to finish before trying.
 *
 * <quality>Here we discuss invariants, including the mutable state 'access_token'</quality>
 * INVARIANT: even with are multiple async callers, only one will attempt to refresh the access token at a time
 * INVARIANT: if refreshing fails, then access_token and refresh_token will be cleared, and all underway and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ljw1004/mini_agent](https://github.com/ljw1004/mini_agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
