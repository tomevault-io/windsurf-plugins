---
trigger: always_on
description: This project is the source repository for the website https://ssa.tools/. The
---

# Project Instructions for GitHub Copilot

## Project Overview

This project is the source repository for the website https://ssa.tools/. The
website is a SvelteKit application that provides a report on the user's social
security retirement benefits. The user inputs their social security earnings
record copy / pasted from ssa.gov as well as their birthdate and optionally the
same for their spouse. The site then shows a report that gives the user insights
into their benefits. The user can choose different future earnings scenarios,
see how those affect their primary insurance amount, and finally see how
different filing dates affect their actual social security benefits.

The philosophy of the site is to provide sufficient detail so that the user can
understand all of the calculations being done, not just see the final answer.

Visualizations are inspired by Tufte's principles for visualizing quantitative
information:

- Visual representations of data must tell the truth.
- Data visualizations should be clear and concise.
- Visualizations should be designed for the intended audience.
- Maximize the proportion of ink that represents data.
- Reduce elements that do not convey data.
- Strive for high-quality design in visualizations.

However, Tufte was typically limited to static representations, while this site
embraces interactivity and dynamic data exploration.

## Technology Stack

- **Framework**: SvelteKit with file-based routing
- **Language**: TypeScript
- **Build Tool**: Vite
- **Deployment**: Vercel (using @sveltejs/adapter-vercel)
- **Testing**: Vitest for unit tests
- **Component Development**: Storybook for component documentation and testing
- **Package Manager**: npm
- **Linting & Formatting**: Biome

## Code Architecture:

- Code is in the `src/` path primarily.
  - Components are primarily in `src/lib/components/`.
  - There are utility libraries in `src/lib/`.
  - Storybook stories are in `src/stories/` for component development and
    testing.
  - Routes are in `src/routes/` following SvelteKit file-based routing.
  - Tests are in `src/test/`.
  - Worker files are in `src/lib/workers/`.
  - Filing strategy calculations are in `src/lib/strategy/`.
- External data sources:
  - Life tables from SSA actuarial data in `data/CohLifeTables_*.csv`.
  - Pre-processing scripts in `scripts/`.
- Some more important utility libraries include:
  - `constants.ts` for social security constants related to credits, earnings,
    tax rates, wage indices, bendpoints, COLAs, retirement ages, month names,
    etc. These require annual updates when SSA publishes new values.
  - `money.ts` has a utility class for managing monetary values and calculations
    with proper precision.
  - `month-time.ts` has utilities for working with months and time-related
    calculations. Many social security calculations depend only on the month,
    not the day.
  - `recipient.ts` is the entrypoint for managing recipient-related data and
    calculations.
  - `birthday.ts` manages recipient birthdays which have some properties related
    to how social security treats a person's age.
  - `earning-record.ts` manages parsing and processing earnings history from
    SSA.
  - `pia.ts` contains Primary Insurance Amount calculations using bend points.
  - `ssa-parse.ts` handles parsing of copy/pasted SSA earnings records.

## Social Security Calculation Flow

The typical calculation pipeline follows these steps:

1. **Parse earnings record**: Extract earnings data from SSA copy/paste input
2. **Index earnings**: Apply wage indexing factors to historical earnings
3. **Calculate PIA**: Determine Primary Insurance Amount using bend points
4. **Apply adjustments**: Factor in early/delayed retirement credits
5. **Spousal/survivor benefits**: Calculate dependent benefits when applicable

Key calculation classes:

- `EarningRecord` - manages earnings history and indexing
- `PrimaryInsuranceAmount` - calculates PIA using current bend points
- `MonthDate`/`MonthDuration` - handles month-based time calculations
- `Money` - handles monetary calculations with proper precision
- `Recipient` - coordinates all recipient-related calculations

## Coding Standards:

- Code should be written in TypeScript.
- Use descriptive variable and function names.
- Use proper JSDoc comments for functions and classes.
- Write unit tests for components and libraries, run them before committing.
- Use Biome for code formatting and linting. Run `npm run quality` before
  committing.
- Prefer 80 character lines when possible.
- Follow SvelteKit conventions for file-based routing and component structure.
- Use private fields with getters/setters for class validation patterns.
- All code will be reviewed by a human senior developer before committing.

## Development Workflow:

Available npm scripts:

- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run test` - Run unit tests with Vitest
- `npm run test:ui` - Run tests with UI interface
- `npm run coverage` - Generate test coverage report
- `npm run storybook` - Start Storybook for component development
- `npm run build-storybook` - Build Storybook for deployment
- `npm run lint` - Run Biome linting
- `npm run lint:fix` - Fix Biome issues automatically

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Gregable/social-security-tools](https://github.com/Gregable/social-security-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
