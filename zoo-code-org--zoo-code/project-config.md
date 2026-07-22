---
trigger: always_on
description: E2E tests run against `@copilotkit/aimock` (`LLMock`) — a local HTTP server that replays recorded LLM responses. This makes tests free, deterministic, and CI-friendly.
---

# E2E Test Fixture Workflow

E2E tests run against `@copilotkit/aimock` (`LLMock`) — a local HTTP server that replays recorded LLM responses. This makes tests free, deterministic, and CI-friendly.

Before adding an e2e test, check whether the regression can be proven with a package-local unit or integration test. E2E tests should cover real extension-host boundaries and full workflow smoke checks, not detailed assertions that belong to service, protocol, or UI component tests.

## How aimock matching works

Fixtures are matched by **substring**: `incoming_last_user_message.includes(fixture.match.userMessage)`. A fixture fires if its match string appears _anywhere_ in the last user message of the API request.

**Critical**: the last user message always contains `<environment_details>` with the current time. Never use a match string that includes a timestamp — it will stop matching on the next run.

Record mode uses **record-on-miss**: if an existing fixture already matches a request, aimock serves it and does **not** re-record. Only unmatched requests are proxied to the real API and saved as `openai-*.json` files.

## Adding a fixture for a new test

1. Write the test in `src/suite/`. Use short, stable, unique text in the task prompt.

2. Clear any stale auto-recorded files first (they accumulate across record runs):

    ```sh
    git clean -fx apps/vscode-e2e/fixtures/
    ```

    The `-x` flag is required because `openai-*.json` files are gitignored — `git clean -f` alone silently skips them.

3. Record fixtures. Use an OpenRouter key (default) or an Anthropic key (for tests that use the
   Anthropic provider directly):

    ```sh
    # OpenRouter (default — most tests)
    OPENROUTER_API_KEY=<key> pnpm --filter @roo-code/vscode-e2e test:record

    # Anthropic provider (tests that call api.setConfiguration({ apiProvider: "anthropic" }))
    # OPENROUTER_API_KEY is still required — the harness always initialises with OpenRouter.
    OPENROUTER_API_KEY=<or-key> ANTHROPIC_API_KEY=<key> TEST_FILE=my-anthropic-test.test.js pnpm --filter @roo-code/vscode-e2e test:record
    ```

    To avoid re-recording unrelated tests, filter to just your file:

    ```sh
    OPENROUTER_API_KEY=<key> TEST_FILE=my-feature.test.js pnpm --filter @roo-code/vscode-e2e test:record
    ```

    This proxies unmatched requests to the real API and writes `fixtures/openai-*.json` (OpenRouter)
    or `fixtures/anthropic-*.json` (Anthropic). Background calls from the extension will also be
    recorded — that's expected, ignore them.

4. Find the auto-recorded file for your test:

    ```sh
    grep -l "your unique prompt text" apps/vscode-e2e/fixtures/openai-*.json
    ```

5. Inspect it to find the `response` block (tool calls the LLM made).

6. Create a named fixture file, e.g. `fixtures/my-feature.json`, with a **short stable match string**:

    ```json
    {
    	"fixtures": [
    		{
    			"match": { "userMessage": "your unique prompt text" },
    			"response": {
    				"toolCalls": [
    					{ "name": "attempt_completion", "arguments": "{\"result\":\"...\"}", "id": "call_001" }
    				]
    			}
    		}
    	]
    }
    ```

    The match string should be unique enough to identify this request but contain **no timestamps, file paths, or environment details**.

7. Delete the `openai-*.json` files — they're gitignored and can't be replayed.

8. Verify in mock mode (no API key needed):
    ```sh
    pnpm --filter @roo-code/vscode-e2e test:ci:mock
    ```

## Multi-turn tests

If the LLM calls a tool first (e.g. `read_file`) and then calls `attempt_completion` after seeing the result, you need two fixtures:

- **Turn 1**: match on the task prompt (with `sequenceIndex: 0` so it fires only once) → respond with the tool call, giving the tool call a unique `id`
- **Turn 2**: match on `toolCallId` → respond with `attempt_completion`

Using `toolCallId` (the `id` of the tool call emitted in turn 1) is the recommended approach for turn-2 matching. It is:

- **Precise**: fires only when that exact tool call's result is in the conversation
- **Cross-test safe**: each test's tool call ids are unique, so accumulated match counts from previous tests can't interfere
- **Stateless**: no `sequenceIndex` needed on turn-2 fixtures — if the task makes extra API calls they'll keep getting the same `attempt_completion`

Example:

```json
{
	"fixtures": [
		{
			"match": {
				"userMessage": "my-e2e-tag:my-test",
				"sequenceIndex": 0
			},
			"response": {
				"toolCalls": [{ "name": "read_file", "arguments": "{\"path\":\"marker.txt\"}", "id": "call_my_read" }]
			}
		},
		{
			"match": { "toolCallId": "call_my_read" },
			"response": {
				"toolCalls": [
					{ "name": "attempt_completion", "arguments": "{\"result\":\"MY_MARKER\"}", "id": "call_my_done" }
				]
			}
		}
	]
}
```

The `model` field can be added to either match when a test targets a specific model.

## 404 errors in logs are expected


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zoo-Code-Org/Zoo-Code](https://github.com/Zoo-Code-Org/Zoo-Code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
