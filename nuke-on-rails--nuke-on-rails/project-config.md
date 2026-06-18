---
trigger: always_on
description: Full health and security audit for Rails apps, the review a principal engineer would do. Runs rubycritic, Brakeman, bundler-audit and ruby_audit, triages every finding with the LLM as judge, applies OWASP Top 10 lenses for what scanners miss, and returns one impact-ranked action plan. Use for a Rails project audit, a security and health check, "nuke on rails", or a deep review of a vibecoded Rails app.
---


# Nuke on Rails

A full project health audit for Rails apps: *what to refactor, what's vulnerable, and in what order to attack it*. Three deterministic engines do the scanning; you are the judge. The output is **one single list, prioritized by impact**, never tool sections stapled together.

**Respond in the user's language.** Write the step announcements and the final report in whatever language the user writes in. (These instructions and the lenses are authored in English, but the audit you produce should speak to whoever ran it.)

This audit takes minutes, so announce each step as you begin it and the user sees progress. The examples below show the format in English; translate them to the user's language: `🔭 Step 1/5 — installing & running engines…`, `🎯 Step 2 — picking hotspots (churn × complexity)…`, `⚖️ Step 3 — triaging N findings & applying lenses…`, `📋 Step 5 — building the impact-ranked report…`. One short line per step; the visible tool calls fill in the rest.

## Step 0 — Detect the project

- **Rails app** (`config/application.rb` exists, or the `rails` gem is in the Gemfile): run the full audit.
- **Plain Ruby** (Gemfile or gemspec, no Rails): graceful degradation — rubycritic and bundler-audit run, Brakeman is skipped. Say so explicitly in the report header.
- **Neither**: stop and tell the user this skill audits Ruby/Rails projects.

**Ruby version trap.** A `.ruby-version` (or `Gemfile`-pinned Ruby) often names a version not installed on this machine — common in unfamiliar or AI-generated repos, and it makes rubycritic and brakeman abort before producing output. The engines do static analysis and don't need the app's exact Ruby. If the pinned version is missing, run them under an available Ruby (e.g. set a temporary `.ruby-version`, or invoke via `rbenv local <installed>` / a system Ruby) rather than failing — and note the substitution in the report. Never install the pinned Ruby just to satisfy the lock.

## Step 1 — Install and run the engines

Zero-dependency principle: the skill brings its own tools. Never touch the app's Gemfile.

```sh
gem list rubycritic -i || gem install rubycritic
gem list brakeman -i || gem install brakeman          # Rails only
gem list bundler-audit -i || gem install bundler-audit
gem list ruby_audit -i || gem install ruby_audit
```

Then run them all and capture machine-readable output (commands validated on a real Rails 8 app):

```sh
OUT=$(mktemp -d)
rubycritic app --format json --no-browser --path "$OUT/rubycritic"   # --path keeps output files out of the audited repo
brakeman --format json --quiet -o "$OUT/brakeman.json"               # Rails only
bundle-audit update                                                  # update the db in a separate step:
bundle-audit check --format json > "$OUT/bundler-audit.json"         # --update mixed in pollutes the JSON on stdout
(cd "$OUT" && ruby-audit check)                                      # run OUTSIDE the app dir — inside a bundled
                                                                     # project the executable fails to load (Ruby 3.4+);
                                                                     # make sure the app's Ruby version is still active
```

If an engine fails, degrade gracefully: report which engine was skipped and why, and continue with the others.

## Step 2 — Decide where to spend context (churn × complexity)

Use rubycritic's churn × complexity data to pick the hotspots: files that are both complex **and** change often. Those are the files you read deeply. Never review the codebase uniformly — a complex file nobody touches is a lower priority than a moderately complex file that changes every week.

**Churn needs git history — verify it before trusting the quadrant.** rubycritic derives churn from `git log`, so a shallow clone (`--depth 1`) or a freshly-initialized repo gives every file the same churn (1 or 0) and the quadrant silently collapses to noise — common exactly in the unfamiliar-repo case this skill targets. If churn is uniform across modules, say so and fall back to ranking by complexity and smell density (rubycritic wraps Reek/Flay/Flog — use the per-module `smells` and `complexity`, not just the score). When possible, unshallow first (`git fetch --unshallow`).

## Step 3 — Triage security findings (you are the judge, not the scanner)

For every Brakeman warning and every bundler-audit CVE:

1. **Read the actual code path.** Confirm the finding is reachable in this app, or kill it as a false positive.
2. **Explain the exploit path** for confirmed findings: who can trigger it, from where, with what impact.
3. **Adversarial verification before it enters the report.** Security findings are held to a higher bar than quality findings: a weak quality finding gets ignored; a false security claim burns trust. If you cannot articulate the exploit path, downgrade it to "theoretical" — never present it as confirmed.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nuke-on-rails/nuke-on-rails](https://github.com/nuke-on-rails/nuke-on-rails) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
