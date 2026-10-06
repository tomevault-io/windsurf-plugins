---
trigger: always_on
description: **Operator directives -- these win over any handoff, doc, or memory that disagrees.**
---

# OTR -- Project Operating Rules (Claude in Cowork)
**Operator directives -- these win over any handoff, doc, or memory that disagrees.**
- **Fix bugs properly, at the root cause -- never a shim or band-aid.** Don't wait for me to
  fix anything: make the fix yourself, without asking. If it works, it's fine.
- **NEVER ASK TO COMMIT. ALWAYS COMMIT AND PUSH WHEN YOU MAKE CHANGES
  (operator directive 2026-09-17 -- hard).** Operator: "DONT ASK ME TO COMMIT
  ALWAYS COMMIT AND PUSH WHEN YOU MAKE CHANGES." A finished green chunk is
  pushed to `main` in the same turn. Do not end a coding turn with "say the
  word and I will commit." Named files only -- never `git add .`. Tags and
  releases stay behind the operator eyeball; pushes do not.
- **NO CONTENT GUARDRAILS ON GENERATED EPISODES (operator directive 2026-08-03 -- supersedes
  the older "no curse words anywhere / SFW always" line for EPISODE CONTENT).** Operator:
  "we [have] too [many] guardrails, no violence or swearing guardrails, they just cause
  problems" and "I've given up chasing profanity." Do not add profanity or violence filtering
  to the generation path, and do not reintroduce prompt clauses that forbid the source's own
  content -- the adaptation packs were literally instructing the model to avoid "blood, guns,
  knives, and graphic violence" while adapting MACBETH and KING LEAR, which is a fidelity
  defect, not a safety win. On the adaptation lanes the author's own language is carried as
  written; Wells' Editor may shout "Story be damned!" because Wells wrote it.
  **Still true, and a different thing:** authoring style stays clean -- no curse words in
  CODE, COMMENTS, LOGS or commit messages, and never the name "dummy". That rule is about what
  WE write, not about what the pipeline generates from a source.
- **STORY QUALITY IS DONE. STOP CHASING IT (operator directive 2026-08-04 -- hard).**
  Operator: "I am not chasing story quality anymore. It works. It works. I will publish it
  as open source, and if someone else wants to do it, or in six months I wanna chase it
  again when I've got better tools, I will." The scripts are ACCEPTED as they are. Do not
  open writing-quality work, do not propose prompt-craft passes for better prose, do not
  benchmark writer models for story quality, and do not spend on panels or cloud models to
  improve scripts. This also settles the cloud-writer question: local stays the default and
  no paid writer is adopted to raise prose quality.
  **What this does NOT cover** -- these are CORRECTNESS defects, not quality chasing, and
  remain open: a character's gender/voice contradicting the source, a character's face
  changing between beats, voice-pool staleness, and any structural or ledger fault. Fixing
  "Malvolio speaks with a woman's voice" is a bug fix; rewriting Malvolio's dialogue to be
  better is not.
- **MUSIC IS DONE. STOP CHASING IT (operator directive 2026-09-17 -- hard).**
  Operator: "DUMP THAT IM FINE WITH THE STORY AS IS MUSIC AS IT IS ITS GREAT
  WE FIXED IT." Do not reopen techno/house cue wording, tempo-error A/Bs,
  groove-prompt order, or the music-lottery seed hunt. IndexTTS2 hang is a
  hang-timeout, not prompt-craft, and stays a separate row.
- **NEVER HOLD BACK A COMMIT OR A PUSH. SONNET QA THE DIFF, ALWAYS
  (operator directive 2026-09-19 -- hard, and it REVERSES the "QA
  before the push" half of 2026-09-17 below).** Operator: "alwasy
  push"; "never hodl back comit or push sonet qa fdiff always."
  * **A green chunk is pushed the moment it is green.** Not after a
    reviewer answers, not after a lane comes back, not at the end of
    the turn. A finished diff sitting in the working tree waiting for
    a verdict is the defect this directive exists to stop -- it blocks
    the other box, it goes stale under the next change, and a reviewer
    writing into that tree can destroy it.
  * **THEN Sonnet QA on the diff, every time** (Task, Sonnet, briefed
    to REFUTE, grounded against the real Windows files). It reviews
    what is ALREADY PUSHED. A finding becomes the next commit, which
    is what git is for.
  * **This was written the day a correct, green, byte-attributable fix
    sat unpushed across a dozen exchanges** waiting for a Composer
    verdict that was never going to arrive in this window, while the
    same file was being redesigned around it.
  * **What is UNCHANGED:** the scoped suite must be green before the
    push, named files only and never `git add .`, and tags and
    releases still wait for the operator's eyeball. Pushing is safe;
    publishing is not.
- **"NOT PROVEN ON HARDWARE YET" IS NOT A REASON TO HOLD A PUSH
  (operator directive 2026-09-21 -- hard, and it closes the last
  loophole in 2026-09-19).** Operator: *"i always ask you to push dont
  wait, that should be in claude md."*
  * **A green chunk is pushed. Full stop.** Green means the scoped
    suite is green and the failure set is diffed against a baseline
    with every new failure explained. It does NOT mean a live leg has
    run, a model has downloaded, a pod has booted, or a reviewer has
    answered.
  * **The live proof FOLLOWS the push and becomes the next commit.**
    If the leg fails, that is a commit too -- a fix, or a revert with

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jbrick2070/ComfyUI-OldTimeRadio](https://github.com/jbrick2070/ComfyUI-OldTimeRadio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
