---
trigger: always_on
description: <!-- intent-skills:start -->
---

<!-- intent-skills:start -->

## Skill Loading

Before substantial work:

- Skill check: run `npx @tanstack/intent@latest list`, or use skills already listed in context.
- Skill guidance: if one local skill clearly matches the task, run `npx @tanstack/intent@latest load <package>#<skill>` and follow the returned `SKILL.md`.
- Monorepos: when working across packages, run the skill check from the workspace root and prefer the local skill for the package being changed.
- Multiple matches: prefer the most specific local skill for the package or concern you are changing; load additional skills only when the task spans multiple packages or concerns.
<!-- intent-skills:end -->

## Repository structure

Important directories:

- `apps/os` - the dashboard for our product. In production, this is served on `os.iterate.com`. In development, it is something like `<username>.iterate-dev.com`
- `apps/daemon` - the entrypoint for our "agent" which runs on Docker-based sandboxed machines (Fly.io or plain Docker (locally))
- `packages/iterate` - the iterate CLI, which is globally installed `iterate`. Note that the CLI delegates to the local source code when run inside this repo, so you can use the globally-installed binary without worrying about which version is running
- `spec` - our Playwright end to end tests. We call them "specs" rather than "e2e" because we use them to declare how our product is supposed to function.

## Dev environment

Locally, the dev server is run with `pnpm dev`. Sometimes, the user will already be running the dev server. If you need to look at its logs, but can't access them, you should kill the server that's running and run it again yourself with nohup, piping stdout to a log file you can tail. Tell the user when you do this to prevent confusion.

The dev server for the OS in general listens on port 5173, but is accessed via a cloudflare tunnel (`<username>.iterate-dev.com`). If you try to access localhost:5173, you will usually get a redirect response.

The database for development runs via docker compose. To get its port on the host machine, you can run `tsx ./scripts/docker-compose.ts port postgres 5432`.

When making changes to the daemon, or any other services that run in the sandbox, run `pnpm sandbox buildx` to build the sandbox Docker image first. This will automatically set the correct image tag in the user's doppler config. To build for fly.io, it's `pnpm sandbox build`.

Doppler is used for secrets management. Most commands don't need to worry about doppler, but if secrets or variables stored in doppler are needed, you can run `doppler run -- ./some-script.sh` and the script will automatically receive the correct environment variables. To look at a variable, you can run a command like `doppler run -- env | grep POSTHOG_PUBLIC_KEY`. You don't in general need to use the `--config` option, you can assume the user has set up their doppler config via the CLI already.

## Writing specs

Specs and end-to-end test are critical to us. They should be readable, coherent and meaningful. These are arguably more important than the product code, because they represent the decisions we've made about how the product should work. You should use specs and tests to drive your feature work - when building something complex, you can write a test roughly describing how it should work, then iterate on the product until the test passes.

We use playwright, but there are some conventions you need to follow when you're writing them.

We have a custom playwright plugin system that adds additional waiters and logic to locator-based assertions. The most important one is `spinner-waiter`. This enables us to have a very short default wait timeout, and looks for loading UI in the DOM when the timeout passes without the element appearing. What this means:

- Timeouts can stay very short. If neither the target UI nor a loading spinner appears within 1s, the test will fail fast.
- When a test fails for this reason, but it's a legitimately long operation, instead of bumping the timeout, we should update the product code to add a loading spinner. This means the test stays fast and reliable and our product actually improves.
- In general, don't use `expect` for DOM verification assertions. Use `await page.locator(...).waitFor()`. This will intelligently wait for loading UI, but `await expect(...).toBeVisible()` won't
- Loading UI gives a 30s grace period. If it's an extremely long operation, it can be extended by importing `spinnerWaiter`:
- Aim not to look for anything to be hidden/detached. Instead, make positive assertions ("element with XYZ became visible")
- If the only user-visible content to match on is ambiguous, you can add `data-*` attributes to the product code to make matchers more robust. (e.g. `data-label="machine-detail"` or `data-testid="email-input"`)

```ts
await spinnerWaiter.settings.run({ spinnerTimeout: 120_000 }, async () => {
  await page.locator(".foo-bar").waitFor(); // assertions in this scope get 120s of "spinner time" granted
});
```

Don't write if statements, ternaries, or other conditionals in tests. You should usually duplicated code over complex helper functions with conditionals.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iterate/iterate](https://github.com/iterate/iterate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
