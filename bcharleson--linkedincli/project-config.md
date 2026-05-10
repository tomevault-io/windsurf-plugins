---
trigger: always_on
description: > This file documents every tool available in the LinkedIn CLI / MCP server.
---

# AGENTS.md — LinkedIn CLI Tool Reference for AI Agents

> This file documents every tool available in the LinkedIn CLI / MCP server.
> AI agents (Claude Code, Cursor, Windsurf, OpenClaw, etc.) should use this as the authoritative reference for managing LinkedIn.

## Authentication

Before using any tool, the agent needs LinkedIn session cookies. These are stored in `~/.linkedin-cli/config.json` after running `linkedin login`, or can be set via environment variables:

```
LINKEDIN_LI_AT=<your li_at cookie>
LINKEDIN_JSESSIONID=<your JSESSIONID cookie>
```

To check if the session is valid:
```bash
linkedin status
```

---

## Tool Reference

### Profile Tools

#### `profile_me`
Get the authenticated user's own LinkedIn profile.
```bash
linkedin profile me
```
**Parameters:** None
**Returns:** Full profile object (firstName, lastName, headline, entityUrn, publicIdentifier, etc.)

---

#### `profile_view`
View any LinkedIn profile by their public identifier (the URL slug).
```bash
linkedin profile view johndoe
```
**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| `public_id` | string | yes | The public profile identifier from their LinkedIn URL |

**Returns:** Full profile view including positions, education, languages, publications, certifications, skills

---

#### `profile_contact-info`
Get contact information for a profile (email, phone, websites, Twitter handles).
```bash
linkedin profile contact-info johndoe
```
**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| `public_id` | string | yes | Public profile identifier |

**Returns:** `emailAddress`, `websites[]`, `twitterHandles[]`, `phoneNumbers[]`, `birthDateOn`

---

#### `profile_skills`
List skills for a profile.
```bash
linkedin profile skills johndoe --count 50
```
**Parameters:**
| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `public_id` | string | yes | — | Public profile identifier |
| `count` | number | no | 100 | Number of skills to return (1-100) |

---

#### `profile_network`
Get network info: connection count, follower count, connection distance, following state.
```bash
linkedin profile network johndoe
```
**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| `public_id` | string | yes | Public profile identifier |

---

#### `profile_badges`
Check if a profile has premium, influencer, job seeker, or open link badges.
```bash
linkedin profile badges johndoe
```
**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| `public_id` | string | yes | Public profile identifier |

**Returns:** `{ influencer, jobSeeker, openLink, premium }` (booleans)

---

#### `profile_privacy`
Get privacy settings for a profile.
```bash
linkedin profile privacy johndoe
```
**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| `public_id` | string | yes | Public profile identifier |

---

#### `profile_posts`
List recent posts from a profile. Requires the numeric URN ID (not the public identifier).
```bash
linkedin profile posts ACoAABxxxxxxx --count 20
```
**Parameters:**
| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `urn_id` | string | yes | — | Profile URN ID (numeric part) |
| `count` | number | no | 10 | Number of posts (1-100) |
| `start` | number | no | 0 | Pagination offset |

---

#### `profile_disconnect`
Remove an existing connection (unfriend).
```bash
linkedin profile disconnect johndoe
```
**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| `public_id` | string | yes | Public profile identifier |

---

### Post Tools

#### `posts_create`
Create a new LinkedIn post. Supports text-only and image posts.
```bash
linkedin posts create --text "Hello LinkedIn!"
linkedin posts create --text "Check this out" --image ./photo.jpg
linkedin posts create --text "For connections only" --visibility connections
```
**Parameters:**
| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `text` | string | yes | — | Post text content (max 3000 chars) |
| `visibility` | enum | no | `anyone` | `anyone` or `connections` |
| `image` | string | no | — | File path to image to attach |
| `comments_scope` | enum | no | `all` | Who can comment: `all`, `connections`, `none` |

---

#### `posts_edit`
Edit the text of an existing post.
```bash
linkedin posts edit "urn:li:share:12345" --text "Updated content"
```
**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| `share_urn` | string | yes | The share URN of the post |
| `text` | string | yes | New post text (max 3000 chars) |

---

#### `posts_delete`
Delete a post.
```bash
linkedin posts delete "urn:li:share:12345"
```
**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| `share_urn` | string | yes | The share URN of the post to delete |

---

### Feed Tools

#### `feed_view`
View your LinkedIn feed in chronological order.
```bash
linkedin feed view --count 20
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bcharleson/linkedincli](https://github.com/bcharleson/linkedincli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
