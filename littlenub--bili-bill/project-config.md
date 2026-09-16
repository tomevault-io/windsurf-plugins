---
trigger: always_on
description: This repository is worked on by multiple Codex development threads. Follow this guide before changing code or docs.
---

# Bili-Bill Agent Guide

This repository is worked on by multiple Codex development threads. Follow this guide before changing code or docs.

## Product Positioning

- Bili-Bill is a personal content ledger for local Bilibili consumption, not a replacement for Bilibili search, recommendation, dynamic feed, or creator relationship management.
- Dynamic Bill is for interest rebalancing. Do not describe it as "猜你喜欢", click prediction, or engagement-ranking.
- The AI assistant is a local ledger and knowledge helper. It must explain what evidence it used. Smart Favorites and Dynamic Bill remain usable from local evidence when AI is unavailable; current-video full-text generation must fail honestly instead of reviving a partial-evidence answer path.

## 0.14 Release Scope

- For the user-approved Chatbot/Wiki increment (#288), read [the Chatbot and Wiki scope](docs/scope-chatbot-wiki-increment.md) before implementation. Once its docs-only PR merges, it overrides conflicting current-video-only, full-text-only and no-saved-notes-to-AI rules only for its explicitly scoped new flows. Until the corresponding implementation lands, do not describe those capabilities as available. Other safety rules remain binding; no release/version change is implied.

- Before planning or dispatching any 0.14 work, read [the bounded learning-loop scope](docs/scope-0.14-bounded-learning-loop.md). The user accepted its direction; its implementation proposal takes effect only after the docs-only scope PR is reviewed and merged. It narrows the first release, not the safety rules below or existing 0.13 runtime behavior.
- For that bounded release, its scope, LG-0 to LG-5 sequence, and completion definition override conflicting full-library obligations in the older 0.14 PRD, acceptance, architecture, glossary, development plan and ADRs. LG-0 must still freeze and validate finite contracts before production implementation.
- The proposed UX-014 addition in `docs/ux-0.14-video-assistant.md` makes compact/expanded in-page assistant simplification a 0.14 requirement after its scope PR merges. It permits an independent presentation-only track for existing assistant behavior, not early persistent-learning implementation. LG-5 then requires both LG-4 and UX-014. Use focused UI regression for layout changes; do not rerun storage matrices for CSS or copy changes.
- `docs/development-plan-0.14.md`, `docs/architecture/gate-contract-0.14-storage-search-and-backup.md`, and `docs/qa-0.14-integration-matrix.md` remain byte-preserved because B1 receipts bind them. Their full-library Gate-first language continues to govern old A2/C/D/E and old implementation tasks, not the new bounded route. Do not refresh receipts or relax the verifier merely to edit those documents.
- Old issues do not close or become dispatchable automatically. Scope adoption is not authorization to mark Ready, merge, close, publish, change package version, or collect data. B1 remains synthetic evidence; A2 remains unpassed until supported independently.

## Safety Boundaries

- Do not read, copy, log, or submit local key files, including `C:\Users\LittleNub\Desktop\Key.txt`.
- Do not read or copy Cookie files, browser profiles, Bilibili login-state files, or personal profile files.
- Do not upload full watch history, full favorites, full following lists, full feedback records, full notes, or full local database contents.
- Do not write back to Bilibili relationships or content: no follow/unfollow, favorite-folder edits, comments, likes, coins, or collection mutations.
- Use current browser extension runtime state only when a task explicitly needs logged-in smoke testing. Never extract credentials or session data from disk.
- Keep Manifest permissions minimal. Do not add the `cookies` permission unless an accepted issue introduces a reviewed `chrome.cookies` use case.

## Language And UX Invariants

- Dynamic Bill status copy is fixed: `未打开 / 已打开 / 已消费 / 已处理`.
- Do not reintroduce `未消费` as ordinary user-facing copy in `dashboard`, `popup`, `public`, `src`, `README.md`, or new docs. Tests and product contracts may name it only as an explicit prohibition or scan target.
- If the active part has no reliable primary text source, do not claim a full-video summary, highlights, or answer.
- Per user decision #286, returned model prose remains readable when output/evidence validation fails. Label it as unverified; do not promote it into verified citations, timestamp jumps or saved evidence. Cancellation, stale requests, source authorization and privacy checks remain binding.
- Current-video assistant answers must label the source video, part when relevant, and natural text source name (`B站字幕` or `本地转录`).
- Smart Favorites Q&A answers must cite source videos and explain why each cited video is relevant.
- Video key nodes must have evidence. Do not fabricate timestamps.
- Auto-jump behavior must be disabled by default and require explicit user confirmation.
- User-visible copy must not expose raw engineering fields or runtime errors such as `fallback`, `transcript`, `confidence`, `sourceHash`, `segmentId`, or `subtitle_url`.

## Worktree And PR Workflow

- Use one GitHub issue per branch/worktree. Keep PR scope limited to that issue.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LittleNuB/BILI-BILL](https://github.com/LittleNuB/BILI-BILL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
