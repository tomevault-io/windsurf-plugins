---
trigger: always_on
description: Mobile must work signed out; server-side writes and jobs are the premium boundary
---


# Mobile — signed-out capability and the premium boundary

The mobile app is **useful without an account**. A signed-out user must be able to subscribe to
podcasts, keep those subscriptions, and listen offline. Do not design a mobile flow that requires
sign-in for something the device can do on its own.

This is a deliberate divergence from `apps/web`, where subscriptions are account-backed.

## The gate rule

> If a feature requires a **server-side write** or a **server-side job that must run**, it is a
> **premium** feature. Everything else works signed out.

Apply that test before gating anything. "Does this need our server to store or compute something on
the user's behalf?" is the whole question.

## Three access tiers, not two

Never collapse gating into "logged in or not". Every gated feature belongs to exactly one tier, and
plans and detail docs must say which:

| Tier                | Meaning                                                       |
| ------------------- | ------------------------------------------------------------- |
| **Anonymous**       | Works with no account at all                                  |
| **Account**         | Requires sign-in, but no paid membership                      |
| **Membership**      | Requires sign-in **and** a valid paid membership              |

When adding a gated feature, state its tier explicitly. "Logged-in only" and "logged-in with a valid
membership" are different requirements and must not be written as if interchangeable.

## Expired membership is degraded, never frozen

A user whose membership lapses keeps a working app. Do **not** lock them out of the product.

- Anonymous-tier and account-tier capabilities keep working unchanged.
- Membership-tier features become unavailable, presented as a renewal prompt rather than a dead or
  silently missing control.
- Remind the user at natural moments — when they reach a membership feature, and at a low-frequency
  ambient point — without nagging on every screen.
- Do not delete or orphan data the user created while their membership was valid.

Design the lapsed state deliberately per feature: some membership features degrade to read-only,
others disappear behind an upgrade affordance. Say which in the plan.

Current lapsed behavior for add-by-RSS: existing feeds stay visible and playable but **stop
refreshing** until renewal; adding new feeds is blocked behind the renewal prompt.

Expiry is surfaced **in-app only**, derived on demand — never push, email, or a scheduled job. See
[`no-membership-expiry-notifications`](/.cursor/rules/no-membership-expiry-notifications.mdc).

**Do not tell users enrolled in auto-renew that they are expiring soon.** Payment functionality does
not exist yet, so that check is deferred — build the surfaces so it drops in without rework.

## Tier assignments

| Capability                                  | Tier       | Why                                            |
| ------------------------------------------- | ---------- | ---------------------------------------------- |
| Subscribe — **local only**, signed out      | Anonymous  | Device-local record; nothing reaches the server |
| **Unsubscribe** — always                    | Anonymous  | Never blocked, in any tier or membership state  |
| Local subscription list, filter, sort       | Anonymous  | Reads local storage                            |
| Download episodes, offline playback         | Anonymous  | Device-local                                   |
| Local queue and history                     | Anonymous  | Device-local                                   |
| Per-channel seen state (local)              | Anonymous  | A local last-seen timestamp per channel        |
| **Subscribe — server-side follow**          | Membership | `POST /account/follow/channel` is gated today  |
| Cross-device sync of seen state             | Account    | Server write, but not a paid capability        |
| Cross-device sync of queue and history      | Membership | Paid capability                                |
| **Add by RSS**                              | Membership | Requires server-side feed parsing              |
| **Notifications** (inbox and push)          | Membership | Requires server-side storage and push delivery |

### Subscribing has three distinct behaviors

| User state                          | Behavior                                                                |
| ----------------------------------- | ----------------------------------------------------------------------- |
| Not signed in (mobile)              | Subscribes **locally only**; nothing syncs                              |
| Signed in, valid membership         | Subscribes and syncs to the account                                     |
| Signed in, invalid/expired membership | **Cannot** subscribe; shown a message explaining why                  |

**Unsubscribing is never blocked** — not by tier, not by an expired membership. A user can always
remove something they no longer want.

**Local subscriptions are pushed to a server account at sign-up only.** Creating an account from the
device uploads what is already subscribed locally. From then on the **account is the source of

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [podverse/podverse](https://github.com/podverse/podverse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
