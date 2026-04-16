---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

# PRD: Mobile-First Spanish Class App (Front-End Only)

## 1) Summary & Goals

A super-simple, mobile-first web app for a once-a-week high school elective. The app shows the current week’s vocabulary and example phrases, lets students tap to reveal meanings and conjugations, and allows quick “reviewed this session” check-offs. No login, no backend — just a static deploy with one editable config file you can update weekly.

**Core goals**

* Be fast, dead simple, and delightful on phones.
* One tap reveals meaning; another tap shows conjugations; another returns to Spanish.
* Press-to-play audio for phrases.
* Per-session review checkmarks with an easy reset.
* “Single source of truth” via a weekly config file (JSON).

**Non-goals**

* Accounts, grades, or analytics.
* Teacher dashboard or per-student tracking beyond a local device session.

---

## 2) Users & Constraints

**Primary users:** 13–18 year-old students using personal phones.
**Environment:** School Wi-Fi/cellular, mixed iOS/Android, short sessions (5–15 min).
**Constraint:** App must run entirely client-side (static hosting). No database.

---

## 3) Scope of Features

### 3.1 Must-Haves

1. **Weekly Vocab Browser**

   * Display only the current week by default (from config).
   * Each vocab item is a tappable card that cycles states:

     * **State A:** Spanish (default)
     * **State B:** English translation
     * **State C:** Conjugations/Forms (if present)
     * **State A again**
   * Cards show a subtle “reviewed” check when the student marks them during the session.

2. **Phrases/Sentences with Audio**

   * List of example phrases that use this week’s vocab (highlight matched words).
   * **Play button**: speak phrase in Spanish.
   * Tap to mark a phrase as reviewed (per session).

3. **Session Review Tracking**

   * A floating **“Session progress”** bar with counts (e.g., “Words: 6/20 · Phrases: 3/8”).
   * **Reset Session** button clears current checkmarks.
   * Session state persists in **localStorage** and auto-expires after 24 hours or manual reset.

4. **One Config File Update**

   * All content controlled by a single `config.json` checked into the repo and deployed with the site.

### 3.2 Nice-to-Haves (V1.1+)

* **Week Picker** to revisit prior weeks (hidden by default, toggleable).
* **PWA** install + offline cache (static assets + current week config).
* **Dark mode** toggle.
* **Speed control** for audio playback (0.75x, 1x, 1.25x).
* **Shuffle mode** for vocab drilling.

---

## 4) Accessibility & UX Principles

* **Mobile-first** layout; target comfortable tap areas (44×44 px).
* **WCAG 2.1 AA**: color contrast, focus states, screen reader labels (ARIA), logical heading order.
* **Tap cycle** must be obvious (micro-animation + helper text “tap to cycle” on first use).
* **Audio**: Play only on user gesture; show **Play/Stop** and a minimal progress bar; provide **phonetic hint** or slow speak option if feasible.

---

## 5) Information Architecture

* **Home / Week View**

  * Header: Week title + info (e.g., “Week 6: La Comida”)
  * Tabs or segmented buttons: **Vocabulary** | **Phrases**
  * Content grid/list:

    * Vocabulary cards
    * Phrase list with play buttons
  * Footer / floating bar: Session progress + Reset

* **(Optional) Week Selector** (modal or side sheet)

  * List of weeks (title + date range)

---

## 6) Functional Requirements

### 6.1 Vocab Cards

* **Tap behavior** cycles through 3 states per item:

  * A → B → C → A
* **Conjugations/forms** display:

  * Verbs: person/tense sets (present, preterite, imperfect; min set in V1)
  * Nouns/adjectives: gender/number forms if provided
* **Reviewed toggle**: a check icon in the corner; tap check area to mark/unmark.

### 6.2 Phrase Items

* Show Spanish phrase, auto-highlight words that match this week’s vocab (case-insensitive).
* **Audio**:

  * **Primary**: Web Speech API (SpeechSynthesis) using `es-ES` voice if present on device.
  * **Fallback**: Pre-generated MP3s (optional field in config) served statically.
* **Reviewed toggle**: same pattern as vocab.

### 6.3 Session Tracking

* **Storage**: `localStorage` keys scoped by `week_id`:

  * `session.vocabReviewed.{week_id}` → array of vocab IDs
  * `session.phraseReviewed.{week_id}` → array of phrase IDs
  * `session.startedAt.{week_id}` → timestamp
* **Auto-expire** session after 24 hours (on load: if `now - startedAt > 24h`, clear).

### 6.4 Config Resolution

* Load `/config.json` at app start.
* Use `current_week_id` to populate default view.
* If fetch fails, show a friendly offline message and prompt to refresh.

---

## 7) Non-Functional Requirements

**Performance**

* Time to interactive < 2s on mid-range phones (over decent connection).
* Bundle size target < 150KB gzipped (no heavy frameworks; consider Preact or vanilla).

**Compatibility**

* iOS Safari 15+, Chrome/Edge/Firefox (latest 2 versions).
* No sign-in; no cookies; only localStorage.

**Security & Privacy**

* No PII. No tracking/analytics by default.
* HTTPS required (static host).

**Reliability**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OlsonDerek) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
