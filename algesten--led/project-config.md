---
trigger: always_on
description: A terminal text editor written in Rust using a custom single-threaded push-based FRP framework (`crates/core/src/rx.rs`).
---

# led2 Editor

A terminal text editor written in Rust using a custom single-threaded push-based FRP framework (`crates/core/src/rx.rs`).

## FRP Architecture Principles

The architecture follows the CycleJS/froop pattern. The core cycle is:

```
Driver inputs -> model() -> fold/reduce -> state -> derived() -> Driver outputs -> cycle back
```

State flows in a single direction. An imitator (`Stream::new()` fed by `fold_into`) allows the state stream to be referenced before it is defined, closing the cycle. Drivers are the boundary where side effects happen — the model and derived layers are pure.

### The Golden Rule: All Logic in Combinators, Reducer Just Assigns

This is the single most important principle. In well-structured FRP code:

1. **Combinator chains** (map, filter, filter_map, sample_combine, dedupe, merge, etc.) contain all decision-making, transformation, and business logic.
2. **The reducer** (fold/fold_into) receives pre-computed values and mechanically assigns them to state fields. No conditionals, no loops, no function calls with logic.
3. **Each stream** produces exactly the `Mut` variant needed — the "what to update" decision is made upstream.

This is how it works in practice in well-structured FRP apps:

```
// Each concern is a small combinator chain that produces an update:
let audio_route_update = merge(
    state.filter_map(|s| s.recorder_state.map(|r| r.audio_route)).dedupe(),
    intent.audio_route_in
).map(StateUpdate::AudioRoute);

let want_camera_update = merge(intent.want_camera, auto_camera_on, auto_camera_off)
    .map(StateUpdate::WantCamera);

// All updates merged:
let state_update = merge(audio_route_update, want_camera_update, /* 40+ more */);

// Reducer: TRIVIAL fold. No logic whatsoever.
let real_state = init.map(|first_state|
    state_update.fold(first_state, |state, update| update.apply_to(state))
).flatten();

// Where apply_to is mechanical:
fn apply_to(self, mut state: AppState) -> AppState {
    match self {
        StateUpdate::AudioRoute(v) => { state.audio_route = v; }
        StateUpdate::WantCamera(v) => { state.want_camera = v; }
        // Each case: one assignment. That's it.
    }
    state
}
```

When a single signal needs to change multiple state fields, it splits into multiple streams — one per field — that each emit a separate update. For example, a "task next" signal:

```
let task_next_step = task_move
    .filter(|m| m.is_next)
    .sample_combine(&state)
    .filter_map(|(_, s)| s.next_task_state())
    .map(StateUpdate::SetGuidedTaskState);

let task_next_visited = task_move
    .filter(|m| m.is_next)
    .map(|_| StateUpdate::SetHasVisitedBrowser(false));

let task_next = merge(task_next_step, task_next_visited);
```

Not one `Mut::TaskNext` with logic in the reducer — two separate streams, each producing a single-field update.

### Principle 1: The Reducer Must Be Trivial — "Just Assign"

Every `Mut` variant handler in the `fold_into` must be 1-3 lines of direct field assignment. No `if`, no loops, no function calls containing logic. All decision-making lives upstream in the combinator chains that produce the `Mut`.

**Bad:**
```rust
Mut::ResumeComplete => {
    s.phase = Phase::Running;
    if let Some(order) = s.session.active_tab_order.take() {
        let non_preview_tabs: Vec<_> = s.tabs.iter().filter(|t| !t.is_preview()).collect();
        if let Some(tab) = non_preview_tabs.get(order) {
            s.active_tab = Some(tab.path().clone());
        }
    }
    // ... 25 more lines
}
```

**Good:**
```rust
Mut::SetPhase(phase) => { s.phase = phase; }
Mut::SetActiveTab(path) => { s.active_tab = Some(path); }
Mut::SetFocus(focus) => { s.focus = focus; }
```

### Principle 2: One Signal, Many Muts — Split via Multiple Streams

When a signal needs to change multiple parts of state, split it into multiple streams that each emit a fine-grained `Mut`. Don't bundle logic into one fat `Mut` variant.

**Bad:** `Mut::SessionRestored` carries 10 fields and the reducer has 40 lines handling it.

**Good:**
```rust
let session_phase_s = session_restored_s
    .map(|sr| if sr.pending_opens.is_empty() {
        Mut::SetPhase(Phase::Running)
    } else {
        Mut::SetPhase(Phase::Resuming)
    });

let session_tabs_s = session_restored_s
    .flat_map(|sr| sr.pending_opens.iter().map(|p| Mut::AddTab(p.clone())));

let session_browser_s = session_restored_s
    .map(|sr| Mut::SetBrowserState {
        selected: sr.browser_selected,
        scroll: sr.browser_scroll_offset,
    });
```

**The pattern: Common parent stream → multiple child chains.** When refactoring a fat reducer handler, first create a stream for the triggering event, then branch it into children that each produce one fine-grained `Mut`:

```rust
// Common parent: computed once, shared by all children
let resume_complete_s = state
    .filter(|s| s.phase == Phase::Resuming)
    .filter(|s| s.session.resume.iter().all(|e| e.state != ResumeState::Pending));

// Child 1: set phase
let resume_phase_s = resume_complete_s
    .map(|_| Mut::SetPhase(Phase::Running));

// Child 2: resolve active tab (all decision-making here, not in reducer)
let resume_tab_s = resume_complete_s

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/algesten) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
