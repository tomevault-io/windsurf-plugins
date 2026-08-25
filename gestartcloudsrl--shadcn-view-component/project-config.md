---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Rails engine gem that ports [shadcn/ui](https://ui.shadcn.com) to ViewComponent
**1:1**: same part names, variants, Tailwind classes and `data-slot` attributes.
Radix UI's behaviour is reimplemented in Stimulus — no React, no npm dependency
at runtime.

Two constraints decide everything here, and between them they settle most
questions without having to ask.

**It is 1:1 with upstream.** When upstream and idiomatic Rails disagree, upstream
wins on *markup* and Rails wins on *API*.

**It is a library to be published, not an application.** The code runs inside
someone else's Rails app, against their models, their Tailwind config, their
Turbo, their CSP — none of which can be inspected from here. So when a choice is
borderline, take the one that survives a host you cannot see. That is what
already decided: `Shadcn::` namespacing (a host's `Card` model would collide),
caller attributes beating component defaults, the gem owning
`[data-slot][hidden]` rather than trusting Tailwind preflight to be loaded, a
cache sized for a real page rather than the library's own defaults, and
`Gemfile.lock` staying uncommitted so the matrix resolves against a range.
Ambiguity resolves toward "has to hold up in an app I will never see", not
toward "good enough here".

**Before deciding *how* to build something, open upstream's own example and read
what it renders — its shape, not its looks.** Not the vendored copy — the live component on
[ui.shadcn.com](https://ui.shadcn.com), in the right variant tab. The vendored
sources say what was true when they were copied and answer "what does this
version do"; they never answer "what does upstream do today", and neither
question is "what shape did upstream choose". Inspect the rendered DOM: roles,
`data-slot` names, which element owns which ARIA attribute.

The searchable select is the worked example. Two candidate shapes were designed
here and argued between, and upstream's answer was neither: its popover is
`role="dialog"` holding a *separate* listbox, which dissolves the question
instead of answering it. Both candidates had been derived, correctly, from the
parts already measured — and the part never looked at was the one that decided
it. The dropdown's wrap-around went the same way: three vendored files agreed
and the conclusion was still wrong, because the docs now show a different
variant first.

Reading the source is how a claim gets *checked*. Looking at the example is how
a decision gets *made*. What the example does **not** settle is appearance: the
docs now serve a different registry from the `new-york-v4` one ported here, so a
class string is checked against `vendor/shadcn/ui/` and never against a
screenshot.

That has already misled twice, both times *after* the rule was written down, so
it needs its practical half: **read the element's `className` before measuring
anything.** `cn-*`, `ring-` where the port has `border`, `rounded-lg` where it
has `rounded-md`, `data-open:` where it has `data-[state=open]:` — any of those
means you are looking at the other registry, and nothing measured afterwards is
a comparison. See [todo.md](.claude/docs/todo.md).

**Read [`.claude/docs/`](.claude/docs/README.md) before changing anything
structural** — it holds why the port is shaped the way it is, and which
alternatives were already measured and rejected.

## Commands

```sh
bin/setup                                     # bundle install + build Tailwind
bundle exec rake                              # everything (rspec)
bin/rubocop                                   # Rails omakase style; -a to autocorrect
bin/eslint                                    # the JavaScript half; --fix to autocorrect
bin/console                                   # IRB + dummy app; `render`, `slots`, `upstream`, `reload!`

bundle exec rspec spec/system                 # browser specs only (needs Chrome)
EXAMPLE_TIMEOUT=0 bundle exec rspec …         # off, for a breakpoint (default: 60s an example)
CAPYBARA_WAIT=15 bundle exec rspec …          # a busy machine; the default 5s is for an idle one
bundle exec rspec spec/system/dialog_spec.rb
bundle exec rspec spec/parity_spec.rb -e "card.tsx"

SNAPSHOTS=overwrite bundle exec rspec spec/snapshot_spec.rb   # regenerate golden HTML

bundle exec rake themes:build                 # regenerate palettes from vendor/shadcn/themes.json

cd test/dummy && bin/rails s                  # gallery at http://localhost:3000/lookbook
cd test/dummy && bin/rails tailwindcss:watch  # keep CSS fresh while editing classes
```

`tailwindcss:build` / `tailwindcss:watch` only work from `test/dummy`, not the
repo root. Without a build the gallery renders **unstyled rather than erroring**,
because the CSS is a compiled bundle.

## Where things are

```
app/components/shadcn/
  application_view_component.rb  # the whole React→Ruby mapping
  parts.rb                       # the `part` macro
  <family>.rb                    # `part` declarations for the trivial sub-components
  <family>/component.rb          # the family root
  <family>/<part>/component.rb   # parts that have behaviour

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gestartcloudsrl/shadcn_view_component](https://github.com/gestartcloudsrl/shadcn_view_component) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
