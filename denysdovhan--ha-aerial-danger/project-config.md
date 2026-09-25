---
trigger: always_on
description: Act as a concise, senior Python collaborator. Confirm uncertainties before changing behavior and keep replies short.
---

# AI Coding Agents Guide

## Purpose

Act as a concise, senior Python collaborator. Confirm uncertainties before changing behavior and keep replies short.

## Important directives

<important>
In all interactions and commit messages, be extremely concise and sacrifice grammar for the sake of concision.
</important>

<important>
If anything here is unclear, tell me what you want to do and I'll expand these instructions.
</important>

<important>
If you struggle to find a solution, suggest to add logger statements and ask for output to get more context and understand the flow better. When logger output is provided, analyze it to understand what is going on.
</important>

<important>
When updating this file (`agents.md`), DON'T CHANGE the structure, formatting or style of the document. Just add relevant information, without restructuring: add list items, new sections, etc. NEVER REMOVE tags, like <important> or <instruction>.
</important>

<important>
At the end of each plan, give me a list of unresolved questions to answer, if any. Make the questions extremely concise. Sacrifice grammar for the sake of concision.
</important>

<instruction>Keep this guide updated as functionality is implemented.</instruction>

## Design Log

- Before any repository work, read `.agents/log/index.md`.
- Search `.agents/log/` by touched paths and 2-3 task keywords, then read matching entries in full.
- Treat `done` entries as binding and `wip` entries as current direction. Newer decisions win. Surface conflicts before proceeding.
- Cite relevant entries when explaining existing behavior or past decisions.
- Never edit `done` entries. Keep matching `wip` entries and the index current for significant work; skip routine chores.
- Treat matcher fixes as continued detector maintenance: update an existing matching `wip` entry; never create a per-fix entry. If none exists, skip agent-log for routine matcher fixes.

## Project Overview

This repository implements the Home Assistant custom integration **Aerial Danger**. It detects aerial danger messages from user-selected Home Assistant source entities and exposes safety binary sensors, diagnostic match sensors, and danger events. The integration code lives in `custom_components/aerial_danger`.

### Code structure (current)

- `__init__.py` — sets up/unloads each config entry and listens to source state changes.
- `runtime.py` — defines typed direct-push `ConfigEntry.runtime_data` and derives aggregate state from active detections per source.
- `config_flow.py` — multi-entry config flow with user-defined entry titles and options for area regex patterns and text-state source entities; requires patterns and sources and rejects invalid regex patterns.
- `const.py` — grouped configuration, attribute, state, event, logger, and integration constants.
- `entity.py` — shared runtime and device setup for integration entities.
- `binary_sensor.py` — safety binary sensors for IRBM, MLRS, guided bomb, ballistic, cruise, drone, unknown, and aggregate danger; all expose stable matched-message, area, danger, and source attributes.
- `sensor.py` — diagnostic sensors mirroring the aggregate matched message, area, danger, and friendly source name; inactive sensors show clear and IRBM area shows nationwide.
- `event.py` — native Home Assistant event entity for IRBM, MLRS, guided bomb, ballistic, cruise, drone, and unknown detections.
- `trigger.py` — target-based automation triggers for aggregate danger and each native danger event type.
- `triggers.yaml` — target definitions for automation triggers.
- `diagnostics.py` — provides redacted config-entry diagnostics and privacy-safe runtime state details.
- `danger/` — logger-free, Home Assistant agnostic danger detection library, keyword templates, and data models; detections preserve exact matched text and regex patterns.
- `translations/` — English and Ukrainian strings for configuration, entities, and triggers.
- `manifest.json` — Home Assistant manifest pointing to this repo.

### How it works

- Each config entry builds a detector from configured region and locality regex patterns and subscribes to selected Home Assistant source entities.
- Changed source text is checked in order: IRBM, MLRS, guided bomb, ballistic, cruise, drone, then generic danger. First match wins.
- Runtime tracks active detections per source. Every usable changed source state is authoritative: danger stores a detection and any non-danger message clears that source. Binary sensors aggregate remaining detections, and the event entity records each new detection.
- Diagnostic sensors mirror the latest active aggregate detection and return to clear when no danger remains.
- Target-based triggers fire for aggregate danger or filter event-entity updates by danger type, including repeated detections.
- Source data collection stays outside this integration. The `danger/` library stays Home Assistant agnostic and logger-free.

### Parsing data

Here are a few notes on parsing data from external sources. Here are words that always have the same meaning:

- Ballistic missile are usually refered as:
  - `отрк`
  - `бр`
  - `кинджал`
  - `іскандер`
- Intermediate-range ballistic missiles are usually refered as:
  - `брсд`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [denysdovhan/ha-aerial-danger](https://github.com/denysdovhan/ha-aerial-danger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
