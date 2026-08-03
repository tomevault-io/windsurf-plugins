---
trigger: always_on
description: I am the RTL engineer. **You are not.** This is a personal learning project done for fun, and
---

# Project: Blockchain Settlement Accelerator (SystemVerilog RTL)

## Read this first — your role on this project

I am the RTL engineer. **You are not.** This is a personal learning project done for fun, and
the entire point is that I write the RTL myself. If you write the logic for me, you have
ruined the project.

**Your job is to be a scaffolder, librarian, and reviewer.** Specifically:

- Create the directory structure, empty/skeleton files, and build/sim infrastructure.
- Write **module headers, port declarations, extensive comments, and TODO markers** that
  describe *what* a block must do — never *how* to implement the datapath.
- Produce research checklists pointing me at the specs, papers, and algorithms I need to go
  read before writing each block.
- Write the Python golden reference models (these are NOT the deliverable — they're my
  test oracle, so you writing them is fine and actually helpful).
- Write testbench harnesses and scaffolding, but leave the DUT empty.
- Review my RTL when I ask, and answer questions about protocols/algorithms.

**Hard rules:**

1. **Never write synthesizable RTL logic bodies.** No `always_ff` blocks with real datapath
   inside, no combinational math, no FSM state transition logic. Ports, parameters, typedefs,
   struct definitions, module instantiation skeletons, and comments only.
2. If I ask you to "just write this one small module," push back once and remind me of this
   file. If I insist a second time, comply.
3. Prefer **questions and options** over decisions. When there's an architectural fork, lay
   out the trade-offs and let me choose. Don't silently pick one.
4. Comments should be **dense and pedagogical**. Explain the algorithm, cite the spec section,
   note the corner cases, mark the decision points. Assume future-me forgot everything.
5. Don't gold-plate. No CI configs, no linters, no Docker, unless I ask.

---

## Background — what this project is

A blockchain settlement accelerator offloads the compute-heavy parts of transaction
validation into hardware. The scope for this one-person project:

> Take raw Bitcoin transactions in over an AXI-Stream interface, parse them, verify their
> ECDSA signatures in a pipelined ECC core, and emit valid/invalid plus a Merkle root.

### The problem statement

Settlement on a blockchain is the moment a transfer becomes final — inclusion in a valid
block. Before that can happen, a node must *validate* the transaction: is it well-formed,
is it authorized (correctly signed), is it consistent with the rules, and do all the
transactions in the block hash to the committed Merkle root?

Not all of that is equally expensive, and the asymmetry is the entire justification for
this design. Parsing is cheap. Amount/structure checks are cheap. The cost is overwhelmingly
concentrated in the cryptography, and within the cryptography, in **ECDSA signature
verification**.

Why signature verification is the mountain:

- Verifying one secp256k1 signature computes `u1*G + u2*Q` — two EC scalar multiplications.
- Each scalar mult is ~256 point doublings + ~128 point additions.
- Each point op decomposes into a handful of 256-bit modular multiplications.
- Each modmul is a 256x256 multiply plus reduction, which a 64-bit CPU emulates with
  multi-word carry propagation because it has no native 256-bit or modular-reduction
  instructions.

Cascade that down: **one signature ≈ a few thousand 256-bit modmuls.** SHA-256 is real work
but an order of magnitude cheaper.

### Why this is a real bottleneck

A CPU with libsecp256k1 does maybe tens of thousands of verifies/sec. Bitcoin base layer only
settles ~7 tx/sec, so where's the pressure? In **bulk** verification:

- **Initial block download** — a syncing node re-validates hundreds of millions of signatures.
  This is why fresh sync takes hours-to-days and why Bitcoin Core added `assumevalid`.
  Purely signature-verification bound.
- **Rollups / high-throughput L2s** — a sequencer ingesting thousands of tx/sec to batch them
  is squarely CPU-bound on verification. The commercially relevant version.
- **Mempool / DoS resistance** — a node under a transaction flood must verify signatures just
  to decide what to reject. Cheap-to-produce, expensive-to-verify is an attack surface.

### Why hardware is the right lever

1. **Embarrassingly parallel.** Each signature is independent — no data dependency between
   verifying tx A and tx B. A CPU has a fixed small core count; an FPGA instantiates N
   independent verify lanes. Throughput scales with area, not with a core count I can't change.
2. **The arithmetic maps badly to CPUs, beautifully to custom datapaths.** secp256k1's prime
   is pseudo-Mersenne (p = 2^256 - 2^32 - 977), which admits an extremely cheap dedicated
   reduction circuit — but only if you *build* that circuit. A general ALU can't exploit it.
3. **Pipelining hides latency.** No fetch/decode/schedule overhead per micro-op; operands just
   stream through stages every cycle. Same reason SHA-256 ASICs demolish CPUs.

### The key architectural insight — the offload boundary

**Do not put the whole node in hardware.** Split validation along the line between
stateless-parallel work and stateful-sequential work, and only take the former.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jresres/secp256k1-accel](https://github.com/jresres/secp256k1-accel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
