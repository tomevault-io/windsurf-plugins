---
trigger: always_on
description: **DSA Meets Design** is an interview prep platform that bridges the gap between DSA and Low-Level Design (LLD). Companies like Amazon, Flipkart, Razorpay, and Meesho now ask multi-part design questions even at SDE-1/fresher level — and no good prep resource exists for this format.
---

# CLAUDE.md — DSA-Meet-Design-Pilot

## Project Overview

**DSA Meets Design** is an interview prep platform that bridges the gap between DSA and Low-Level Design (LLD). Companies like Amazon, Flipkart, Razorpay, and Meesho now ask multi-part design questions even at SDE-1/fresher level — and no good prep resource exists for this format.

Each problem simulates a real LLD interview: a base requirement followed by 2-4 extensions that unlock progressively. The candidate's design either survives the new requirement or exposes why it doesn't.

**Owner:** Jatin Kaushal — SDE at Amazon India, active LinkedIn creator, runs free interview prep sessions on topmate.

## Product Vision & Roadmap

- **Current (Pilot):** Free/open-source repo with 20-25 problems. Acts as community builder and lead magnet via LinkedIn.
- **Full Launch:** 100 real company-tagged problems, cloud-based code execution, Next.js website with SEO/SSR. Monetized as premium content.
- **End Goal:** AlgoMaster-level platform for DSA + Design interview prep.
- **UX Benchmark:** LeetCode — 95% of target audience are LeetCode users, so the experience must feel familiar and intuitive.

## Tech Stack

- **Dashboard:** React + Vite + Tailwind CSS + shadcn/ui
- **Backend:** Express.js (server.js in dashboard/)
- **Languages supported:** C++17, Go, Java, Python, JavaScript — all five run through a single
  spec-driven harness (see Architecture below). The dashboard auto-detects which language runners
  are installed and disables submit for missing ones.
- **Code Execution:** Local runners under `harness/<lang>/`. Interpreted languages (Python, JS,
  Java) read `spec.yaml` + `tests/cases/*.yaml` at submit time; compiled languages (C++, Go)
  codegen a per-part runner that compiles against the user's solution. (Cloud judge planned for full launch.)
- **Future Migration:** Next.js for the production website

## Architecture — spec-driven, one contract per problem

Each problem is defined once by **`spec.yaml`** (types, function signatures, progressive parts)
and **`tests/cases/partN.yaml`** (language-agnostic test cases). There are NO per-problem,
per-language test files. To add a language you write exactly two things:

1. A generic runner at `harness/<lang>/` (runtime YAML for interpreted langs; a codegen script that
   emits a compilable runner for compiled langs — see `harness/cpp/codegen.py` and `harness/go/codegen.py`).
2. A boilerplate emitter in `scripts/gen_stubs.py` (`emit_<lang>`), then
   `python3 scripts/gen_stubs.py --all --lang <lang> --force`.

Then register the language in `dashboard/server.js` (`LANGS` + a submit branch) and in
`dashboard/src/pages/ProblemView.jsx` (`LANG_META`). Verify with `python3 scripts/stress_test.py`.

**Reference solutions** live at `solution.{cpp,go,java,py,js}` per problem and must pass every case.

## Project Structure

```
DSA-Meet-Design-Pilot/
├── problems/
│   ├── tier1-foundation/       # Foundation-level problems
│   │   └── XXX-problem-name/
│   │       ├── README.md       # Problem statement (LeetCode tone)
│   │       ├── DESIGN.md       # Why this pattern, what breaks without it
│   │       ├── AI_REVIEW_PROMPT.md        # Tailored Claude review prompt
│   │       ├── spec.yaml                  # Interface contract — drives ALL languages
│   │       ├── solution.{cpp,go,java,py,js}   # Reference solution per language
│   │       ├── boilerplate/<lang>/partN/  # interview / guided / learning stubs
│   │       └── tests/cases/partN.yaml     # Language-agnostic test cases
│   └── tier2-intermediate/     # Intermediate-level problems
├── harness/                    # ONE generic runner per language (spec-driven)
│   ├── cpp/codegen.py          # compiled langs: generate a runner per part
│   ├── go/codegen.py
│   ├── java/Runner.java        # interpreted langs: read spec + cases at runtime
│   ├── python/runner.py
│   └── javascript/runner.js
├── patterns/                   # Design pattern primers (GFG tone)
├── docs/_data/problems.yml     # Problem registry
├── dashboard/                  # React + Express dashboard app
│   ├── server.js               # Express API server
│   └── src/                    # React frontend
├── scripts/
│   ├── gen_stubs.py            # Regenerate boilerplate from spec.yaml
│   └── stress_test.py          # Verify every reference solution × language
├── e2e/                        # Plain-English Playwright stories
├── progress.json               # Local user progress (gitignored)
└── package.json                # Root — proxies to dashboard/
```

## Commands

```bash
npm install          # Install dashboard dependencies
npm run dev          # Start dev server (dashboard at :5173, API at :3000)
npm run build        # Build dashboard for production
npm start            # Start production server
```

```bash
# Content tooling
python3 scripts/gen_stubs.py <problem-dir> --force      # regenerate boilerplate from spec.yaml
python3 scripts/gen_stubs.py --all --lang go --force    # ...for one language, every problem

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jkaus324/machine-coding-interview-questions](https://github.com/jkaus324/machine-coding-interview-questions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
