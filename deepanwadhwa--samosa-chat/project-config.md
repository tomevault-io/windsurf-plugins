---
trigger: always_on
description: Local browser + terminal chat app around Qwen3.6-35B-A3B. CPU-only C engine,
---

# Samosa Chat — agent guide

Local browser + terminal chat app around Qwen3.6-35B-A3B. CPU-only C engine,
expert streaming from disk, no framework, no build system, no dependencies.

## Start here

**Working on a GitHub issue (#1–#5)? Read [docs/ISSUE_TASKS.md](docs/ISSUE_TASKS.md)
first — including its Working agreement — then your issue's spec.** The issues
themselves are one-line titles; the specs are where the work is defined.

| Issue | Spec | Branch |
|---|---|---|
| #1 Linux | [docs/TASKS_LINUX.md](docs/TASKS_LINUX.md) | `issue-1-linux` |
| #2 Windows (Docker) | [docs/TASKS_WINDOWS.md](docs/TASKS_WINDOWS.md) | `issue-2-windows-docker` |
| #3 Vision | [docs/TASKS_VISION.md](docs/TASKS_VISION.md) | `issue-3-vision` |
| #4 Internet | [docs/TASKS_INTERNET.md](docs/TASKS_INTERNET.md) | `issue-4-internet` |
| #5 Documents | [docs/TASKS_DOCUMENTS.md](docs/TASKS_DOCUMENTS.md) | `issue-5-documents` |
| — Hardware/perf | [docs/TASKS_HARDWARE.md](docs/TASKS_HARDWARE.md) | cross-cutting |
| — Apple Silicon experiments | [docs/TASKS_EXPERIMENTS.md](docs/TASKS_EXPERIMENTS.md) | macOS-only |

User-facing docs: [INSTALL](docs/INSTALL.md) · [USAGE](docs/USAGE.md) · [PERFORMANCE](docs/PERFORMANCE.md) · [DESIGN](docs/DESIGN.md) · [ROADMAP](docs/ROADMAP.md)

App-level plan: [docs/APP_TASKS.md](docs/APP_TASKS.md) (phases A2/A3 are
superseded in part — see ISSUE_TASKS.md). Serve API: [docs/SERVE_API.md](docs/SERVE_API.md).

## Open defects

**J11 (FIX LANDED on `issue-7-jobs`, real-Downloads re-dogfood pending, #7)**
— the shipped find-job path (`jobs_find`, `src/samosa_gateway.c`) failed the
owner's Titli dogfood a second time (2026-07-23) from five compounding defects
rooted in one inversion — C hardcoded the intelligence the JF spec assigned to
the model: a tokenizer that treated the **letter "t" as a delimiter** (so
"Titli"/"cat" could never become search terms; a wallpaper zip outscored a
file named after the cat), a hardcoded "What is your pet's name?" fallback,
answer-restarts that discarded the whole run (violating JF.3), state that
persisted only goal+folder, and PDFs read text-layer-only while the prompt
steered away from `doc.read`'s OCR cascade. Zero find-path test coverage let
all five ship. **Phase JI rebuilt the find path** (model triages → skim index
→ classify → verify → structured `finish()`); the condemned code
(`candidate_score`, `build_candidates`, the canned question, the goal-restart
path) is demolished — verified absent. JI.0–JI.8 pass offline
(`make compiled-gateway-test` + `make jobs-test`, both `-Werror`), and
E-JI1/E-JI2 ran on real Ornith against a **50-file synthetic fixture** with
committed SSE evidence. **Remaining to fully close:** the real-Downloads
re-dogfood that originally exposed J11 — the true "works" gate per the
non-negotiables (synthetic fixtures are not the owner's real folder). Evidence:
[docs/regressions/jobs/titli-find-2026-07-23.md](docs/regressions/jobs/titli-find-2026-07-23.md)
(root cause) and
[docs/regressions/jobs/e-ji1-2026-07-23/report.md](docs/regressions/jobs/e-ji1-2026-07-23/report.md)
(real-model gate);
rebuild spec: [docs/TASKS_JOBS_INTELLIGENCE.md](docs/TASKS_JOBS_INTELLIGENCE.md)
**Phase JI** (supersedes the shipped implementation, enforces the JF spec's
model-decides design).

**G9 (OPEN, #1)** — the cgroup pressure signal counts page cache and
over-triggers. `linux_memory_pressure_level()` uses `memory.current/memory.max`,
but cgroup v2's `memory.current` includes the page cache the engine fills by
streaming `experts.bin`. Measured on a **2-token** run: ratio 0.85 fired WARN
while real usage (`anon`) was 0.56 — the engine dumped 323 MB of its own expert
cache to relieve pressure that did not exist (2% hit rate, 1803 evictions). Lives
inside G2, the port's highest-risk change. Evidence:
[docs/regressions/linux/real-model-run.md](docs/regressions/linux/real-model-run.md);
spec: [docs/TASKS_LINUX.md](docs/TASKS_LINUX.md) **G9**.

**G10 (OPEN)** — the AVX2/AVX512 kernels are **dead code in every shipped
x86 build**. `install.sh` and the `Dockerfile` compile with `-O3` and no
`-march`, so `__AVX2__` is undefined and [kernels.h](src/kernels.h)'s scalar
remainder does 100% of the work. Measured **7.6× slower** (17.09 → 2.26 GFLOP/s).
`-march=native` is *not* the fix — one image serves many CPUs — so runtime
`cpuid` dispatch is required. **Cannot be validated on the reference Mac**: an
amd64 container there has no AVX2/AVX512/SSE4.2. Needs real x86 hardware.
Spec: [docs/TASKS_HARDWARE.md](docs/TASKS_HARDWARE.md) **H2**; evidence:
[docs/regressions/linux/x86-dispatch.md](docs/regressions/linux/x86-dispatch.md).

**Published-claim defect (H1) — FIXED in `c829f20`; this entry was stale and
was re-reported as open on 2026-07-28. Verify the files before reopening it.**
[dist/MODEL_CARD.md](dist/MODEL_CARD.md) now states the correct physics — "SSD
reads do **not** consume drive endurance. Flash endurance is rated in TBW
(Terabytes *Written*, JEDEC JESD218) and DWPD (Drive *Writes* Per Day);
program/erase cycles wear NAND, reads do not" — and [README.md](README.md)
carries no wear claim at all, only that Qwen performance depends on SSD
throughput. Nothing tells users to avoid thinking mode to protect hardware.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deepanwadhwa/samosa-chat](https://github.com/deepanwadhwa/samosa-chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
