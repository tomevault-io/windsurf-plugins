---
trigger: always_on
description: Data privacy comes first, always.
---

Data privacy comes first, always.

Jarvis is offline-first by principle. Do not add integrations that depend on a specific proprietary cloud vendor (e.g. ElevenLabs TTS, or any SaaS that requires shipping user audio/text/data to a third party's servers), not even as an opt-in feature. The line is the offline path: a backend is acceptable only if it can run fully locally. Generic, self-hostable protocols where the user points at their own endpoint (Ollama, OpenAI-compatible servers, LM Studio) are fine precisely because they can be local; a vendor-locked cloud service with no local option is not. When a feature request asks for such a service, decline it on principle and redirect to the offline equivalent (e.g. better local TTS voices rather than cloud TTS).

All user-facing command line output should make use of emojis. Especially an initial emoji to start off the lines that depict what the line is about. Output should make use of indentation spacing to establish a visual hierarchy and aim to make output as easy to sift through as possible. Exception: Windows .bat scripts cannot use emojis (cmd.exe doesn't render Unicode properly).

Any important point in our logical flows should have debug logs using the `debug_log` method from `src/jarvis/debug.py`. Avoid excessive logging to keep the logs easily readable and actionable.

Any code change must either adhere to our spec files perfectly or you should ask the user to confirm changes, which should also propagate to the specs themselves. Spec files follow the \*.spec.md format and live next to the code that implements them. Always search for related spec files before starting any work. When corrected about how something should work, check if there's a spec for it and whether it needs updating.

### Spec File Registry

| Spec file | Covers | Key principles |
|-----------|--------|----------------|
| `src/desktop_app/desktop_app.spec.md` | System tray app, startup flow, daemon integration, windows, theme, updates | Desktop is separate from core; jarvis has no knowledge of desktop_app |
| `src/desktop_app/settings_window.spec.md` | Auto-generated settings UI from config metadata | Metadata-driven; only non-default values written; preserves unknown keys |
| `src/desktop_app/setup_wizard.spec.md` | First-run wizard (Ollama, models, Whisper, location) | Minimal friction; only shown when user action required; doesn't configure everything |
| `src/jarvis/dictation/dictation.spec.md` | Hold-to-dictate engine, hotkey, clipboard paste | Independent from assistant pipeline; shared Whisper model; pause flag on listener |
| `src/jarvis/listening/listening.spec.md` | Voice listener, wake word detection, audio pipeline | — |
| `src/jarvis/reply/reply.spec.md` | LLM reply generation, tool use, profiles | Tools return raw data; profiles handle formatting |
| `src/jarvis/reply/evaluator.spec.md` | **Deprecated** — evaluator no longer runs in the reply engine; preserved for reference | Replaced by the planner; see planner.spec.md |
| `src/jarvis/reply/planner.spec.md` | Task-list planner: pre-loop query decomposition + direct-exec step resolver for small models | Fail-open; rides warm small model chain; advisory for large models, direct-exec for small |
| `src/jarvis/tools/builtin/tool_search.spec.md` | toolSearchTool escape hatch for mid-loop tool routing | Re-runs the same router; never removes stop/self; capped per reply |
| `src/jarvis/tools/external/mcp_runtime.spec.md` | Persistent MCP runtime: per-server long-lived stdio session, queue-based dispatch, retry on transient session loss | One worker per server keyed by config; calls to the same server serialise; `MCPServerSessionError` for session-level failures; opt-in `idle_timeout_sec` for stateless servers |
| `src/jarvis/reply/prompts/prompts.spec.md` | System/user prompt templates | — |
| `src/jarvis/tools/builtin/web_search.spec.md` | webSearch tool: cascade fetch, SSRF guard, prompt-injection fence, links-only envelope | Untrusted web content is fenced as data, not instructions; rank preference over speed; honest failure over confabulation |
| `src/jarvis/tools/builtin/nutrition/log_meal.spec.md` | logMeal tool: single-property schema for planner fast-path, internal nutrition extraction, untrusted-data fence, follow-ups | Public schema is a single optional `meal` string; nutrition fields are internal; user text is fenced as data |
| `src/jarvis/utils/location.spec.md` | GeoIP location detection | Privacy-first; local GeoLite2 DB only |
| `src/jarvis/memory/graph.spec.md` | Node graph memory (v2), self-organising tree, UI explorer | Dynamic structure; access-aware; auto-split/merge (future) |
| `src/jarvis/memory/summariser.spec.md` | Diary summariser prompt contract, hygiene rules (deflection, attribution, topic separation), post-process scrub, and bulk-sweep clean button | Two-layer defence: prompt + deterministic scrub; corrupted summaries poison every downstream consumer |
| `src/jarvis/memory/recall_gate.spec.md` | Deterministic skip-enrichment heuristic when the hot window covers a follow-up | Fail-open; language-agnostic via `\w{3,}` + `re.UNICODE`; planner intent always wins |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tiwarisundar/jarvis-main](https://github.com/Tiwarisundar/jarvis-main) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
