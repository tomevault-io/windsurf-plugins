---
trigger: always_on
description: Follow the root [AGENTS.md](../../AGENTS.md). These instructions govern agents
---

# Director agent workflow

Follow the root [AGENTS.md](../../AGENTS.md). These instructions govern agents
operating a Studio iteration flow as well as agents implementing Studio.
Use the running service manifest and current source for exact tool signatures;
do not invent a successful tool response or bypass an unavailable operation.

## Delegation context

Codex / Astra manages the work, designs the system and interactions, delegates
implementation, and reviews the integrated result. Fable handles much of the
delegated implementation, in particular the difficult design and code, and can
also provide independent design and code reviews. Grok takes bounded mechanical
work and validation. Preserve this context in new and resumed lanes, and in
every delegated child, unless the user updates it. Use explicit task ownership
and acceptance criteria when delegating.

Studio exposes separate Fable and Codex lane launchers. Keep each lane's actual
provider and conversation resume ID with its history. Before stopping or
archiving a lane, save and verify that identity; do not silently replace a
conversation with a new chat. Restore by reattaching a still-running terminal
or resuming the saved conversation. Ordinary Studio shutdown only detaches.

## Agent tree and recursive delegation

- Every lane is an agent node keyed by its stable terminal origin. The
  synthetic Director root holds the independent root lanes; each agent may
  delegate children, and children may delegate again. A history split changes
  an agent's current flow, never its node or its children. Deleting a parent
  keeps a tombstone record while a child still references it; children are
  never stopped, moved or reparented implicitly.
- Delegate visible work only through the scoped operation: `flow_agent` over
  `POST /call`, or `director-flow agent '{"action":"start","provider":"codex",
  "title":"...","task":"...","repo":"/abs/worktree"}'`. `provider` is one of
  `claude`, `codex` or `grok`. The parent comes from the caller's capability. Director records the child
  flow, its parent edge and the task as the child's first requirement, creates
  the child's control directory, publishes its callback binding, and only then
  starts the installed provider in the repository `makepad-agents` helper. The
  child has its own terminal, `director-flow` callback, conversation resume
  identity and lane; it appears under its parent without stealing tab
  selection or focus.
- The call id is the durable launch request, retained with the payload's
  signature under the parent agent. The same id with the same payload
  reports the existing child (`already_started`); the same id with a
  different provider, task, repo or context is refused; a deleted child never
  restarts under an old id. Requests never expire: a lane that has used all 64
  retained launch requests is refused new ones, whatever was deleted, split or
  cleared since, so delegate from another lane. After an uncertain reply, run
  `agent '{"action":"list"}'` before retrying.
- `status.launch` is observed evidence from the session owner, not intent:
  `pending` (binding or terminal not yet observed), `starting`, `running`,
  `ended` or `failed` with the terminal's error, provider, session and
  conversation id. An active lane whose terminal was never observed stays
  `pending`. After a Director restart the stored fact is only last-run
  metadata: the launch reads `unverified` (the terminal record carries
  `live: false`), in agent status and in the tree alike, until the session
  owner observes that terminal again. After `start`, at most one `status`
  check to confirm launch. Do not seq/sleep-poll `status`; do other work or
  end the turn. Further `status` only if the user asks or a launch failed.
  Child results arrive later through the durable inbox, not that poll.
- Omit `repo` for read-only shared source: the child reads the parent's
  checkout and cannot report prepared code, checkpoint, build, promote, sync
  or fetch. Pass an explicit `repo` (an assigned existing worktree) to give the
  child its own source; paths are canonicalized, and a checkout already owned
  by another live agent is refused. Director allocates no worktrees. Lanes
  that share one canonical checkout cannot build, promote or sync while
  another lane on it has a human app open or a build queued or running. An AI
  test runs an immutable retained executable and does not hold the checkout.
- The Tasks view shows the tree on the left: Director, then agents nested
  under their delegating parent. Selecting an agent shows its own lane first,
  with its terminal, and its direct children to the right in stable sibling
  order: never grandchildren, siblings or ancestors. A leaf therefore shows
  its own lane; the tree is the only navigation. Director itself is no agent
  and shows the independent root lanes. A stopped or archived agent's own lane
  follows the normal lifecycle rules (an archived one is read-only) and
  selecting it never starts or reactivates anything; a deleted agent that only
  remains as a parent has no lane and shows just its children. Selection is a
  projection only: it never restarts, stops, focuses or transfers a lane, and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [makepad/makepad](https://github.com/makepad/makepad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
