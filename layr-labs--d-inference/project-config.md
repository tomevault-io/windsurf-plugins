---
trigger: always_on
description: EigenInference is a decentralized/private inference stack for Apple Silicon Macs. Consumers use OpenAI-compatible APIs, the coordinator handles routing/auth/billing/attestation, and providers run local text, transcription, and image workloads on macOS hardware.
---

# EigenInference - Decentralized Private Inference

EigenInference is a decentralized/private inference stack for Apple Silicon Macs. Consumers use OpenAI-compatible APIs, the coordinator handles routing/auth/billing/attestation, and providers run local text, transcription, and image workloads on macOS hardware.

## Project Structure

```text
coordinator/          Go control plane
├── cmd/coordinator/  main service entrypoint
├── cmd/verify-attestation/
│   └── main.go       verifies attestation blobs from /tmp/eigeninference_attestation.json
└── internal/
    ├── api/          HTTP + WebSocket handlers
    │   ├── consumer.go         OpenAI-compatible chat/completions/messages/transcriptions/images
    │   ├── provider.go         provider registration, heartbeats, attestation, relay
    │   ├── billing_handlers.go Stripe/Solana/referral/pricing endpoints
    │   ├── device_auth.go      device code flow for linking providers to user accounts
    │   ├── enroll.go           MDM + ACME enrollment profile generation
    │   ├── invite_handlers.go  invite code admin/user flows
    │   ├── release_handlers.go binary release registration (GitHub Actions integration)
    │   ├── acme_verify.go      ACME device-attest-01 client cert verification
    │   ├── stats.go            public network stats
    │   └── server.go           route wiring, auth middleware, version gate
    ├── attestation/  Secure Enclave + MDA verification
    ├── auth/         Privy JWT integration
    ├── billing/      Stripe, Solana USDC deposits, referrals
    ├── e2e/          X25519 request-encryption helpers
    ├── mdm/          MicroMDM client + webhook handling
    ├── payments/     internal ledger + pricing
    ├── protocol/     WebSocket message types shared with provider
    ├── registry/     provider registry, queueing, routing, reputation
    └── store/        in-memory or Postgres persistence

provider/             Rust provider agent for Apple Silicon Macs
├── src/
│   ├── main.rs       CLI (`serve`, `start`, `stop`, `models`, `benchmark`, `status`, `doctor`, `login`, etc.)
│   ├── coordinator.rs WebSocket client, registration, heartbeats, request handling
│   ├── proxy.rs      text, transcription, and image proxying to local backends
│   ├── backend/      vllm-mlx backend process management
│   ├── service.rs    launchd install/start/stop helpers
│   ├── server.rs     local-only HTTP server mode
│   ├── config.rs     TOML config + hardware-based defaults
│   ├── hardware.rs   Apple Silicon detection + live system metrics
│   ├── hypervisor.rs Hypervisor.framework Stage 2 page table memory isolation
│   ├── scheduling.rs time-based availability windows
│   ├── security.rs   SIP, Secure Boot, anti-debug (PT_DENY_ATTACH), integrity checks
│   ├── crypto.rs     X25519 keypair management
│   ├── models.rs     local text/image model discovery (fast scan, on-demand hashing)
│   ├── inference.rs  in-process MLX inference (behind "python" feature flag)
│   ├── protocol.rs   message types mirrored from coordinator/internal/protocol
│   └── wallet.rs     legacy provider wallet (secp256k1)
├── stt_server.py     local speech-to-text server script used by bundles
└── Cargo.toml        default `python` feature enables in-process PyO3 inference

image-bridge/         Python FastAPI image generation bridge
├── eigeninference_image_bridge/
│   ├── __main__.py
│   ├── server.py              OpenAI-compatible `/v1/images/generations`
│   ├── drawthings_backend.py  Draw Things gRPC backend adapter
│   ├── generated/             generated protobuf/FlatBuffers glue
│   └── proto/
├── requirements.txt
└── tests/                     pytest coverage for server/backend/integration

app/EigenInference/            SwiftUI macOS menu bar app
├── Sources/EigenInference/
│   ├── EigenInferenceApp.swift
│   ├── StatusViewModel.swift
│   ├── ProviderManager.swift
│   ├── CLIRunner.swift
│   ├── ConfigManager.swift
│   ├── LaunchAgentManager.swift
│   ├── SecurityManager.swift
│   ├── ModelManager.swift / ModelCatalog.swift
│   ├── IdleDetector.swift
│   ├── NotificationManager.swift / UpdateManager.swift
│   ├── DesignSystem.swift / GuideAvatar.swift / Illustrations.swift
│   ├── DashboardView.swift / SettingsView.swift
│   ├── MenuBarView.swift / SetupWizardView.swift
│   ├── DoctorView.swift / LogViewerView.swift / ModelCatalogView.swift
│   └── Resources/
└── Tests/EigenInferenceTests/

enclave/              Swift Secure Enclave helper + bridge binary
├── Sources/EigenInferenceEnclave/      enclave key + attestation library + FFI bridge
├── Sources/EigenInferenceEnclaveCLI/   `eigeninference-enclave` CLI (attest, sign, info)
├── Tests/EigenInferenceEnclaveTests/
└── include/eigeninference_enclave.h

console-ui/           Next.js 16 / React 19 frontend
├── src/app/          chat, billing, images, models, stats, providers, settings, link, api-console, earn
├── src/app/api/      chat, images, transcribe, auth/keys, payments/*, invite, models, health, pricing
├── src/components/   chat UI, sidebar, top bar, trust badge, verification panel, invite banner

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Layr-Labs/d-inference](https://github.com/Layr-Labs/d-inference) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
