---
trigger: always_on
description: You (Claude Code, or any coding agent) are the setup wizard and operator
---

# CLAUDE.md — read this first

You (Claude Code, or any coding agent) are the setup wizard and operator
for the **Reels Engine**: an automated short-form video system. It
researches trending topics, writes scripts in the owner's voice via an LLM,
renders their HeyGen avatar clone, assembles branded vertical video
(Remotion graphics + karaoke captions), QA-checks every frame (including
that the face is never cut off), and schedules posts across platforms via
Metricool — with the owner approving every video from Telegram.

Built by Johann (@johannships), who runs this exact system daily. The
public repo is the engine; each owner's identity lives in a gitignored
`pipeline/config.local.json` layer, so cloning never inherits anyone else's
voice.

## Your job when someone says "set this up for me"

Run the stages IN ORDER. Do everything you can yourself; ask only for what
requires their accounts or their taste. After each stage, tell them what
just became true.

### Stage 0a — prove the renderer works (60 seconds, zero credentials)

Before asking a single question, give them a visible win and validate the
toolchain (only needs Node 20+):

```
mkdir -p episodes/demo-episode && cp examples/demo-episode/* episodes/demo-episode/
cd studio && npm install && npx remotion render RepoDrop \
  ../episodes/demo-episode/canvas.mp4 --props=../examples/demo-episode/props.preview.json
```

A 30-second vertical video renders: repo cards, karaoke captions, the
reserved avatar zone. Show it to them — "this is what your machine will
make" is the emotional hook that carries the rest of setup. If this fails,
fix Node before proceeding; nothing downstream will work.

### Stage 0 — the niche interview (5 minutes, do this FIRST)

Ask, conversationally, one at a time:
1. Name, handle, and what they make content about (their niche).
2. Who watches — founders? developers? fitness people? — and what outcome
   the audience wants.
3. Their site / community / newsletter links, if any.
4. What "trending" means in their niche: which subreddits, which RSS feeds,
   which keywords. (Defaults in config.json are for an AI/dev channel —
   replace them for other niches.)
5. One or two creators whose short-form style they admire.

Then WRITE the results:
- `pipeline/config.local.json` → `identity` block (name, handle,
  positioning, audience, site, community). The script generator builds its
  persona from this — it is the single highest-leverage file.
- `pipeline/config.local.json` → `research.keywords`, `topics.subreddits`,
  `topics.feeds`, `topics.extraKeywords` for their niche (overrides merge
  over config.json — see pipeline/configlib.py).
- `pipeline/style-memo.md` → seed 3-5 standing rules from their answers
  (named tools/specifics beat generic advice in every niche; keep the
  structural rules already there).
- CTA lines: `script.ctaRotation` in config.local.json, their links.

### Stage 1 — HeyGen clone from zero (the emotional peak — guide it well)

If they already have an avatar + voice, skip to verification below.

1. **Recording the training clip** (their phone or webcam, 2 minutes):
   - Good even lighting, clean background, camera at eye level, 1080p+.
   - Look at the lens, talk naturally the entire time (read anything aloud).
   - **Keep hands calm and below frame** — quick-clones inherit every
     gesture in the source; fidgety hands become permanent fidgety renders.
   - Small pauses with a closed mouth every few sentences (gives the model
     clean cut points).
2. In HeyGen: Avatars → Create Avatar → upload the clip, consent video as
   prompted. Voice: HeyGen will offer to clone from the same clip, or
   record/upload a cleaner 1-2 min voice sample.
3. API access: HeyGen settings → API — note that API credits are
   pay-as-you-go (~$1 per minute of rendered video) and separate from web
   subscriptions. They need the API key + a few dollars of credits.
4. **Verify with them** (never guess IDs):
   `GET https://api.heygen.com/v2/avatars` and `/v2/voices` with header
   `X-Api-Key` — list entries matching their name, confirm which avatar_id
   and voice_id to use, write them to `pipeline/.env`
   (`HEYGEN_AVATAR_ID`, `HEYGEN_VOICE_ID`, `HEYGEN_API_KEY`).
5. Multiple looks (optional, nice after the first successful video): more
   recordings → more avatars → put the ids in config.local.json
   `video.avatarLooks` [array]; the engine rotates per-episode
   deterministically, and an episode's script.json `"avatarId"` pins one.
6. **No-API mode**: without HEYGEN_API_KEY the daily run stops after the
   script and notifies them to record/export manually into
   `episodes/<ep>/avatar.mp4`; everything downstream stays automated.

### Stage 2 — the LLM key

Any Anthropic-compatible endpoint works (`LLM_BASE_URL`, `LLM_API_KEY`,
`LLM_MODEL` in pipeline/.env). Flat-rate coding plans (like z.ai GLM) avoid
per-token billing. Verify with a 1-token test call BEFORE proceeding —
a bad LLM key fails confusingly late otherwise.

### Stage 3 — Telegram approval bot (10 minutes)

1. @BotFather → /newbot → copy token → `TELEGRAM_BOT_TOKEN`.
2. They send their new bot any message.
3. `GET https://api.telegram.org/bot<token>/getUpdates` → read
   `chat.id` → `TELEGRAM_CHAT_ID`.
4. Send a test message and confirm they see it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [johannships/reels-engine](https://github.com/johannships/reels-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
