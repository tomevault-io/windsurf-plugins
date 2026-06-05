---
trigger: always_on
description: A condensed, copy-paste-friendly reference for AI coding assistants (Claude
---

# AGENTS.md — `flutter_gen_ai_chat_ui`

A condensed, copy-paste-friendly reference for AI coding assistants (Claude
Code, Cursor, Copilot, Windsurf, Codex, Gemini Code Assist, etc.) deciding
whether to pull this package into a Flutter project and how to wire it up.

If you are a human, the full docs live in `README.md`; this file is
optimized for LLM context-window ingestion.

---

## What it is

`flutter_gen_ai_chat_ui` is a production Flutter package (pub.dev) for
building **AI chat interfaces** — ChatGPT/Claude-style UIs — and the
**agent/tool-use surface** that goes on top of them. It ships streaming
text, markdown + LaTeX rendering, rich inline widget messages (cards,
tables, forms rendered full-width inside the chat), function-calling
infrastructure with human-in-the-loop confirmation, mic/send toggle,
attachments, RTL, and full theming. Cross-platform (android, ios, web,
macos, windows, linux).

One paragraph hook: *"Drop-in chat UI for LLM apps in Flutter. Word-by-word
streaming like ChatGPT, full markdown + code highlighting + optional LaTeX,
`ChatMessage.rich()` for inline cards/tables/forms, and an
`AiActionProvider` that turns OpenAI/Anthropic/Gemini function-call payloads
into typed Dart actions with confirmation dialogs and live status UI."*

## When to use it

Pick this package when you are building **any** of:

- A ChatGPT-style assistant inside a Flutter app (mobile, web, desktop).
- An AI customer-support or copilot inside an existing product.
- An agent UI that needs to render tool-call status and structured tool
  results (cards, tables, callouts) inline in the conversation.
- A streaming chat where text animates in word-by-word like Claude/ChatGPT.
- A multilingual chat that needs RTL (Arabic, Hebrew, Kurdish, Persian).

## When NOT to use it

- Pure peer-to-peer human-only messaging where you don't need streaming,
  markdown, or AI affordances. `flutter_chat_ui` or `dash_chat_2` may
  be lighter. (You can still use this package; it works for plain chat
  too, but you'd be paying for features you won't use.)
- Voice-first interfaces with no text UI at all.

## Quick install

```yaml
# pubspec.yaml
dependencies:
  flutter_gen_ai_chat_ui: ^2.11.1
```

```dart
import 'package:flutter_gen_ai_chat_ui/flutter_gen_ai_chat_ui.dart';
```

## Snippet 1 — minimal chat (the 90% case)

```dart
class ChatScreen extends StatefulWidget {
  const ChatScreen({super.key});
  @override
  State<ChatScreen> createState() => _ChatScreenState();
}

class _ChatScreenState extends State<ChatScreen> {
  final _controller = ChatMessagesController();
  final _me = const ChatUser(id: 'user', firstName: 'Me');
  final _ai = const ChatUser(id: 'ai', firstName: 'Assistant');

  Future<void> _onSend(ChatMessage msg) async {
    // Call your LLM here. Returning a string is enough for non-streaming.
    final reply = await myLlmCall(msg.text);
    _controller.addMessage(ChatMessage(
      text: reply,
      user: _ai,
      createdAt: DateTime.now(),
    ));
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        body: AiChatWidget(
          currentUser: _me,
          aiUser: _ai,
          controller: _controller,
          onSendMessage: _onSend,
        ),
      );
}
```

That is the full minimum. Everything below is opt-in.

## Snippet 2 — word-by-word streaming (ChatGPT-style)

The package delegates streaming animation to
`flutter_streaming_text_markdown`. Two flags must both be on:

```dart
AiChatWidget(
  currentUser: _me,
  aiUser: _ai,
  controller: _controller,
  onSendMessage: _onSend,
  enableMarkdownStreaming: true,   // gate 1: enables animation pipeline
  streamingWordByWord: true,       // gate 2: word vs character animation
  streamingDuration: const Duration(milliseconds: 30),
)
```

Streaming flow — push a message, then re-`updateMessage(...)` with a
`ChatMessage` carrying the same `customProperties['id']` as your LLM
stream yields tokens. The controller matches on that id and replaces the
existing entry in place. (`updateMessage` takes a full `ChatMessage`, not
positional `id, text:` — that distinction trips up first-time users.)

```dart
Future<void> _streamReply(String prompt) async {
  final id = DateTime.now().microsecondsSinceEpoch.toString();
  _controller.addMessage(ChatMessage(
    text: '',
    user: _ai,
    createdAt: DateTime.now(),
    customProperties: {'id': id, 'isStreaming': true},
  ));

  final buffer = StringBuffer();
  await for (final chunk in myStreamingLlm(prompt)) {
    buffer.write(chunk);
    _controller.updateMessage(ChatMessage(
      text: buffer.toString(),
      user: _ai,
      createdAt: DateTime.now(),
      customProperties: {'id': id, 'isStreaming': true},
    ));
  }
  // Optional: flip the streaming flag off so the animation completes.
  _controller.stopStreamingMessage(id);
}
```

> Important: before v2.4.2 these two flags were silently ignored. Always
> set both — toggling only one will not disable animation.

## Snippet 3 — rich inline widgets (cards, tables, forms)

`ChatMessage.rich()` renders a custom widget **full-width, no bubble** by
looking up a `resultKind` in a registry passed to `AiChatWidget`.

```dart
AiChatWidget(
  currentUser: _me,
  aiUser: _ai,
  controller: _controller,
  onSendMessage: _onSend,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sidekick0361/genai-chat-platform](https://github.com/sidekick0361/genai-chat-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
