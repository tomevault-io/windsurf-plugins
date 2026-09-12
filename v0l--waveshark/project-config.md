---
trigger: always_on
description: Read [`docs/design.md`](docs/design.md) first. It has the layout, the
---

# Working on WaveShark

Read [`docs/design.md`](docs/design.md) first. It has the layout, the
measurements behind the current shape, and the mistakes that produced it. Then
[`docs/protocols.md`](docs/protocols.md) for what is decoded and what is not,
[`docs/views.md`](docs/views.md) before adding a pane,
[`docs/references.md`](docs/references.md) before adding a data source or a
decoder, since it holds the terms each publisher asks for and the rules two
of them make the code follow,
[`docs/mcp.md`](docs/mcp.md) before adding or changing what an agent can reach
over `--mcp-listen`, and
[`docs/web.md`](docs/web.md) only knowing it is a plan and not a status report.
What follows is the two rules that are easiest to break without noticing, and
the procedure for adding a capture to the test corpus.

Two cargo features change what builds: `tea` is TETRA decryption and key
recovery, and `ambe` is DMR speech through `crates/mbe`. Both are on by
default, so a plain `cargo test` at the root builds them and `crates/mbe` is
compiled through the `ambe` feature, though it is still not a default
workspace member and `cargo test -p mbe` has to be asked for by name. What the
release workflow publishes is built with `--no-default-features --features
limesdr,stt,mcp`, because a cipher and a patent-encumbered vocoder are things
a person compiles for themselves rather than things this project ships. `mcp`
is the agent server behind `--mcp-listen` and is shipped: nothing listens
until an address is given.

## Everything the receiver does is in the graph

The flow graph is not an implementation detail of the signal path. It is the
description of what the receiver is doing, and the chain view is that
description drawn. Anything the receiver does that is not a node is invisible
there: it cannot be seen, tapped, parameterised, saved in a patch, or moved by
an operator, and the drawing on the screen is then a lie by omission.

So: **if it processes, routes, records, mixes or decides, it is a node.** The
spectrum is a node. The recorder's ring is a node. The DC blocker, the packet
bus, the packet log, the raw IQ capture, every front end the scanner table
places, and every decoder the auto node builds for a source it found are all
nodes. A helper that runs once per block from the radio thread is not a design
choice, it is a node somebody has not written yet.

What this rules out in practice:

- No processing in the radio loop. `crates/app/src/radio.rs` moves blocks and
  commands. If you find yourself filtering, mixing or deciding in it, that
  belongs in a node the graph holds.
- The same applies to transmitting. `derived_patch` draws four stages,
  `tx_clock`, the source, the modulator and `radio_tx`, whether or not a key is
  down, so the chain can be looked at before it is used.
- No state that only one hard-coded stage can produce. Reaching into a named
  stage by field (`self.m17`, `self.record`) works until the same front end
  exists somewhere else. Live speech was read from the one M17 stage the
  scanner table places, so every M17 transmission the auto node found for
  itself decoded, logged, and played back as silence. The fix was a port kind
  every front end can publish on: `Receiver::voices` reads `PortKind::Voice`
  off the whole graph rather than off a named stage.
- No behaviour keyed on a protocol name where a capability will do. Ask nodes
  what they can do; do not keep a list here of which ones can.
- A composite node owns inner graphs and must say so through
  `Node::subgraphs`, or the work it does disappears from the view. The auto
  node is a node holding a graph per front end on every open source, and
  reports all of them; a bank is a node holding hundreds of channels running
  the one chain, and reports that with a count.

Reading state back by downcasting is fine and is how the spectrum, the
recorder and the capture are read. What is not fine is the work itself
happening outside.

The speaker is on the graph too. `PortKind::Voice` carries decoded speech
with the call it belongs to, front ends publish it on a port of their own,
every listening channel's chain ends in real audio, and `AudioBusNode`
(`crates/app/src/audiobus.rs`) is the node all of them are wired into: one
input per strip, a level and a mute on each, the subscriptions that decide
which calls are heard, the master, the clip, and one stereo output the radio
thread hands to the device. The mix used to be a loop in the radio thread and
the faders were fields on a channel list, so a demodulator drawn by hand had
nothing to be wired to. If you find yourself summing audio anywhere but on
the bus, or keeping a level anywhere but as one of its parameters, you are
adding that back.

## A protocol is one `impl Protocol`, and nothing else

What the receiver knows about a protocol lives in one place:
`crates/nodes/src/protocol.rs` declares the trait, and each `*_nodes.rs` has
the one implementation for its protocol, registered in `protocol::all()`.
Where it can be, what stream it reads, how sticky a channel it has read on
is, the chain of stages that reads it, and which packets repeat. The auto
node, the scanner table, the strip's mode menu, the spectrum markers and the
chain view labels all ask that registry and keep no list of their own.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [v0l/waveshark](https://github.com/v0l/waveshark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
