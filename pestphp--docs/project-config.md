---
trigger: always_on
description: The Agent plugin gives AI coding agents a single command to verify that a change actually works — running inside your full test suite, and, with the Browser plugin installed, driving a real browser too.
---


# Agent

**Source code**: [github.com/pestphp/pest-plugin-agent](https://github.com/pestphp/pest-plugin-agent)

AI coding agents excel at writing code, yet they often have no way to know whether that code actually works. After editing a controller, a Livewire component, a Blade template, or a bit of CSS, an agent cannot *see* the result — so it guesses, then moves on.

<iframe width="100%" height="315" src="https://www.youtube.com/embed/Z6878g7OSMo" title="10x better than vercel agent-browser (pest 5, day 2/6)" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

Thankfully, the Agent plugin closes that loop. It gives your agent a single command to run a one-off verification against your application:

```bash
./vendor/bin/pest --agent='$user = \App\Models\User::factory()->create(); $this->actingAs($user)->get("/dashboard")->assertOk();'
```

Your agent receives a definitive pass or fail instead of a hopeful guess, with the full power of Pest at its disposal. Everything your test suite can do is available: factories, the database, mail and notification fakes, authentication, and expectations — all in the same probe.

> **Note:** The snippet is wrapped in single quotes. Single quotes tell the shell to pass everything through to PHP untouched, so `$variables` and `\App` class names need no escaping — with double outer quotes, your shell would interpolate `$user` to an empty string before PHP ever sees it. Within the snippet, you may use double quotes for PHP string literals.

The plugin is not specific to the browser. It verifies *any* code your test suite can reach — backend behavior, queued jobs, mail, notifications, and more — with nothing beyond a standard Pest install. That said, it truly shines once the [Browser Testing](/docs/browser-testing) plugin is installed, because your agent may then drive a real browser *and* assert the side effects it triggers, all in a single command.

## Getting Started

To get started with the Agent plugin, require it via Composer:

```bash
composer require pestphp/pest-plugin-agent --dev
```

The plugin adds the `--agent` option to Pest. This alone is enough to verify backend behavior. However, to also verify frontend behavior — visiting pages, screenshots, clicks, responsive checks — you should install the [Browser Testing](/docs/browser-testing) plugin as well. It is optional, yet highly recommended, as it is where the Agent plugin truly shines:

```bash
composer require pestphp/pest-plugin-browser --dev

npm install playwright@latest
npx playwright install
```

Finally, teach your AI agent how to use the plugin by installing its guidelines and skills through [Laravel Boost](https://github.com/laravel/boost):

```bash
php artisan boost:install
```

When prompted for third-party AI guidelines and skills, select `pestphp/pest-plugin-agent`. This installs the guidelines and the `pest-plugin-agent` skill, so your agent knows exactly when and how to reach for the `--agent` command.

That's it. Your agent may now verify backend behavior, frontend behavior, or both, from a single command.

## How It Works

When you run `./vendor/bin/pest --agent='<code>'`, Pest writes your snippet into a temporary test file that resembles the following:

```php
<?php

it('verify', function () {
    // your snippet goes here...
});
```

The file runs with your project's real Pest configuration: the classes and traits you registered in `tests/Pest.php` via `uses()` — including `RefreshDatabase` — are applied to the generated test automatically, so the snippet behaves exactly like a test living in your test suite. Once the run finishes, the temporary file is removed.

There are a few details worth keeping in mind:

- **Every class must be fully qualified.** The generated file contains no `use` imports, so your snippet should reference `\App\Models\User` rather than `User`.
- **Snippets may not be empty.** Passing `--agent` without a value, or with an empty one, will abort the run with an error instead of silently passing.
- **Directory-scoped hooks do not apply.** Classes and traits from `uses(...)->in('Feature')` carry over to the generated test; however, `beforeEach()` hooks attached to a directory are bound to that path and will not run for the snippet. If required setup lives in such a hook, your agent should inline it at the top of the snippet.

## Why Agent?

A new category of tooling has emerged to give agents "eyes" on the browser — Vercel's [agent-browser](https://github.com/vercel-labs/agent-browser) being a prominent example. These tools drive a headless Chromium instance and let an agent click, type, and screenshot its way through your app.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pestphp/docs](https://github.com/pestphp/docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
