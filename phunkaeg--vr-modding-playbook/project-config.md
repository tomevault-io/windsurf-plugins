---
trigger: always_on
description: Read `CLAUDE.md` once as the paired entry point. For RE work, read
---

# Using this playbook

Read `CLAUDE.md` once as the paired entry point. For RE work, read
[the shared agent workflow](AGENT_RE_WORKFLOW.md) once, then route below.
Use `vr-re-workflow` and `re-mcp-toolkit` for selected procedures; session tool
discovery determines availability (`cheatengine`, Ghidra, optional `local-llm`, etc.).

**Read the relevant route, not the playbook end to end.**

It is a large corpus; current measured size and source counts live in
[coverage](docs/coverage.md). The retrieval layer is built so you rarely open a whole
chapter. A normal lookup is one table row plus one
linked section — a few hundred words. Reading a chapter end to end is almost always the
wrong move, and reading several is a sign you skipped the routing below.

---

## Route by what you have

| What you have | Go to | Why |
|---|---|---|
| **A symptom** — something looks or behaves wrong | [`docs/failure-atlas.md`](docs/failure-atlas.md) | Symptom → *fast discriminator* → likely cause → route. The discriminator is the point: it is chosen to be cheap. |
| **A symptom, but you want the chapter** | [`docs/symptom-index.md`](docs/symptom-index.md) | Short rows mapping what you see to the chapter that covers it |
| **A solved problem you need the recipe for** | [`docs/pattern-catalog.md`](docs/pattern-catalog.md) | 148 atomic patterns, stable IDs, five fixed fields each |
| **A new or inherited target** | [`docs/start-new-port.md`](docs/start-new-port.md) | The router. Classifies integration authority first, then sends you down the RE-owned or source-owned route |
| **Several plausible next steps** | [`docs/bottleneck-map.md`](docs/bottleneck-map.md) | The earliest uncleared dependency, in order. Its *Agent operating protocol* section is worth reading once. |
| **A question of "has anyone solved this?"** | [`docs/cross-project-index.md`](docs/cross-project-index.md) | Who solved what, and where the raw working lives |
| **A target whose engine/API you know** | `python tools/prior_art.py <engine> <api>` | Matching tracked prior art, with what was harvested and what is thin |
| **Maths you are about to write yourself** | `docs/a1`–`a5` | Working code with the test that catches the error. Rotation, pose pipeline, stereo projection, hook safety, noise floor. |
| **A term used oddly** | [`docs/glossary.md`](docs/glossary.md) | Camera, pose, stereo, lifecycle, render and evidence vocabulary |
| **"Did another project already hit this?"** | `graphify explain "<concept>" --graph cross-engine-graph/graphify-out/fleet-graph.json` | Fleet documentation with cross-project concept links; inspect current coverage |

**On that graph, in short.** Start with `explain` on a **concept name**, not `query` on a sentence:
seed matching is lexical and unstemmed, so a plain-English question lands on whatever noun happens to
collide (*"how do projects stop geometry being culled"* once seeded on a minigame, because *game* and
*flat* matched). `explain "Frustum and Culling Adjustment"` returns every project's implementation in
one hop. For `query`, use engine nouns — `frustum culling`, `viewmodel pose handoff`, `9On12`.

It tells you *where* a problem was solved — which project, which document, which named symbol — and
never *what the answer was*, so treat every hit as a lead and read the document it names.
Cross-project edges are graded `INFERRED` and carry the concept and the reason, so you can reject a
bad link on sight.

**What it will not announce.** It indexes **documentation, not code**, and it is a **snapshot**:
absence means nobody wrote it down, or wrote it after the build. Coverage is uneven — MoH-VR (39
nodes) and SoF-VR (46) joined on 2026-09-07 and have far smaller corpora than SS2VR (659), so a
thin result for a young project is a statement about its documentation. Rebuild and extension
instructions are in [`cross-engine-graph/README.md`](cross-engine-graph/README.md).

**If the first stereo image will not fuse**, skip all of the above and go straight to the
[five-minute alignment diagnosis](docs/09-d3d11-openxr-injection.md). Every project in the
fleet hit it and none recognised it first time.

## Two ID namespaces, deliberately distinct

- `CAM-001` — a **pattern**: a reusable recipe in the pattern catalog.
- `FAIL-CAM-001` — a **failure record**: a symptom row in the failure atlas.

They are never the same record. Cite the qualified form for failures. IDs are permanent and
are never recycled; a retired pattern keeps its ID and gains a status note.

**Several agents write to this repository at once, so re-read the ID ceiling immediately before
you allocate one** — not from earlier in your session, and not from a number you were told.

```bash
grep -o "FAIL-CAM-[0-9]*" docs/failure-atlas.md | sort -u | tail -1
```

`git pull` does not protect you here: fleet agents commit straight into this working copy rather
than through the remote, so a colliding row can appear between your first read and your write with
no divergence to fetch. `tools/verify.py` catches the duplicate — this only saves you the
renumbering. (*Measured 2026-09-05: `FAIL-RE-024` was allocated twice in one afternoon, once by a
session that had read the ceiling forty minutes earlier.*)

## Three orthogonal axes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phunkaeg/vr-modding-playbook](https://github.com/phunkaeg/vr-modding-playbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
