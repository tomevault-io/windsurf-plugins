---
trigger: always_on
description: You are probably reading this because somebody said *"read AGENTS.md and set
---

# For a coding assistant

You are probably reading this because somebody said *"read AGENTS.md and set
this up for me"*. Good. This file is the whole brief.

Read it all before you run anything. It is short on purpose.

---

## What this is

A thermal printer with a web page in front of it. Somebody opens a link, writes
a message, and it comes out on paper on the user's desk.

Three parts:

* `web/` — the page people write on. Plain HTML and ES modules, no build step.
* `worker/` — a Cloudflare Worker plus a D1 (SQLite) database. The queue, the
  filter, the admin desk, the API.
* `agent/`, `firmware/`, `web/bridge/` — three ways of driving a printer.
  The user picks **one**.

---

## Your job, in order

Do not skip ahead. Each step fails in a way the next one cannot diagnose.

1. **Ask which printer they have.** Not which they want — which is on the desk,
   or on its way. `docs/04-printers.md` lists what works. If they have not
   bought one yet, help them choose and then stop: nothing below can be tested
   without it.

2. **Ask which of the three set-ups they want.** `README.md` has the table.
   Default to the browser bridge unless they have said otherwise: it is the one
   they can see working in half an hour.

3. **Cloudflare.** They need an account (free, no card). You cannot create it
   for them and you should not try. Once they have it:
   ```sh
   cd worker
   npm install
   npx wrangler login          # opens a browser; they click Allow
   npx wrangler d1 create printer
   ```
   That last command asks what to call the binding and offers `printer`.
   **The answer is `DB`** — the Worker asks Cloudflare for `env.DB`, and taking
   the default adds a second entry while leaving the real one on its
   placeholder. It fails two steps later as "Invalid uuid".

   On Windows, `npm` and `npx` are PowerShell scripts and the default execution
   policy refuses to run them. The cure is one command in their terminal,
   `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`, and it is theirs to
   run, not yours to run for them: it changes a security setting on their
   machine and they should see it happen. `docs/01-quick-start.md` explains it.

4. **Secrets.** Three are required, and one script does all of it:
   ```sh
   node setup.mjs
   ```
   It fills in the database id from their account if step 3 did not, generates
   three 256-bit secrets, uploads them, and writes the two they need again into
   `my-tokens.txt`, which is gitignored. **Tell them that file matters** before
   they close it: Cloudflare will not show a secret twice.

   Do not read the tokens back to them in the chat, and do not put them in a
   commit, a comment, or a file that is not that one.

   By hand, if they would rather watch it happen: generate with
   `node -e "console.log(require('crypto').randomBytes(32).toString('base64url'))"`
   — not `openssl`, which does not exist on Windows — and set each with
   `npx wrangler secret put NAME`.

   Also copy `.dev.vars.example` to `.dev.vars` with different values, for
   local runs. `.dev.vars` is gitignored; keep it that way.

   **`npm run dev` is not offline.** The AI binding has no local
   implementation, so the dev server opens a connection to the user's
   Cloudflare account before it serves anything: it needs `wrangler login`
   and, if their login has more than one account, `account_id` uncommented in
   `wrangler.jsonc`. Without that it exits mid-start-up and no port opens. Do
   not go hunting in the code for this - it is the configuration, and
   `docs/08-troubleshooting.md` says so too.

5. **The database, then the deploy.**
   ```sh
   npm run db:remote
   npm run deploy
   ```

6. **Check it before handing it over.** Open the URL. Open `/admin` and log in
   with the admin token. Send a message. It should appear on the desk, waiting.

7. **Then the printer end**, following whichever of `docs/01`, `docs/02` or
   `docs/03` matches step 2.

**If they already have it running and want the latest fixes**, none of the
above applies: that is `node update.mjs` from the repository root, and
`docs/11-updating.md` explains what it will and will not touch. Do not
hand-roll a pull, a schema run and a deploy - the order matters and the script
has it.

---

## Things you must ask rather than decide

* **The name and the wording on the page.** `web/index.html` says "Print on my
  desk" and a few sentences about what happens. That is theirs to write, and a
  page in your voice rather than theirs is the one thing they will notice
  immediately. Ask, then edit.
* **Whether messages are held for approval.** The default is yes: nothing
  prints until they tap approve. Do not turn `hold_all` off to make a demo
  smoother. If they ask you to, make sure they understand it means a stranger's
  message reaches their paper unread.
* **The daily limit.** Default is three messages per person per day. Ask how
  many people they are sending the link to.
* **Notifications.** Discord and ntfy are both optional and both off. Ask
  before wiring either; it means giving this project a webhook URL.

## Things you must not do

* **Do not deploy to production without saying so first**, and never as a side
  effect of "checking something".

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eliorpom-cmd/print-on-my-desk](https://github.com/eliorpom-cmd/print-on-my-desk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
