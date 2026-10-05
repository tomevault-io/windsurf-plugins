---
trigger: always_on
description: A physical-modeling software synthesizer. End goal: generate the music for a **64k PC
---

# pfsynth

A physical-modeling software synthesizer. End goal: generate the music for a **64k PC
intro** targeting **Revision 2027** (~late March 2027).

## The one constraint that drives every decision

In a 64k intro the scarce resource is **compressed bytes of data**, not CPU:

- Samples are hopeless — incompressible waveform data.
- Physical modeling is the opposite: the instrument is **code** (a fixed, one-time cost)
  and a song is just a tiny event stream plus a few patch floats.
- Intros render the whole tune to a PCM buffer during the precalc/loading screen, so we
  are **not real-time**. CPU is effectively free.

**Therefore: optimize for small, compressible code and data. Do NOT optimize for runtime
speed.** Prefer generating constants in code over storing float tables. Prefer compact
integer event streams. A transcendental call per sample in the audio path is fine — it
runs once at precalc and costs zero bytes.

## Architecture — keep this split clean from day one

- `src/core/` — the **synthesis kernel**. Pure, portable C. No external deps beyond
  `<math.h>`/`<string.h>`. **No dynamic allocation, ever** (structs are fully defined in
  the headers so the host can stack/static-allocate them). This is what eventually ships
  inside the intro as the replayer, so code size and compressibility are first-class
  concerns. Must stay platform-agnostic — the eventual shipping target is Windows PE x64,
  but it must never know that.
- `src/host/` — **offline tooling**: WAV writing, CLI, the render harness. Runs only on
  the dev machine (macOS/clang for now). Unlimited size, free to use libc/libm/whatever.
  **Never let host concerns leak into core.**

## Synthesis approach

Digital-waveguide piano as the flagship voice. The current voice (`pf_string`) is a small
bank of coupled dispersive, lossy waveguide loops (the 2–3 unison strings of a piano note)
sharing one nonlinear felt hammer. Each loop is:

- **Delay line** — the waveguide loop, tuned to the note pitch (integer delay + a
  first-order allpass fractional tuner).
- **Loss filter** — a one-pole lowpass in the loop for frequency-dependent decay (bright
  attack, mellow tail). Overall loop gain sets the fundamental T60, which is **pitch-
  dependent**: `T60(f0) = decay_t60·(f0/440)^decay_pitch`, `decay_pitch ≈ -0.6` — the bass
  rings many seconds, the treble dies in ~1–2 s, fit to a real grand.
- **Dispersion** — a cascade of first-order allpass filters producing stiff-string
  inharmonicity, partials stretched as `f_n ≈ n·f0·√(1 + B·n²)`. `B` is **pitch-dependent**,
  fit to a real grand: `B(f0) = inharmonicity·(f0/440)^inharm_pitch` with `inharm_pitch ≈
  1.5` — the treble is far stiffer/sharper than the bass. The allpass coefficient is fit at
  init to approximate the target stretch (one coefficient can't match all partials exactly;
  it also realizes ~1.8× the requested `B`, so the `inharmonicity` param is pre-divided to
  land the *output* B on the measured value — see Calibration).
- **Nonlinear felt hammer** — a mass with a hardening spring, force `F = K·δ^p` (compression
  `δ`, exponent `p ≈ 2–3`). The hammer–string interaction is a delay-free loop (force depends
  on string velocity depends on force); it's resolved per sample with an **implicit Newton
  step**. The hammer injects the excitation — this is the attack and the soul of the sound.
  **Velocity → brightness**: the felt stiffness `K` scales with strike velocity as
  `(vel/0.6)^hammer_vel_hardness`, so a fast strike meets a harder hammer (shorter contact →
  brighter, clangy ff) and a slow one a softer hammer (longer contact → mellow pp). This is
  the main dynamics→timbre coupling; without it, velocity only changes loudness. (The rich
  soundboard masks some of it in the mix — a trade against the body fullness.)
- **Graduated hammers + contact-noise bite** — the hammer is stiffer/lighter toward the
  treble (`hammer_pitch_k/m`) so the strike stays short relative to the period and excites a
  rich harmonic series at every pitch (a fixed hammer lowpasses high notes into dull sines).
  A smooth felt force still can't make the percussive *attack flash*, so `attack` adds a
  short broadband contact-noise burst on the radiated output at each onset, enveloped to the
  attack peak (the hammer/key "knock") — the difference between "piano" and "harp."
- **Strike-point comb** — the hammer hits at `strike_pos` (a fraction of the string length,
  ~1/8), so partials with a node there can't be excited. Modeled as the injected excitation
  being high-passed by `(1 − z^−β)`, `β = strike_pos·period`: the delayed term is the wave
  reflecting off the near end back to the hammer (which feels it, so the comb falls out of
  the physics, not a post-EQ). Notches partials `n = 1/strike_pos, 2/strike_pos, …` and
  boosts the low-mid partials between — the characteristic woody piano shape. `0` = off.

The voice also has a **damper**: `pf_string_release()` switches the loop to a fast
`release_t60` decay, modeling the felt damper falling on note-off. A held sustain pedal
just means the host doesn't call it.

### Coupled strings — beating + two-stage decay
A single loop decays as a pure exponential, which sounds electronic. A real note has 2–3

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [olaugh/pfsynth](https://github.com/olaugh/pfsynth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
