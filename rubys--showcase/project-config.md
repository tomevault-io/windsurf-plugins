---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture Documentation

For understanding the overall system architecture, deployment model, and operational patterns, see [ARCHITECTURE.md](ARCHITECTURE.md). Review this document when:

- Working on deployment, multi-tenancy, or Navigator-related features
- Understanding how the application runs in production (Fly.io, Tigris storage, global distribution)
- Investigating performance, scaling, or infrastructure concerns
- Making changes to configuration management, CGI scripts, or lifecycle hooks
- Understanding the four-component system: Rails app, administration, Navigator reverse proxy, and glue scripts

## Rails Configuration

The application runs on Rails 8.0.2 with Ruby 4.0.1 and full Rails 8.0 configuration defaults (`config.load_defaults 8.0`).

### Rails 8.0 Migration Completed

All SQL reserved word compatibility issues have been resolved:

1. **Ordered Scopes**: Models with `order` columns use an `ordered` scope with `arel_table[:order]`:
   - `Dance.ordered`, `Category.ordered`, `Billable.ordered`, `Song.ordered`, etc.

2. **By Name Scopes**: Models with `name` columns use a `by_name` scope with `arel_table[:name]`:
   - `Person.by_name`, `Studio.by_name`, `Dance.by_name`

3. **All migrations complete**: The codebase is now fully compatible with Rails 8.0's SQL reserved word quoting requirements.

Future work could consider renaming these columns to non-reserved words for cleaner code.

## Project Overview

This is a Rails 8 application for managing ballroom dance showcase events. It handles event scheduling, heat management, scoring, and participant tracking across multiple locations and competitions.

Models are split into two categories: **Base models** support ballroom dance event management (ages, billables, categories, dances, entries, events, formations, heats, judges, people, scores, solos, studios, tables, etc.), while **Admin models** support system administration and multi-tenancy (locales, locations, regions, showcases, users).

### Frontend Architecture

This application uses **import maps** for JavaScript module management (not Webpack or esbuild). When adding JavaScript functionality:

- **Prefer Stimulus controllers** over inline scripts in views
- Stimulus controllers are located in `app/javascript/controllers/`
- Import maps configuration is in `config/importmap.rb`
- Avoid `<script>` tags with inline JavaScript in ERB templates unless absolutely necessary

The frontend primarily uses **ERB templates** with Turbo Drive for enhanced navigation and **Stimulus controllers** for interactive behaviors. This approach handles all event preparation, administration, participant management, and post-event publishing.

#### Single-Page Application (SPA) for Live Event Features
For features used heavily **during live events** (where offline capability and real-time updates are critical), the application uses a **Web Components-based SPA** approach:

**Current SPA Implementation: Judge Scoring Interface**
The judge scoring interface has been reimplemented using Web Components to provide offline-first functionality and real-time updates during events:

**Core Components:**
- `heat-page.js` - Main container managing heat navigation and state
- `heat-solo.js` - Solo heat rendering with formations and scoring
- `heat-rank.js` - Finals with drag-and-drop ranking
- `heat-table.js` - Standard heat table with radio/checkbox scoring
- `heat-cards.js` - Card-based drag-and-drop scoring interface
- `heat-header.js` - Heat details (number, dance, slot display)
- `heat-info-box.js` - Contextual help and instructions

**Data Management:**
- `HeatDataManager` - IndexedDB-based offline storage with automatic sync
- Queues scores in IndexedDB when offline, uploads when connectivity returns
- Fetches heat data from JSON endpoints (`/scores/:judge_id/heats.json`)
- ActionCable integration for live score updates during events

**Key Features:**
- **Offline-first**: Judges can score heats without network connectivity
- **Progressive enhancement**: Falls back to traditional views when JavaScript unavailable
- **Behavioral parity**: SPA matches Rails view behavior exactly (verified by comprehensive tests)
- **Real-time sync**: Live updates via ActionCable when other judges score
- **Touch-friendly**: Optimized for tablet use (iPads) during live events

**Implementation Status:**
The ERB-based judge scoring views remain in place and will only be removed once the Web Components version is proven by actual usage during live events. Both implementations coexist, with the traditional views serving as a fallback.

**When to use SPA vs Traditional:**
- **Use SPA for live event features** that require offline capability, real-time updates, or heavy interaction during events (e.g., judge scoring, heat management)
- **Use traditional ERB/Stimulus for everything else**: event preparation, administration, participant management, reporting, publishing
- The majority of the application uses and will continue to use ERB templates with Stimulus controllers

## Common Development Commands

### Running the Application

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rubys/showcase](https://github.com/rubys/showcase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
