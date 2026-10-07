---
trigger: always_on
description: atelier talks to agents through one model of its own. The app and the UI see events, commands and
---

# Agents

atelier talks to agents through one model of its own. The app and the UI see events, commands and
capabilities. They never see an agent's wire format, its tool names or its process. Code lives in
`crates/agents`: `session` is the model, `claude_code` and `acp` are the backends that run an agent's
process, `own` is atelier's own agent, and `subprocess` is a helper for backends that run a child process.
`claude` and `cursor` hold what is particular to each agent: its launch, models and look.

Nothing outside `crates/agents` names an agent, a lab or a forge.

## The model

A session pushes **events** into a sink and takes **commands**. Both are plain data.

| Event | Says |
| --- | --- |
| `Started` | The session id, the model and the permission mode in force. |
| `UserMessage` | A user message atelier did not send: history, or another client. |
| `Text`, `Thinking` | Streamed deltas of a block. `ThinkingDone` carries the time it took. |
| `ToolStarted`, `ToolInput`, `ToolStatus`, `ToolFinished` | One tool call: its name, its `ToolKind`, its input, its file, its output. |
| `ToolEdit` | The text an edit or a write changes, as a `FileEdit` (path, old text, new text): told again as it grows while the agent writes the call. |
| `SubagentStarted`, `SubagentProgress`, `SubagentEnded` | A subagent: its task, kind and model. Its calls carry it as `parent`. |
| `Todos` | The whole todo list, each time it changes. |
| `Permission`, `PermissionCancelled` | A question with its tool call and its choices as data. |
| `Usage` | Tokens and cost of a turn. |
| `TurnEnded` | `Completed`, `Interrupted` or `Failed`, with the closing text. |
| `Warning` | A problem that does not end the session, such as a line that does not parse. |
| `Ended` | The session is over: closed by atelier, the process exited, or it failed. Nothing follows. |

Commands: `Send` (a text and its attachments), `Answer` (a request and one of its choices), `Interrupt`,
`SetModel`, `SetPermissionMode`.

An `Attachment` is a comment on lines of a file with the lines quoted, or a file. A backend that takes only
text writes them out with `message_text`. `crates/review` makes the comments (`docs/review.md`).

`ToolTarget { id, file }` names the file a call will touch as soon as the stream says so, before the call's
input is whole and before the tool runs. Claude Code sends the input as `input_json_delta` chunks; the
mapper reads the file out of the first top-level `file_path`, `notebook_path` or `path` key whose value has
closed. A review takes the file's text before the edit lands from this event.

`ToolEdit { id, edit }` is the one way the UI learns what an edit changes. Each backend fills the `FileEdit` from
its own words, and the UI reads only it and the call's `ToolKind`, never a tool's name or its input's keys:

| Backend | Where the edit comes from | Streams as it is written |
| --- | --- | --- |
| Claude Code | `Edit` (`file_path`, `old_string`, `new_string`) and `Write` (`content`), read from the `input_json_delta` chunks once the file has closed, and again from the whole input. `MultiEdit` has none. | Yes |
| Own agent | Each tool's `Tool::edit` (`edit`, `write`), from the Anthropic `input_json_delta` chunks or the OpenAI argument pieces once the path has closed, and again from the whole input. | Yes |
| ACP | The first `diff` in a call's content. A new file (no old text, or Cursor's `-- /dev/null`) has no old text. | No: Cursor sends the path while the call runs and the diff only when it is done (checked on 2026.10.01-14929f9). |

`crate::partial_json` reads the string fields of an input that is not whole yet, for every backend that gets it in
pieces. When `ToolEdit`s of one call come back to back in one frame, only the newest is kept.

A tool call names its tool as data. `ToolKind` (read, edit, write, search, shell, fetch, other) lets the
UI pick a look without knowing the agent's names. A permission request carries its choices, each with a
`ChoiceKind` (allow, allow always, deny). The UI draws whatever the agent offers.

`Capabilities` says what a backend supports: resume, interrupt, the models and permission modes a
session can switch to, thinking, subagents and todos. The UI hides what a backend lacks.

### The seam

```rust
trait Backend: Send + Sync {
    fn name(&self) -> &str;
    fn capabilities(&self) -> Capabilities;
    fn open(&self, project: Arc<dyn Project>, request: OpenRequest, sink: EventSink)
        -> Result<Box<dyn Session>, SessionError>;
    fn sessions(&self, project: &dyn Project) -> Result<Vec<SessionSummary>, SessionError>;   // optional
    fn history(&self, project: &dyn Project, session: &SessionId) -> Result<Vec<Event>, SessionError>; // optional
}
trait Session: Send {
    fn send(&self, command: Command) -> Result<(), SessionError>;   // never waits for the agent
}
```

The trait does not say "process", "stdio" or "JSON". `EventSink` is a closure, like the project's
`ChangeSink`, so a backend calls it from whatever thread it has. Dropping a session stops the agent.
`open` does not block on the agent: `Started` says when it is ready.

Nothing blocks the UI thread. The app calls `open`, `sessions` and `history` from a background task.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flazouh/atelier](https://github.com/flazouh/atelier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
