---
trigger: always_on
description: This package is **platform-agnostic**. It must never import from `src/adapters/` or any platform SDK. The only exception is `core/registry_builder.py` which uses lazy imports inside try/except.
---

# Core Framework (src/marketmenow/)

This package is **platform-agnostic**. It must never import from `src/adapters/` or any platform SDK. The only exception is `core/registry_builder.py` which uses lazy imports inside try/except.

## Module Map

| Module                    | Purpose                                                      |
|---------------------------|--------------------------------------------------------------|
| `models/content.py`       | `ContentModality` enum, `BaseContent` and all content variants |
| `models/campaign.py`      | `Audience`, `ScheduleRule`, `CampaignTarget`, `Campaign`     |
| `models/result.py`        | `PublishResult`, `SendResult`, `MediaRef`, `AnalyticsSnapshot` |
| `models/distribution.py`  | `DistributionRoute`, `DistributionMap` (modality → platforms) |
| `ports/platform_adapter.py` | `PlatformAdapter` protocol                                 |
| `ports/content_renderer.py` | `ContentRenderer` protocol                                 |
| `ports/uploader.py`       | `Uploader` protocol                                         |
| `ports/analytics.py`      | `AnalyticsCollector` protocol                                |
| `normaliser.py`           | `NormalisedContent` model + `ContentNormaliser` (match/case dispatch) |
| `registry.py`             | `PlatformBundle` dataclass + `AdapterRegistry`               |
| `exceptions.py`           | `MarketMeNowError` hierarchy (`AdapterNotFoundError`, `UnsupportedModalityError`, `AuthenticationError`, `PublishError`, `RenderError`, `UploadError`) |
| `cli.py`                  | Top-level Typer app (`mmn`) — `run`, `workflows`, `auth`, `distribute`, `platforms`, `version`, `heal` + hidden adapter CLI groups for web frontend |
| `core/workflow.py`        | `WorkflowStep` protocol, `WorkflowContext`, `Workflow` runner, `ParamDef` |
| `core/workflow_registry.py`| `WorkflowRegistry` + `build_workflow_registry()` — auto-discovers workflows |
| `steps/*.py`              | Reusable workflow steps (generate_reel, post_to_platform, package_capsule, post_from_capsule, discover_posts, discover_prospects, enrich_profiles, score_prospects, generate_messages, send_messages, etc.) |
| `workflows/*.py`          | Built-in workflow definitions (instagram_reel, twitter_engage, twitter_outreach, tiktok_reel, post_capsule, etc.) |
| `outreach/models.py`      | `CustomerProfile`, `UserProfile`, `ScoredProspect`, `OutreachMessage`, rubric models |
| `outreach/ports.py`       | `DiscoveryVector`, `ProfileEnricher`, `MessageSender` protocols |
| `outreach/scorer.py`      | `ProspectScorer` — Gemini rubric evaluation (platform-agnostic) |
| `outreach/message_generator.py` | `OutreachMessageGenerator` — Gemini message generation (platform-agnostic) |
| `outreach/history.py`     | `OutreachHistory` — JSON tracking of contacted handles |
| `core/pipeline.py`        | `ContentPipeline` — normalise → render → upload → publish    |
| `core/orchestrator.py`    | `Orchestrator` + `CampaignResult` — runs campaigns across targets in parallel |
| `core/distributor.py`     | `ContentDistributor` — resolves platforms from `DistributionMap`, delegates to `Orchestrator` |
| `core/registry_builder.py`| `build_registry()` — auto-registers adapters (lazy imports, graceful skip on missing config) |
| `core/text_sanitiser.py`  | `sanitise_text()` — strips em/en-dashes from all text fields (anti-AI-detection) |
| `core/scheduler.py`       | `Scheduler` — in-process scheduled campaign execution        |
| `core/capsule.py`         | `ContentCapsule`, `CapsuleManager` — content capsule CRUD, media management, publication tracking, conversion to content models |
| `core/distribute_cli.py`  | Shared async helper for CLI `distribute` command             |
| `core/prompt_builder.py`  | `PromptBuilder` — composable prompt assembly from persona + function + ICL blocks |
| `core/embedding_store.py` | `EmbeddingStore` — Gemini text-embedding-004 wrapper with batch embed and cosine distance |
| `core/diversity_selector.py` | `select_diverse_examples()` — farthest-point sampling for diverse ICL example selection |
| `core/reel_id.py`           | Word-based reel ID encoding/decoding for discrete tracking in video descriptions |
| `core/feedback/models.py`   | `VideoMetrics`, `CommentData`, `ReelIndexEntry`, `ContentGuideline`, `FeedbackReport` |
| `core/feedback/ports.py`    | `VideoAnalyticsFetcher` protocol                             |
| `core/feedback/sentiment.py`| `SentimentScorer` — Gemini 0-10 comment sentiment analysis   |
| `core/feedback/guideline_generator.py` | `GuidelineGenerator` — avoid/replicate rules from reel performance |
| `core/feedback/orchestrator.py` | `FeedbackOrchestrator` — full fetch→score→analyze→persist cycle |
| `core/feedback/classifier.py` | `TemplateClassifier` — embedding-based retroactive template classification |
| `integrations/langchain.py`| LangChain tool/chain integration                            |

## Pipeline Flow

```
BaseContent
  → ContentNormaliser.normalise()  →  NormalisedContent
  → bundle.renderer.render()      →  NormalisedContent (platform-adapted)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thearnavrustagi/marketmenow](https://github.com/thearnavrustagi/marketmenow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
