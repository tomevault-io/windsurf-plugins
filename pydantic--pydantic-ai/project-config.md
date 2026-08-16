---
trigger: always_on
description: [`GoogleRealtimeModel`][pydantic_ai.realtime.google.GoogleRealtimeModel] connects an agent to Gemini
---

# Google Gemini Live

[`GoogleRealtimeModel`][pydantic_ai.realtime.google.GoogleRealtimeModel] connects an agent to Gemini
Live, including native audio, live images, and provider-native tools. Start with the
[realtime quickstart](overview.md#quickstart) or [camera example](../examples/realtime-camera.md).

## Setup

To use Gemini Live models, install `pydantic-ai-slim` with the `google-realtime` optional group,
which bundles the `google-genai` SDK together with the realtime transport dependencies:

```bash
pip/uv-add "pydantic-ai-slim[google-realtime]"
```

Authentication comes from `provider`, mirroring
[`GoogleModel`][pydantic_ai.models.google.GoogleModel]. Use `provider='google'` for the Gemini
Developer API or `provider='google-cloud'` for Vertex AI/ADC, with API keys and credentials
configured as described in the [Google model documentation](../models/google.md#configuration).
Pass a [`GoogleProvider`][pydantic_ai.providers.google.GoogleProvider] or
[`GoogleCloudProvider`][pydantic_ai.providers.google_cloud.GoogleCloudProvider] for custom
credentials, project, region, or client.

## Model names

Use a Gemini Live model ID, for example `gemini-2.5-flash-native-audio-latest` or
`gemini-3.1-flash-live-preview`. Native-audio and other Live models differ in thinking,
asynchronous tools, and output behavior. Use the
[official Gemini Live documentation](https://ai.google.dev/gemini-api/docs/live) as the canonical
model and availability source.

## Settings

[`GoogleRealtimeModelSettings`][pydantic_ai.realtime.google.GoogleRealtimeModelSettings] — the
realtime counterpart of [model run settings](../agent.md#model-run-settings) — extends the
[shared settings](overview.md#shared-settings) with Google generation and Live controls:

```python
from pydantic_ai.realtime.google import GoogleRealtimeModel, GoogleRealtimeModelSettings

settings = GoogleRealtimeModelSettings(
    temperature=0.7,
    top_p=0.9,
    google_voice='Puck',
    google_language_code='en-US',
    google_affective_dialog=True,
    google_proactive_audio=True,
    google_vad={'start_sensitivity': 'high', 'end_sensitivity': 'low'},
    google_turn_coverage='all_video',
    google_context_compression={'trigger_tokens': 16000, 'target_tokens': 8000},
)
model = GoogleRealtimeModel('gemini-2.5-flash-native-audio-latest', settings=settings)
```

| Setting | Purpose |
| --- | --- |
| `google_voice`, `google_language_code`, `google_multi_speaker` | Voice, output language, and per-speaker voices |
| `google_affective_dialog`, `google_proactive_audio` | Emotion-aware delivery and model-decided speech on native-audio models |
| `google_vad` | Exact automatic VAD; fully overrides shared [`turn_detection`](turns.md#automatic-turn-detection) |
| `google_activity_handling`, `google_turn_coverage` | [Interruption](turns.md#barge-in) behavior and which input belongs to a turn |
| `google_input_transcription`, `google_output_transcription` | Native [transcription](audio.md#input-transcription) switches, enabled by default |
| `google_context_compression` | Sliding-window compression for long sessions |
| `google_enable_session_resumption` | Native state restoration; enabled automatically by a `reconnect` policy |
| `google_async_tool_calls` | Lets supported native-audio models continue speaking during tools |
| `google_config_overrides` | Raw `LiveConnectConfig` keys merged last as a forward-compatibility escape hatch |

`google_voice` is the provider voice setting. `google_thinking_config` takes precedence over the
shared [`thinking`](../capabilities/thinking.md) setting when a token budget or other
Gemini-specific control is needed.

!!! warning "Keep automatic VAD enabled"
    Pydantic AI does not expose Gemini activity markers or manual turn verbs. Do not set
    `google_vad={'disabled': True}`; shared `turn_detection=False` is rejected for the same reason.

### Asynchronous tool calls

Gemini normally pauses generation while a function tool is outstanding. Set
`google_async_tool_calls=True` on supported native-audio models to let it continue speaking. This is
best for slow tools; a fast result can interrupt speech that barely started and leave an empty
interrupted turn in history. Other Live models ignore the setting.

### Native tools

Gemini Live maps [`WebSearch`][pydantic_ai.capabilities.WebSearch] to Google Search grounding, the
only native tool it supports — no Live model runs native code execution or URL context, so neither
[`CodeExecutionTool`][pydantic_ai.native_tools.CodeExecutionTool] nor
[`WebFetch`][pydantic_ai.capabilities.WebFetch] is advertised in `supported_native_tools`. Give
those a [`local=` fallback](tools.md#native-tools) and the session runs the local tool instead:
`CodeExecutionTool(local=...)`, or `WebFetch(native=False, local=True)`, which requires the
`web-fetch` optional group (`pip/uv-add "pydantic-ai-slim[google-realtime,web-fetch]"`).

Gemini 2.5 also cannot combine native Google Search grounding with function tools; choose native
grounding or local function-tool fallbacks unless using a model that supports the combination.

### Specialist streaming models

The built-in profile describes the speech-to-speech Live models. Gemini also serves specialist

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pydantic/pydantic-ai](https://github.com/pydantic/pydantic-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
