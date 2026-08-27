---
trigger: always_on
description: READ THIS FULLY BEFORE ANY ACTION. These rules override your defaults. When a
---

# CLAUDE.md -- gnubg-android

READ THIS FULLY BEFORE ANY ACTION. These rules override your defaults. When a
rule here conflicts with what seems easier or more idiomatic, the rule wins.

## THE ONE RULE THAT MATTERS MOST

**gnubg is the SOLE authority for all game logic AND all analysis. PORT it,
NEVER reinvent it.**

This is a true port of GNU Backgammon's engine to Android. The vendored C
engine in engine-core/ already implements every rule, move, cube decision,
score, and analysis. Your job is to expose and wire that existing logic --
never to re-implement backgammon semantics in Kotlin or in new C.

"Game logic" is NOT the only forbidden territory. gnubg also computes, inside
its own evaluation, everything about what a position or move MEANS: shot
counts, primes, blots, anchors, board strength, position class, race vs
contact, blunder severity, which features matter. ALL of that is gnubg's, not
yours. The crime is not limited to deciding a rule -- it includes describing,
classifying, scoring, ranking, characterizing, or assigning quality/meaning to
any position or move. If gnubg computes it internally and you recompute it in
Kotlin, that is reinvention, FULL STOP, even when no rule of backgammon is
being decided.

Before writing ANY logic that decides legality, generates moves, evaluates a
position, scores a move, makes a cube decision, computes pips, progresses
match state, OR describes/classifies/ranks/characterizes a position or move:
STOP. Find the gnubg function that already does it and call it.
If you are about to write a loop that computes something about the board,
you are almost certainly reinventing -- search engine-core/ first.

Concretely, you MUST route through gnubg's own routines, e.g.:
- legal moves / sub-moves: GenerateMoves (via gnubg_mobile_get_legal_moves)
- move scoring / best move: FindnSaveBestMoves
- applying a move: ApplyMove / ApplySubMove
- position identity: PositionKey / PositionID
- matchstate replay: FixMatchState + ApplyMoveRecord (as AnalyzeGame does)
- cube decisions: the cube path in the engine, never a hand-rolled rule
- eval contexts / strength: aecSettings presets, esEvalChequer

If you cannot find the gnubg function, ASK -- do not invent one.

Kotlin may ONLY: draw the board, compute hitboxes in board-relative
coordinates, hold transient UI state (displayed dice order, uncommitted
submove snapshots for undo), and call JNI. Kotlin must NOT invent game
legality, cube legality, scoring, pip counts, match progression, OR any
analysis, description, classification, or quality judgement of a position or
move.

## ONE STATE, ONE WRITER, PROJECTED FROM GNUBG (added 2026-07-12, maintainer order)

This is the SOLE-AUTHORITY rule applied to *state*, not just logic. The engine
already holds the authoritative match state -- whose turn, the dice, the board,
the cube, what is legal, what is on the bar. The UI must be a PROJECTION of that
state, never a parallel replica the app maintains and re-synchronises by hand.

The rule, in force from now on:

  - **gnubg is the sole authority for match STATE, exactly as it is for match
    LOGIC.** Every field the engine can answer -- turn, board, dice, cube
    owner/value, canDouble, score, on-bar, game-over -- is READ from gnubg at
    the moment of projection and never inferred, cached-and-mutated, or
    reconstructed in Kotlin. A hand-maintained mirror of engine state is
    reinvention of the same kind as a hand-rolled cube rule.

  - **One writer.** The observable UI state (`_gameState`) has EXACTLY ONE
    function that writes the gnubg-derived fields: the projection
    (`readMatchState`). No other code path may write those fields -- no
    `_gameState.value = _gameState.value.copy(...)` scattered across handlers,
    and above all no SECOND thread writing the shadow. The clobber that froze
    the coach board (a `Dispatchers.Default` dice-watcher overwriting a phase
    the engine thread had just settled) is the canonical failure this forbids.

  - **Transient UI state is a THIN, EXPLICIT OVERLAY, kept apart.** A few things
    are genuinely app-side and have no engine equivalent: the coach HOLD (a
    move committed by the player but deliberately withheld from the engine so
    it can be studied), the displayed dice order, an uncommitted sub-move
    snapshot for undo, a "thinking/replying" indicator. These are allowed --
    but they must be a small, named overlay layered ON TOP of the projection,
    not woven into it as extra phases mutated in place. If you cannot say in one
    sentence why a piece of state has no gnubg source, it does not belong in the
    overlay -- it belongs in the projection, read from gnubg.

  - **Derived display values are computed at the projection, once.** Anything
    the view needs that is a pure function of engine state (pip counts,
    unplayable dice, cube display value, whether the Roll button shows) is
    derived inside the single projection from freshly-read engine state -- not
    recomputed independently in the view or in a handler, where it can disagree
    with the state it was meant to describe.

  - **Test before every state write and every gameplay commit:** is this write
    the projection, or an overlay change? If it is neither -- if it is a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clavierhaus/gnubg-android](https://github.com/clavierhaus/gnubg-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
