---
trigger: always_on
description: A workbench for generating Meta/Facebook image ads with **Luma's uni-1** model and uploading them to a Meta ad account via the **Meta Ads CLI**. The whole project is built around uni-1 specifically — its reference-grounded generation, brand-fidelity behavior, and aspect-ratio support are baked into the prompt library and the workflow. The model lock is enforced at the tooling layer, not left to humans to remember.
---

# Uni1 — uni-1 image ads for Meta

A workbench for generating Meta/Facebook image ads with **Luma's uni-1** model and uploading them to a Meta ad account via the **Meta Ads CLI**. The whole project is built around uni-1 specifically — its reference-grounded generation, brand-fidelity behavior, and aspect-ratio support are baked into the prompt library and the workflow. The model lock is enforced at the tooling layer, not left to humans to remember.

> **If a user asks you to "install this" / "set this up" / "help me get started" on a fresh clone:** the install flow is two scripts plus walking them through `.env`. Run them in order:
>
> 1. **`./install.sh`** — checks Python 3.12+ and uv, installs the `meta-ads` CLI if missing, symlinks the skill into `~/.claude/skills/`, scaffolds `.env` from `.env.example`. Idempotent. Will tell the user exactly what failed if a prereq is missing.
> 2. **Walk the user through `.env`** — they need to provide `LUMA_API_KEY`, `ACCESS_TOKEN`, and `AD_ACCOUNT_ID`. You can't generate these for them. The Meta token in particular requires a multi-step UI flow in Meta Business Suite — see "Required env vars" + "Meta token setup" in `README.md` for the exact steps and scope list.
> 3. **`./verify.sh`** — confirms `.env` is filled, the skill is symlinked, `meta auth status` returns Authenticated, and the Luma API key works. Exits non-zero with a clear error if anything is wrong.
> 4. **Tell the user to restart Claude Code.** Skills load at session start; the new symlink won't be picked up by the current session. They need to open a fresh Claude Code session in this repo, then they can say "make a uni-1 image ad" to use the skill.

If you're an agent helping someone set this up, **read the rest of this file too** for context on credentials, hard rules, and common pitfalls.

## Repository layout

```
.
├── .env                            # local secrets, gitignored — see "Required env vars"
├── .env.example                    # template; copy to .env and fill in
├── .gitignore
├── CLAUDE.md                       # ← you are here (agent-facing setup notes)
├── README.md                       # human-facing intro + quickstart
├── install.sh                      # symlinks ./skills/* → ~/.claude/skills/
├── verify.sh                       # confirms env + skills + Meta auth + Luma API are working
├── skills/                         # Claude Code skills (versioned in this repo)
│   ├── uni1-image-ad/              # template-USING skill — generate + upload Meta ads
│   │   ├── SKILL.md
│   │   ├── scripts/                # generate_image.py, top_spending_ads.py, create_text_variant_creative.py
│   │   ├── references/             # prompt-library.md, ad-copy-frameworks.md, meta-cli-flags.md
│   │   └── state.json              # per-account cache (gitignored once populated)
│   └── image-ad-clone/             # template-CREATING skill — reverse-engineer ads into reusable prompts
│       ├── SKILL.md
│       └── references/template-format.md
├── docs/luma-agents-api/           # offline Luma API reference
│   ├── quickstart.md
│   ├── model.md                    # uni-1 capabilities, params, output specs
│   ├── image-generation.md         # type: "image" — every parameter
│   ├── image-editing.md            # type: "image_edit" — source + image_ref
│   ├── rate-limits.md              # 30 RPM / 10 concurrent jobs, headers, backoff
│   ├── error-handling.md           # every status code + failure_code
│   └── faq.md
├── Ad References/                  # swipe-file of real ads (used to seed prompts)
├── iterations/                     # validation runs that built the prompt library
│   ├── run_round.py                # parallel batch runner — fires N generations at once
│   ├── r1/                         # round-1 reproductions of Ad References
│   └── ag1-v2/                     # AG1 example fills, chrome-stripped (PNGs gitignored)
└── generated/                      # output PNGs + runs.jsonl audit log (gitignored)
```

`./skills/<skill-name>/` are the canonical copies. After clone, run `./install.sh` to symlink them into `~/.claude/skills/` so Claude Code picks them up. Editing the in-repo copy is the same as editing the live skill.

**Two skills, complementary:**
- `uni1-image-ad` is the **template-using** skill: trigger it to generate a uni-1 image and upload it as a paused Meta ad, optionally filling in a template from the prompt library.
- `image-ad-clone` is the **template-creating** skill: trigger it with a reference ad image and it reverse-engineers it into a parameterizable prompt that gets appended to the prompt library, ready for `uni1-image-ad` to use later.

## The Luma API at a glance

The full reference is in [`docs/luma-agents-api/`](./docs/luma-agents-api/). Read it when you need detail. The 30-second version:

- **Base URL:** `https://agents.lumalabs.ai/v1`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [krusemediallc/uni1-image-ad](https://github.com/krusemediallc/uni1-image-ad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
