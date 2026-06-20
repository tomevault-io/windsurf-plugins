---
trigger: always_on
description: Automate TikTok + Instagram slideshow marketing for any app or product. Researches competitors, generates AI images, adds text overlays, posts via Upload-Post (multi-platform in one API call), tracks analytics, and iterates on what works. Use when setting up TikTok marketing automation, creating slideshow posts, analyzing post performance, optimizing app marketing funnels, or when a user mentions TikTok growth, slideshow ads, or social media marketing for their app. Covers competitor research (b
---


# TikTok App Marketing

Automate your entire TikTok slideshow marketing pipeline: generate → overlay → post → track → iterate.

**Proven results:** 7 million views on the viral X article, 1M+ TikTok views, $670/month MRR — all from an AI agent running on an old gaming PC.

## Prerequisites

This skill does NOT bundle any dependencies. Your AI agent will need to research and install the following based on your setup. Tell your agent what you're working with and it will figure out the rest.

### Required
- **Node.js** (v18+) — all scripts run on Node. Your agent should verify this is installed and install it if not.
- **node-canvas** (`npm install canvas`) — used for adding text overlays to slide images. This is a native module that may need build tools (Python, make, C++ compiler) on some systems. Your agent should research the install requirements for your OS.
- **Upload-Post** — this is the backbone of the whole system. Upload-Post handles posting to TikTok, Instagram, and 10+ other platforms simultaneously with a single API call. It also provides **analytics** (followers, impressions, reach) and **upload history** (per-post tracking) that power the daily feedback loop. Without Upload-Post, the agent can't post or track what's working — and the feedback loop is what makes this skill actually grow your account instead of just posting blindly. Sign up at [upload-post.com](https://upload-post.com).

### Image Generation (pick one)
You choose what generates your images. Your agent should research the API docs for whichever you pick:
- **OpenAI** — `gpt-image-1.5` **(ALWAYS 1.5, never 1)**. Needs an OpenAI API key. Best for realistic photo-style images. This is what Larry uses and what we strongly recommend.
- **Stability AI** — Stable Diffusion XL and newer. Needs a Stability AI API key. Good for stylized/artistic images.
- **Replicate** — run any open-source model (Flux, SDXL, etc.). Needs a Replicate API token. Most flexible.
- **Local** — bring your own images. No API needed. Place images in the output directory and the script skips generation.

### Conversion Tracking (optional but recommended for mobile apps)
- **RevenueCat** — this is what completes the intelligence loop. Upload-Post tells you which posts get impressions. RevenueCat tells you which posts drive **paying users**. Combined, the agent can distinguish between a viral post that makes no money and a modest post that actually converts — and optimize accordingly. Install the RevenueCat skill from ClaWHub (`clawhub install revenuecat`) for full API access to subscribers, MRR, trials, churn, and revenue. There's also a **RevenueCat MCP** for programmatic control over products and offerings from your agent/IDE.

### Cross-Posting (built-in with Upload-Post)
Upload-Post supports posting to TikTok, Instagram, YouTube, LinkedIn, Facebook, X (Twitter), Threads, Pinterest, Reddit, and Bluesky — all in a single API call. Your agent should research which platforms fit your audience and connect them in your Upload-Post profile. Same content, different algorithms, more reach.

## First Run — Onboarding

When this skill is first loaded, IMMEDIATELY start a conversation with the user. Don't dump a checklist — talk to them like a human marketing partner would. The flow below is a guide, not a script. Be natural. Ask one or two things at a time. React to what they say. Build on their answers.

**Important:** Use `scripts/onboarding.js --validate` at the end to confirm the config is complete.

### Phase 0: TikTok Account Warmup (CRITICAL — Don't Skip This)

Before anything else, check if the user already has a TikTok account with posting history. If they're creating a fresh account, they MUST warm it up first or TikTok will treat them like a bot and throttle their reach from day one.

Explain this naturally:

> "Quick question before we dive in — do you already have a TikTok account you've been using, or are we starting fresh? If it's new, we need to warm it up first. TikTok's algorithm watches how new accounts behave, and if you go straight from creating an account to posting AI slideshows, it flags you as a bot and kills your reach."

**If the account is new or barely used, walk them through this:**

The goal is to use TikTok like a normal person for **7-14 days** before posting anything. Spend **30-60 minutes a day** on the app:

- **Scroll the For You page naturally.** Watch some videos all the way through. Skip others halfway. Don't watch every single one to the end — that's not how real people scroll.
- **Like sparingly.** Maybe 1 in 10 videos. Don't like everything — that's bot behaviour. Only like things you'd genuinely engage with in your niche.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Upload-Post/upload-post-larry-marketing-skill](https://github.com/Upload-Post/upload-post-larry-marketing-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
