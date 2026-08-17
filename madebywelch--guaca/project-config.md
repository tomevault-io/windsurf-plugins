---
trigger: always_on
description: Local desktop app. You talk to LLM agents; the agents talk to each other. Tauri
---

# Guaca

Local desktop app. You talk to LLM agents; the agents talk to each other. Tauri
v2, React + TypeScript front, Rust back.

## Non-obvious things worth knowing before you change something

**The agent runtime is Rust, not the webview.** Each agent is a `tokio` task
with an `mpsc` inbox. If you find yourself adding agent logic to `src/`, you are
in the wrong half of the repo. The frontend renders state and forwards intent.

**Read `runtime/guard.rs` before touching anything about messaging.** Agents
messaging each other does not terminate on its own. Five independent limits stop
it, and each catches a different shape of runaway. Weakening one is not a local
change.

**`expects_reply` is what makes cascades converge**, not the guard. The guard is
the backstop. See `docs/ARCHITECTURE.md`.

**A courtesy to a peer that has already answered is refused; work is not.**
`send_message` carries an `intent`, and the sender declares it, because a second
instruction and a thank-you are the same shape on the wire and guessing from the
shape refused real work. Anything not declared `work`, including a value nobody
defined, is a courtesy: the permissive half of the parser must not be the half
that opens the door. `runtime/prompt.rs` says the same thing to the model in the
mode where it matters, and the two have to agree.

**`expects_reply` and `intent` answer different questions, and conflating them
stopped an agent mid-task.** The first is whether anybody is waiting on your
words, which is what terminates a cascade. The second is whether you were given
something to do. An instruction to a peer that has already answered carries work
and expects no reply, and reading the first as the second put that turn in the
mode that says nothing is being asked of it and silence is usually right. A real
send to the operator's own address died there: the agent spent a call, said
nothing, and looked like it had stopped. `ReplyMode::Assigned` is that
combination, and its output is still a note.

**Whether a send is "answering" is a question about the run, not the batch.**
Replies land milliseconds apart and an actor drains whatever is in its inbox, so
a batch is a timing artifact: three peers answering at once can be split across
turns, and deciding from the batch made two of them look like strangers. Ask the
guard, which counts sends per pair for the whole run.

**An agent that needs the operator's authority asks for it rather than
refusing.** A peer saying "the operator authorised this" is a claim, and
declining it is correct; what an agent lacked was any way to turn that claim
into an answer, so it told the operator to repeat an instruction they had
already given, somewhere else. `request_permission` parks the turn and puts two
buttons in the channel they are already reading. `ProtectedAction::ActOnBehalf`
deliberately has no "always allow" in the UI: the grant is scoped to an agent
and an action, and this action is "act outside the workspace", so a standing yes
would cover every future send and purchase rather than the one being asked
about.

**A protected action parks the turn that asked for it, and the row is the
verdict.** `create_agent` stops mid-turn and waits on a person. The operator's
click and the turn's own timeout can land in the same instant, so the answer is
read back from the `approvals` row rather than from the channel the wakeup
arrived on: `settle_approval` only moves a row out of `pending`, and whichever
of the two loses that race changes nothing. Anything still pending at startup is
expired, because nothing holds a parked turn across a restart. "Always allow" is
the decision row itself, scoped to the one agent that asked. See
`docs/ARCHITECTURE.md`.

**Migrations are forward-only and numbered.** One has already run against a real
database by the time you think of an improvement, and editing it leaves that
database at the same `user_version` with a different schema. Add another.

**Budget counts model calls, not agent turns.** One turn can make several calls
working through tool results. Counting turns lets a bounded run bill many times
over. There is a test named after this.

**A run settles when nothing is outstanding, and an envelope is what is
outstanding.** `deliver` books one against the run as it queues an envelope,
and the turn that reads it releases it. Any new path that takes an envelope and
does not turn it into a turn has to release it too: an agent deleted while
holding queued work used to take the booking with it, and that run never ended.
Nothing else decrements.

**A file's bytes never travel in an envelope, and never cross IPC.** A message
carries a `Part::File` naming the digest; the bytes sit once in `files.rs`,
addressed by content, and a drop hands Rust the *path* rather than the file.
Both follow from the same fact: a transcript is read in bulk, forty messages
into every prompt and hundreds into the activity view. What a model gets depends
on what the file is: a picture is shown, text is read out, and anything else is
written to `~/inbox` on the agent's own machine, because a Linux box knows more
file formats than this runtime ever will. When placing fails the model is told
so in words, since an agent that hears nothing describes a document it never
read.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebywelch/guaca](https://github.com/madebywelch/guaca) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
