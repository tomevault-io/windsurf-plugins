---
trigger: always_on
description: Omamail reads the default AI selected in Omarchy. There is no Omamail AI settings
---

# AI beside your mail

Omamail reads the default AI selected in Omarchy. There is no Omamail AI settings
page or separate Agent page. The background adapter currently supports Claude;
other defaults produce an inline explanation without opening a terminal or picker.
The installed Claude CLI uses its normal system login and provider configuration.

Choose the outline **AI icon** button beside Compose in the window header, the
message menu, or `Alt+G` in the list, reader or composer. The right dock displays
the conversation, aligned to the bottom with older turns above. Drag its left
edge to resize; double-click the divider to restore the default width. User messages
have a background and a › marker; AI replies have no background and each offers
a copy icon after generation finishes that copies the original reply. Bold and code are formatted
through an escaping formatter that cannot create links or remote resources. Execution status appears just above the input, which starts at one line and
grows with newlines.
Type `/` to show commands, then use Up/Down and Return or click a suggestion.
Selecting a command only fills editable instructions. **Enter** sends;
**Shift+Enter** inserts a newline. Ctrl+Enter also sends. While running, the
stop icon ends the request. The **…** menu contains **New chat** and **History...**
for the current mail or draft. The header AI button closes the dock without stopping an active request.
While running, a timed Working line stays above the input and Escape interrupts
the request; when idle, Escape closes the dock.

While AI is working, Enter adds another message to a Pending queue and clears
the input immediately. Messages run in order in the same conversation after
each successful reply. Click a pending message to bring it back into an empty
input for editing, or remove it with ×. A failed start retains the message;
interrupting or a failed reply pauses the queue. Pending messages are held only
for this application session, with up to 20 messages and bounded text size.
They never switch to another mail or conversation. The queue for a request still
preparing its first turn waits until that conversation can be identified.

The worker runs silently in the background. Text appears progressively, along
with public status events such as reading a file or finishing a tool. Raw tool
arguments/results, diagnostics, and internal reasoning are not displayed.
Completed requests can be followed up in the same native Claude conversation.
The **New chat** action reads the current mail or draft into a fresh conversation.
Follow-ups keep the original context; a notice identifies a draft edited since
that context was captured.

A request can cover at most 20 messages from one mailbox. The owning provider's
normal read interface supplies complete bodies without selecting or marking mail
read. Results stay bound to their account, messages and draft identity. You can
select history text or copy each answer. Translation and rewriting commands act
only on mail titles and bodies, excluding addresses and metadata. Their results
separate Title and Body; draft insertion takes only the Body section, so a
translated title is not accidentally inserted into the body. In a draft, **Insert at cursor**
and **Replace body** apply only a completed successful reply; neither sends mail.
Replacement is two text edits and can require two undo steps. The mail list has no AI icon. The header AI button stays static without a
breathing animation.

## Suggested events

With **Suggest calendar events from mail** on in Settings — off until it is — a
message opened in the reader whose text mentions a date or a time is handed to
the system AI once, in the background, with rules that ask for a JSON array of
the events it finds and nothing else. The reader draws a card per event above
the message: what, when, where, with **Add** and **Dismiss**. Add opens the
calendar's event composer with the fields filled in, so the owner chooses the
calendar and looks the times over before anything is written; a written event
waves its suggestion away, a dismissed one stays away for the session.

The gates are local and cheap, and they come before the model, because a
look is a model call with the whole message in it: the setting; a message
from a person rather than a machine or a list — `Agent.automatedMail` reads
the sender (`noreply`, `notifications@`, `mailer-daemon`, a newsletter or
alerts address) and Gmail's Promotions, Updates, Forums and Social
categories off the row, and the reader knows a list by its List-Unsubscribe
header; a time or a date in the subject or the text (`Agent.mentionsDate`: a
clock time, a month with a day, a numeric date, "tomorrow", or a weekday
bound to a plan like "on Thursday" — a bare "Sunday" in prose is a word);
a message from the last two months (one with no known date is not looked
at); no look at that message yet; and at most two looks running at once — a
look that cannot start yet waits its turn. The worker sends at most the
first 8,000 characters of the message. A
look is a job started through the same account-bound `agent.context` read as an
ask, with `events: true` on the payload; Rust records it with kind `events`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huacnlee/omamail](https://github.com/huacnlee/omamail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
