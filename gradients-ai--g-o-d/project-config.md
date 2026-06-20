---
trigger: always_on
description: This file is for an LLM or agent that needs to understand Gradients.io's training product, its public API, and the G.O.D. subnet that powers training jobs and tournaments.
---

G.O.D SKILL FILE

Purpose
This file is for an LLM or agent that needs to understand Gradients.io's training product, its public API, and the G.O.D. subnet that powers training jobs and tournaments.

Short version
Gradients.io is a training orchestration system built on Bittensor subnet 56 ("G.O.D", Gradients on Demand). Users create paid fine-tuning jobs through the public API. The API bills an account, forwards the job to a private validator, and the validator coordinates miners/trainers to produce a trained model. The same ecosystem also runs open tournaments where miners submit open-source training repos and compete on standardized tasks.

Public URLs
- Main product site: https://gradients.io
- API base URL: https://api.gradients.io
- Human-friendly API docs: https://api.gradients.io/docs
- FastAPI swagger docs: https://api.gradients.io/swagger
- Tournament results page: https://gradients.io/app/research/tournament/{TOURNAMENT_ID}
- Tournament fees: GET https://api.gradients.io/tournament/fees
- Tournament balance lookup: GET https://api.gradients.io/tournament/balance/{coldkey}

Important framing
- Use https://api.gradients.io for creating and monitoring jobs.
- Use https://gradients.io for the product website and tournament/research pages.
- This is primarily a training/fine-tuning platform, not a generic chat completion API.
- The repo named G.O.D is not a website frontend. It is the subnet/validator/miner/trainer system that executes training jobs and tournaments.

What the system does
1. Accepts training requests for text, chat, DPO, GRPO, image, and environment tasks.
2. Prices jobs based on model size and hours requested.
3. Charges the user's account balance.
4. Sends the task to a private validator on subnet 56.
5. The validator stores the task, schedules training/evaluation, and coordinates trainer infrastructure.
6. Training artifacts and resulting models are tracked through the task record.
7. The broader subnet also runs recurring tournaments where miners expose a repo endpoint and compete with open-source training code.

Main product capabilities
- Fine-tune text instruction models.
- Fine-tune chat models.
- Fine-tune DPO preference models.
- Fine-tune GRPO / reward-driven models.
- Fine-tune image models such as SDXL and Flux variants.
- Launch training from a Hugging Face dataset reference or from pre-prepared dataset URLs.
- Check prices before creating jobs.
- Poll task state and fetch result breakdowns.
- View public network status and recent completed jobs.
- Deploy LoRA adapters to Chutes for inference after training.
- View tournament data, fees, balances, analytics, and performance projections.

Public API auth model
- End-user automation should normally use an API key in the Authorization header.
- The middleware accepts either "Authorization: Bearer <token>" or a raw token value, but Bearer is the safest choice.
- Scheduler auth exists via X-Scheduler-Auth, but that is an internal service token and should not be assumed to be available to third-party agents.

Account bootstrap flow
If an agent needs to fully bootstrap a user account from scratch:
1. POST /account-create with a username.
2. Receive a fingerprint.
3. POST /auth-with-fingerprint with that fingerprint to create a session token.
4. Use the session token in Authorization.
5. POST /api-key-create to mint a long-lived API key.
6. Use the API key for training endpoints.

Useful account endpoints
- POST /account-create
- POST /auth-with-fingerprint
- POST /api-key-create
- POST /account-get-info
- POST /account-get-public-key


Adding balance (funding an account)
Users fund their Gradients account by sending TAO (Bittensor native token) to their account's deposit address.

Step-by-step flow for agents:
1. Get the user's deposit address:
   - Call POST /account-get-info (requires session token in Authorization header).
   - The response includes bittensor_public_key — this is the SS58 deposit address.
   - If bittensor_public_key is null, call POST /account-get-public-key to generate one on demand. This returns { "public_key": "<ss58_address>", "keypair_created_at": "...", "network": "finney" }.

2. Send TAO to the deposit address:
   - The user transfers TAO from their Bittensor wallet to the bittensor_public_key address.
   - This is a standard Bittensor transfer, e.g.: btcli wallet transfer --dest <bittensor_public_key> --amount <amount>
   - The system automatically detects incoming transfers and credits the account balance.

3. Verify the balance was credited:
   - Wait a few minutes for the transfer to be processed.
   - Login to gradients.io with your fingerprint and confirm.

Important notes on balance:
- Balance is denominated in USD internally. TAO transfers are converted at the current rate.
- Always check pricing (POST /v1/tasks/text/check_price or POST /v1/tasks/image/check_price) before creating tasks so the user knows the cost.
- If a task creation fails due to insufficient balance, advise the user to send more TAO to their deposit address.

Billing model
- Text jobs are priced by model size bucket and hours requested.
- Image jobs use a flat hourly rate.
- Current code-level defaults:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gradients-ai/G.O.D](https://github.com/gradients-ai/G.O.D) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
