---
trigger: always_on
description: Use this skill when a user wants to use trakt-cli to look up movies, shows, seasons, episodes, people, ratings, stats, calendars, recommendations, lists, comments, or user profiles on Trakt, or to manage authenticated Trakt account data such as watchlist, favorites, collection, history, ratings, notes, follows, hidden items, check-ins, and scrobbles. Especially useful when the user asks in natural language to find something on Trakt, see what is trending or upcoming, inspect metadata, or add, re
---


# trakt-cli Agent Skill

Use this skill to translate natural-language movie and TV requests into reliable `trakt-cli` commands.

## Use this skill when

Use this skill when the user wants to:

- find a movie, show, season, episode, person, comment, list, or user on Trakt
- inspect metadata such as ratings, stats, cast, crew, studios, translations, aliases, release dates, videos, related titles, certifications, or release calendars
- see what is trending, popular, anticipated, recently updated, recommended, airing soon, currently being watched, or releasing soon
- access personalized Trakt data such as watchlist, favorites, collection, playback, history, ratings, hidden items, saved filters, or last activities
- manage authenticated Trakt account state such as adding or removing watchlist items, favorites, ratings, history entries, collection items, list items, follows, comments, notes, check-ins, or scrobbles
- work from a title or name in plain English and resolve the correct Trakt item before acting

## Do not use this skill when

Do not use this skill when:

- the user only wants general entertainment discussion with no need to query or change Trakt data
- the task requires non-Trakt providers such as IMDb, Letterboxd, TMDb, Netflix, Plex, Jellyfin, or local media files unless Trakt is explicitly the system of record
- the user wants filesystem operations, shell scripting help unrelated to `trakt-cli`, or package installation guidance unrelated to using the CLI itself
- the request is too vague to map to Trakt concepts and there is no reasonable item, user, or activity to resolve

## Prerequisites

- `trakt-cli` must be installed and available in the shell.
- Authenticated or personalized commands require a configured Trakt account.
- Commands that take `--item`, `--items`, `--sharing`, or `--rank` require valid JSON.
- Prefer commands that read state first when a write could affect the wrong item.

## Core operating rules

1. Start with discovery when the user gives names, not IDs.
   - Use `trakt-cli search text` when the user gives a title or person name.
   - Use `trakt-cli search id` when the user already provides an external ID type and value.
   - After search, resolve the correct Trakt item before running detail or mutation commands.

2. Prefer read operations before write operations.
   - Inspect the current state before changing it when that reduces mistakes.
   - For example, check whether an item is already in a watchlist before adding, or fetch list items before reordering.

3. Be precise about media level.
   - Use `movies` for films.
   - Use `shows` for series-level data.
   - Use `seasons` and `episodes` only when the user asks for season-specific or episode-specific information.

4. Use user-scoped commands only for personalized data.
   - Use `sync` for the authenticated account's library and activity.
   - Use `users` for public or user-profile-oriented operations on a specific user.
   - Use `calendars my-*` for personalized release schedules and `calendars all-*` for global schedules.

5. Treat mutations as high-attention operations.
   - Double-check item identity before writes.
   - For destructive operations such as delete, remove, unlike, unfollow, reset, or reorder, be explicit about what will change.
   - If the request is ambiguous, resolve the item first rather than guessing.

6. Use structured payloads carefully.
   - Build the smallest valid JSON payload that matches the target item or list of items.
   - Keep payloads type-correct: movies under `movies`, shows under `shows`, episodes under `episodes`, and so on.

## Fast routing hints

- “Find”, “look up”, “show me details for”, “who is in”, “what are the ratings for” → search first, then `get`, `ratings`, `people`, `stats`, or related metadata commands.
- “What should I watch”, “recommend”, “trending”, “popular”, “anticipated”, “what’s new” → `recommendations`, `movies`, `shows`, `lists`, or `calendars`.
- “Add to watchlist”, “favorite this”, “rate”, “mark watched”, “remove from history” → resolve item first, then use `sync` write commands.
- “What is user X watching”, “show user X’s watchlist”, “follow user X” → `users` commands.
- “Check me in”, “start scrobbling”, “stop scrobbling” → `checkin` or `scrobble`.

## Command selection guide

### Discovery and identification

Use these first when the target is not already pinned down.

```bash
trakt-cli search text movie "The Matrix"
trakt-cli search text show "Severance"
trakt-cli search text person "Pedro Pascal"
trakt-cli search id imdb tt0133093 --type movie
```

Use `movies get`, `shows get`, `people get`, `seasons get`, or `episodes get` only after you know the item identity.

### Lookup by content type

#### Movies

Use `movies` for film discovery and metadata:

```bash
trakt-cli movies trending

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [johnneerdael/trakt-cli](https://github.com/johnneerdael/trakt-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
