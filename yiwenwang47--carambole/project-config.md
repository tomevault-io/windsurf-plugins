---
trigger: always_on
description: > ## ▶ ACTIVE TASK — START HERE (next session, continue without re-asking)
---

# Carambole — French Grammar Trainer

> ## ▶ ACTIVE TASK — START HERE (next session, continue without re-asking)
>
 Everything below is DONE, MERGED to `main`, and PUSHED to origin (2026-06-19).
> Use the same discipline (branch off `main`; subagent-driven implement → spec
> review → code-quality review per task; build gate `xcodebuild build ... -scheme
> carambole -destination 'platform=iOS Simulator,name=iPhone 16'
> CODE_SIGNING_ALLOWED=NO` = EXIT 0; screenshot UI changes; commit per task; update
> this block as you go).
>
> **▶ PROGRESS BAR + TIME-TO-FINISH ESTIMATE (branch `progress-tracking-estimate`,
> head `c2eeccc` — NOT yet merged/pushed; 2026-06-21).** Answers "how far through the
> book am I / how much more time to finish" on the book home. Driven by web research
> (Lingvist's learned/learning/at-risk buckets; course-completion stats: only ~8%
> finish, realistic pace-based timelines → 3.2× more likely to finish) → design
> `docs/plans/2026-06-21-progress-tracking-design.md`. Locked decisions: estimate =
> **recent actual mastery pace** (not the daily-goal setting); finish-line = element
> **mastered** (`distinct_correct ≥ 3` OR suspended — the live `satisfiedIDs` gate);
> headline = **3-segment book bar** (maîtrisé / en cours / à venir); scope = bar +
> estimate only (History untouched). Also FIXED a confusing daily zone: the old
> `Aujourd'hui · X new · Y due` (to-do) sat right above `N appris · M révisés` (done)
> — two look-alike number pairs, opposite meaning. New: to-do counts move onto the
> **buttons** (`Apprendre · N` / `Réviser · M`, badge hidden at 0); one `fait : …`
> done-line remains. **3 tasks, subagent-driven (implement → spec ✅ → code-quality ✅
> each), + final whole-branch review = SHIP:** **T1** `90361d8` `mastery_log(element_id
> PK, day_start)` table + `user_version` **3** migration (additive; backfilled once
> from `last_review_date`, forward-only) + write-hooks in `upsertReviewStatus` AND
> `setSuspended` (the only 2 satisfaction paths) + `fetchMasteryLog()`. **T2**
> `594f80a` pure `ProgressEstimate` value type (`Core/ProgressEstimate.swift`):
> segments + `percentComplete` (caps 99 unless truly done, floors 1 if any progress)
> + trailing-14-day mastery pace ÷ elapsed-calendar-days → `daysRemaining` /
> `projectedFinish`, with an early-state gate (suppress unless ≥3 days history AND ≥3
> window masteries; nil → "Continue quelques jours…"); deterministic (injected
> `today`/`calendar`), 8-case `#if DEBUG _selfTest()` wired at launch. **T3** `c2eeccc`
> `BookView`/`BookViewModel`: capsule 3-segment bar + % + 3-state estimate line, French
> formatting (`≈ N j`/`≈ N mois`, `~15 août`, `8,5/j`) IN the view; `mastered`/segments/%
> come from the LIVE satisfied gate, `mastery_log` feeds ONLY the pace (no desync).
> Build EXIT 0 (sim id `7C7BA0A7-…`; `name:iPhone 16` still broken). Launch self-tests
> pass. Populated bar SCREENSHOT-validated against the real `BookView` via the
> `CB_SCREEN` harness (reverted; same gotcha as prior) — caught+fixed a doubled-`≈`.
> STILL UNTESTED HERE: literal on-device tap into BookView (BookView sits behind
> `LibraryView`, taps unautomatable — see NEXT (a)). FINISHING THE BRANCH (merge/PR)
> awaiting user choice.
>
> **▶ DRILL CARD: VERTICAL LOCKED + FIT-TO-HEIGHT (commit `a5dff57` — MERGED to
> `main` + PUSHED; 2026-06-21).** Bug: drill content overflowed with no way to
> reach the bottom, and vertical scroll was inconsistent (locked on live drills,
> scrolling in review — distracting). Root cause: each card's `ScrollView` fought
> the carousel's `simultaneousGesture` drag → non-deterministic vertical
> arbitration. Fix (DrillView.swift only): **drop the per-card `ScrollView`** →
> vertical is now deterministically LOCKED everywhere; the card is instead SIZED to
> fit. (1) `GeometryReader` `compact` mode (`geo.height < 540`): when the region is
> tight (keyboard up, or small screen / SE) hide the secondary chrome (cue,
> translation, Indice/Écouter) so prompt + input + footer always fit — also stops
> the English translation from half-giving the answer while typing. (2) Tighter
> `contentStack` spacing/padding (`xl`→`lg`, `xl`→`md`) reclaims ~the exitBar
> height. (3) Build/MC token input wrapped in a bounded `.basedOnSize` ScrollView —
> invisible for normal drills (locked feel holds), scrolls ONLY the slots+bank when
> a fully-built long answer (worst case: 10 slots / 16 chips) outgrows the card;
> prompt + footer stay fixed. (4) Review explanation wrapped in a bounded 220pt
> `.basedOnSize` ScrollView so long bilingual explanations fit without scrolling the
> whole locked card. Build EXIT 0. SCREENSHOT-VALIDATED on sim against the REAL
> `DrillView` via a temp `#if DEBUG` env-gated harness (`CB_HARNESS`/`CB_SCREEN`,
> launched with `SIMCTL_CHILD_*` env, reverted + deleted — same revert-then-
> uninstall/reinstall gotcha applies), using REAL worst-case content and a faithful
> exit-bar-height spacer: worst construis fits all 16 chips, typing+keyboard goes
> compact and fits, review explanation bounded-scrolls. STILL UNTESTED HERE (taps
> unautomatable): the literal place-all-10-tokens-then-scroll gesture + the two
> nested `.basedOnSize` scrolls coexisting with the carousel horizontal swipe —

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yiwenwang47/carambole](https://github.com/yiwenwang47/carambole) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
