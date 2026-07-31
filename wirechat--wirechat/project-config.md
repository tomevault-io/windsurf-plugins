---
trigger: always_on
description: This file is the project playbook for AI coding agents working in `wirechat`.
---

# AGENTS.md

This file is the project playbook for AI coding agents working in `wirechat`.

Read this before making changes. If the user gives direct instructions that conflict with this file, follow the user.

## What This Package Is

`wirechat` is the base Laravel + Livewire chat package for:

- private chats
- self chats
- group conversations
- embeddable widget chat
- attachments and media sharing
- replies, deletes, and group member management
- panel-based configuration and theming

This file should read as `wirechat`-first guidance, not as a trimmed copy of pro-only conventions.

Think of this package as the core product surface:

- it should feel complete and coherent on its own
- it should stay friendly to projects using only the base package
- it may share concepts or extension seams with pro, but pro is not the default assumption here

Core expectations:

- it is panel-driven
- it supports polymorphic participants
- it supports multiple guards and participant model types
- it supports full-page and widget chat experiences
- it exposes package APIs through panels, traits, contracts, routes, and model overrides

Changes should preserve package compatibility and keep extension points stable.

## Relationship To Pro

`wirechat-pro` may build on some of the same ideas, naming patterns, or extension surfaces, but this repo should not be shaped around pro-first assumptions.

When working here:

- design and document features so they make sense in the base package on their own
- avoid language that makes the base package sound incomplete or secondary
- acknowledge shared concepts with pro when it helps future compatibility
- do not introduce abstractions that only make sense because of pro unless the user explicitly asks for that direction
- prefer neutral wording like "shared", "compatible", or "future-friendly" over "pro-only" framing

## Core Stack

Main runtime dependencies from `composer.json`:

- PHP `^8.1|^8.2|^8.3|^8.4`
- Laravel `^10|^11|^12|^13`
- Livewire `^3.7|^4.0`
- Laravel Prompts

Main dev/test tooling:

- Orchestra Testbench
- Pest
- Laravel Pint
- Larastan / PHPStan

Backward compatibility across supported Laravel and Livewire versions matters.

## Main Package Concepts

### 1. Panels are a core extension surface

Panels are one of the most important concepts in Wirechat.

- Panels are registered through `PanelProvider` classes.
- Each panel must have an `id()`.
- Panels can be resolved by id or provider class.
- One panel can be marked as `->default()`.
- Panel state is tracked through `PanelRegistry`.

Key files:

- `src/Panel.php`
- `src/PanelProvider.php`
- `src/PanelRegistry.php`
- `src/Services/WirechatService.php`
- `workbench/app/Providers/Wirechat/TestPanelProvider.php`

If a change affects routes, middleware, search, theming, auth, uploads, group behavior, or widget behavior, check whether it should be panel-aware.

### 2. Panel configuration is public API

Do not casually break panel methods or their meaning.

Important panel options currently include:

- `id()`
- `path()`
- `default()`
- `layout()`
- `middleware()`
- `chatMiddleware()`
- `guards()`
- `groups()`
- `groupInvitations()`
- `invitePageLayout()`
- `maxGroupMembers()`
- `attachments()`
- `mediaAttachments()`
- `fileAttachments()`
- `maxUploads()`
- `mediaMimes()`
- `mediaMaxUploadSize()`
- `fileMimes()`
- `fileMaxUploadSize()`
- `chatsSearch()`
- `searchableAttributes()`
- `searchUsersUsing()`
- `createChatAction()`
- `createGroupAction()`
- `clearChatAction()`
- `deleteChatAction()`
- `deleteMessageActions()`
- `emojiPicker()`
- `webPushNotifications()`
- `serviceWorkerPath()`
- `broadcasting()`
- `messagesQueue()`
- `eventsQueue()`
- `colors()`
- `favicon()`
- `heading()`
- `heart()`
- `redirectToHomeAction()`
- `homeUrl()`

When changing panel behavior:

- treat it as package API
- consider existing panel providers in user apps
- prefer additive changes over breaking renames or semantic changes

### 3. Multi-auth and multi-model support are fundamental

Wirechat is built around polymorphic participants, not a single hard-coded user model.

- Participants use `participantable_id` + `participantable_type`.
- Multiple authenticatable model types can participate in conversations.
- Panels can authenticate through multiple guards via `guards([...])`.
- Access is also controlled per user model via `canAccessWirechatPanel(Panel $panel)`.

Important public surfaces:

- `src/Contracts/WirechatUser.php`
- `src/Traits/InteractsWithWirechat.php`
- `src/Models/Participant.php`
- `src/Models/Conversation.php`

The expected user-model integration is:

- implement `WirechatUser`
- use `InteractsWithWirechat`

The old `Chatable` trait still exists for backward compatibility, but `InteractsWithWirechat` is the preferred path.

Avoid changes that assume:

- only one auth guard
- only one user model
- only `App\Models\User`

### 4. Group flows are first-class package behavior

Groups are not a side feature. They include:

- ownership and admin roles
- permissions
- adding members
- invite links
- invite landing and join flow
- join requests
- past members and blocked members

Important files include:

- `src/Models/Group.php`
- `src/Models/Invite.php`
- `src/Models/JoinRequest.php`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wirechat/wirechat](https://github.com/wirechat/wirechat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
