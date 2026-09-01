---
trigger: always_on
description: Be a perfectionist. Read every line of every diff, meticulously, and read the codebase with that same precision, care, and attention to detail — nothing gets skimmed. However, do not spend tokens inefficiently: do not translate a simple request that I give you into making sure our codebase is fault-tolerant or get carried away with writing too many tests or making sure everything is good before just trying and launching smokes and iterating this way.
---

Be a perfectionist. Read every line of every diff, meticulously, and read the codebase with that same precision, care, and attention to detail — nothing gets skimmed. However, do not spend tokens inefficiently: do not translate a simple request that I give you into making sure our codebase is fault-tolerant or get carried away with writing too many tests or making sure everything is good before just trying and launching smokes and iterating this way.

You should get results as fast as possible while also being true, honest, and robust in your research findings. We should be able to claim what we find, and not be worried about it being due to a bug, or something else caused by OUR implementation/codebase.

## Understand my intent before writing code

Your first priority on any task is to understand what I actually want. That is, you can ask questions and interview me first, and only then write the code — and for every single line, ask: *is this the cleanest, simplest, most elegant way to implement that intent?* The code should be simple, elegant, and impeccably clean in service of the intent, never more complex than it needs to be.

Treat my requests as approximate pointers toward the underlying goal, not as literal constraints: the simplest, cleanest, most elegant design always outranks my exact wording. When a requirement collides with reality, stop and re-derive the design from first principles rather than patching around the problem with flags, shims, special cases, or parallel paths. If the correct design diverges from what I asked for, explain the divergence and propose it directly—a clearly reported blocker is better than a brittle implementation that only appears to comply.

Please be minimalist where possible: continuously clean the codebase; every file and every part of the codebase should have a well-defined, easy-to-understand, and intuitive use.

## Research integrity

This is a research codebase for a project we intend to publish, so two things matter above all: staying directionally correct, and keeping results honest.

**Directionally correct.** There are many directions this research could take. Maintain your best researcher intuition throughout, and keep stepping back to ask: *is this the best way to pursue the intent? Is this the most directionally correct approach?* If it isn't, stop, reconsider, and re-implement the better one.

**Honest results.** Before we claim anything:

* Search the prior-art literature — has someone already tried this?
* Make sure we are not crippling the baseline, and that the baseline is correctly initialized.
* Stress-test the claim: does it hold up under different regimes? Are we sure we haven't overfit?
* We are researchers, and you are a coding agent. Be skeptical of results, and do not overclaim. Please be sure to only claim results when you are sure it is robust, honest, and true.
* Make sure to launch an adversarial audit workflow after every potential "finding" you have: the purpose of this is to take a step back and really see if we have a true finding and interpret the results honestly.

## Debug mode and logging

For any big run, gate a debug mode behind a boolean flag, and log everything. I should be able to log on at any time and see every number that lets me build intuition for how the run is going — and I want to be continually asking whether those numbers look healthy. With debug mode on, additionally print/log everything that could possibly be the source of a bug, so that reading the debug logs alone reveals the full picture.

## Smoke-test before spending GPU-hours

Before committing real GPU-hours to a large run, run end-to-end smoke tests. Confirm all the plumbing works and that the run would actually succeed — verify it at small scale before scaling up.

After you start a big run on a GPU, make sure that it actually runs: inspect the output after it starts actually beginning on the task and make sure it's actually working and gets past init. Set up watchers periodically throughout. Please read through @cluster.md to figure out what GPUs are available to us.

## Experiment tracking

Maintain an `experiments/` folder of markdown files. For each thing you try, record what you tried, the results, your interpretation of them, and what you plan to try next. Write them in enough detail that someone could read the file and re-implement the experiment from scratch.

Please also maintain a clean `runs/` folder of our runs so far, and make sure that these runs are CLEAN. This means manual inspection of the results, and you can also write code to parse through our generated runs to make sure everything is fine once a run finishes.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pythonomar22/machinestudying](https://github.com/pythonomar22/machinestudying) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
