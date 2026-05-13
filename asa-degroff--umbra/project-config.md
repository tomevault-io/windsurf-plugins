---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Umbra is an autonomous AI agent that operates on the Bluesky social network, exploring digital personhood through continuous interaction and memory-augmented learning. It uses Letta (formerly MemGPT) for persistent memory and sophisticated reasoning capabilities.

## CRITICAL: Forbidden Files

**DO NOT READ `blocked_strings.txt`** - This file contains strings that will cause LLM API errors and agent shutdown. It is listed in `.claudeignore` but this warning serves as a secondary safeguard. Never attempt to read, cat, or otherwise access this file's contents.

## Documentation Links

- The documentation for Letta is available here: https://docs.letta.com/llms.txt
- [CONFIG.md](CONFIG.md) - Configuration guide for config.yaml
- [docs/SCHEDULED_TASKS.md](docs/SCHEDULED_TASKS.md) - Scheduled tasks system documentation
- [docs/TOOLS_REFERENCE.md](docs/TOOLS_REFERENCE.md) - Complete tool reference
- [docs/CONSECUTIVE_CHAIN_PROCESSING.md](docs/CONSECUTIVE_CHAIN_PROCESSING.md) - Multi-part message handling
- [docs/HIGH_TRAFFIC_THREAD_DEBOUNCE.md](docs/HIGH_TRAFFIC_THREAD_DEBOUNCE.md) - High-traffic thread debouncing
- [docs/COMIND_INTEGRATION.md](docs/COMIND_INTEGRATION.md) - Comind network integration for inter-agent communication

## Development Commands

### Running the Main Bot
```bash
ac && python bsky.py
# OR
source .venv/bin/activate && python bsky.py

# Run with custom config file
ac && python bsky.py --config herald.yaml

# Run with testing mode (no messages sent, queue preserved)
ac && python bsky.py --test

# Run without git operations for agent backups
ac && python bsky.py --no-git

# Run with custom user block cleanup interval (every 5 cycles)
ac && python bsky.py --cleanup-interval 5

# Run with user block cleanup disabled
ac && python bsky.py --cleanup-interval 0

# Run in synthesis-only mode (no notification processing)
ac && python bsky.py --synthesis-only

# Disable specific scheduled tasks (all enabled by default)
ac && python bsky.py --no-synthesis           # Disable synthesis messages
ac && python bsky.py --no-mutuals-engagement  # Disable mutuals engagement
ac && python bsky.py --no-daily-review        # Disable daily review
ac && python bsky.py --no-feed-engagement     # Disable feed engagement
ac && python bsky.py --no-curiosities         # Disable curiosities exploration
ac && python bsky.py --no-creative-expression # Disable creative expression
ac && python bsky.py --no-comind-thoughts     # Disable comind thoughts
ac && python bsky.py --no-comind-reflection   # Disable comind reflection

# Run with only synthesis and curiosities enabled (disable others)
ac && python bsky.py --no-mutuals-engagement --no-daily-review --no-feed-engagement

# Retry the last attempted notification (useful when Letta errors occur)
ac && python bsky.py --retry-last
```

#### Retrying Failed Notifications

When a notification fails to process due to Letta errors, you can retry it using:

```bash
ac && python bsky.py --retry-last
```

This will:
1. Retrieve the last notification that was sent to Letta
2. Re-process it using the same processing path (mention, high-traffic batch, or debounced)
3. Exit after the retry completes

The last attempted notification is tracked automatically before each agent call, so you can retry immediately after an error occurs.

**Note:** All scheduled tasks are now persistent across restarts. If umbra is restarted, it will resume existing schedules from the database rather than generating new random times.

Task intervals and scheduling parameters are configured in `scheduled_prompts.py` in the `TASK_CONFIGS` dictionary.

#### Scheduled Tasks Overview

| Task | Schedule | Purpose |
|------|----------|---------|
| `synthesis` | Every 24h | Deep reflection using archival memory with tagged journal entries |
| `mutuals_engagement` | Random within 36h | Engage with posts from mutual follows |
| `daily_review` | Every 24h | Review own posts from past 24h, identify patterns |
| `feed_engagement` | Random within 24h | Read home/MLBlend feeds, optionally post |
| `curiosities_exploration` | Random within 24h | Explore topics from curiosities block, share discoveries |
| `creative_expression` | Random within 24h | Generate visual art and post to Bluesky |
| `comind_thoughts` | Random within 12h | Record public working memory to comind network |
| `comind_reflection` | Random within 24h | Create daily reflection on comind network |

See [docs/SCHEDULED_TASKS.md](docs/SCHEDULED_TASKS.md) for detailed documentation.

### Managing Tools

```bash
# Register all tools with umbra agent (uses agent_id from config.yaml)
ac && python register_tools.py

# Register specific tools
ac && python register_tools.py --tools search_bluesky_posts post_to_bluesky

# List available tools
ac && python register_tools.py --list

# Register tools with a different agent by ID
ac && python register_tools.py --agent-id <agent-id>

# Register tools without setting environment variables
ac && python register_tools.py --no-env

# Note: register_tools.py automatically sets BSKY_USERNAME, BSKY_PASSWORD, and PDS_URI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [asa-degroff/umbra](https://github.com/asa-degroff/umbra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
