---
trigger: always_on
description: Quick reference for AI agents and new contributors.
---

# AGENTS.md — xllama

Quick reference for AI agents and new contributors.

## Conventions

- **Language**: English for code, comments, filenames, commit messages.
- **C++ standard**: C++17 (UWP / SDK 22621 + MSVC).
- **Style**: concise, no over-engineering. Prefer RAII and `unique_ptr`.
- **Formatting**: auto-format on save if possible; otherwise follow existing style.

## Directory structure

```
xllama/
├── include/xllama/          # Shared public headers (WinRT-free, host-testable)
│   ├── inference_params.h   # InferenceParams / InferenceResult
│   ├── inference.h          # run_inference, write_bench_csv
│   ├── training_params.h    # TrainingJob / TrainingCapability (training pillar)
│   ├── training.h           # validate/load job, capability matrix, stage names
│   ├── device_train.h       # Lane B run_device_train_job + progress callbacks
│   ├── personalize.h        # Phase 11: last-block filter, job builder, sample count
│   ├── preference_capture.h # preference JSONL (UI rate + POST /v1/preferences)
│   ├── routing_policy.h     # routing decision + prompt budget (threshold must stay under it)
│   ├── sampling.h           # sampling defaults shared by CLI/bench and GUI/API
│   ├── session.h            # xllama::Session API (persistent model across turns)
│   ├── session_hub.h        # SessionHub: the ONE process-wide resident-Session owner (GUI+API)
│   ├── ort_raii.h           # RAII unique_ptr for OGA* types (UWP/ORT GenAI path)
│   ├── llama_raii.h         # RAII unique_ptr for llama_* types (Linux path)
│   ├── cli.h                # parse_cli_args (Linux)
│   ├── platform.h           # log_output, detect_threads(_llama), peak_working_set_mb
│   ├── path_utils.h         # resolve_model_path, first_gguf_in_dir, model_uses_llama_backend
│   └── utf8_utils.h         # utf8 <-> wstring (Windows)
├── src/bridge/              # Shared implementation (Linux + UWP)
│   ├── inference.cpp        # ORT GenAI and/or llama_decode (unified: runtime dispatch)
│   ├── sampler_chain.h      # add_sampler_stages — the one llama.cpp sampler chain (#125)
│   ├── ort_sampling.h       # apply_ort_sampling — the ORT twin, greedy guard shared (#141)
│   ├── session.cpp          # xllama::Session (OrtSession UWP + LlamaSession Linux)
│   ├── training.cpp         # TrainingJob validate/parse (host + UWP linkable)
│   ├── device_train.cpp     # Lane B engine: prepare → train → export → evaluate
│   ├── personalize.cpp      # Phase 11 pure helpers
│   ├── preference_capture.cpp
│   ├── bench.cpp            # bench CSV writer (incl. run_index)
│   ├── platform.cpp         # log_output (writes xllama.log in UWP)
│   ├── path_utils.cpp       # resolve_model_path: LocalState\models\ + InstalledPath fallback
│   ├── utf8_utils.cpp
│   └── cli.cpp
├── src/main.cpp             # Linux entry point (getopt_long; --train-job)
├── training/                # Training pillar ops: jobs, datasets, host PEFT
├── docs/                    # SSOT map in docs/README.md
│   ├── architecture.md      # System structure SSOT
│   ├── training-architecture.md  # Training SSOT (RE + capability matrix + §11 UI arc)
│   └── api-endpoint.md      # LAN protocol (chat + prefs + train status + images)
├── uwp/                     # C++/WinRT UWP app
│   ├── App.cpp / App.h      # Application::OnLaunched
│   ├── MainPage.cpp / .h    # MainPageController; personalize UI; autopilot
│   ├── inference-bridge.cpp / .h   # main_loop, run_train_job_localized, headless flags
│   ├── api-server.cpp / .h  # opt-in LAN endpoint
│   ├── chat-history.cpp / .h
│   ├── model-downloader.cpp / .h   # catalogue download + LoadModelManifest
│   ├── packages.config      # NuGet pins (versions: packages.config; lifecycle: docs/vendor-lifecycle-plan.md)
│   └── xllama.sln / .vcxproj
├── scripts/
│   ├── deploy.sh                      # Device Portal: deploy, logs, bench trigger
│   ├── build-uwp.ps1                  # Windows UWP packaging script
│   ├── bench-xbox-ort.sh              # benchmark runner (run_index, multi-run)
│   ├── validate-console.sh            # autopilot: the 9 console gates (docs/console-validation-runbook.md)
│   ├── validate-console-training.sh   # rate / serve / device-train
│   ├── validate-api.sh                # LAN: spike|chat|prefs|train|all
│   ├── generate-benchmark-summary.py  # raw results → docs table + dashboard
│   ├── install-latest-build.sh        # fetch + deploy latest CI artifact
│   └── …
├── tests/                   # Unit tests (doctest; incl. test_personalize)
├── bench/                   # configs, raw results, summary policy
├── demo/                    # demo-script.json — what the capture records, reviewable in a PR
├── diffusion/               # SD-Turbo → ONNX host toolchain (not shipped in the MSIX)
├── patches/                 # AppContainer / runtime patches applied at build time
├── cmake/
└── .github/workflows/       # build-linux.yml + build-uwp.yml
```

**Doc ownership (do not invent a second SSOT):** see `docs/README.md`. Structure →
`architecture.md` (incl. catalogue `n_ctx`/`role`, ChatFormat, deferred surfaces);
training → `training-architecture.md`; inventory/status → `model-matrix.md`;
numbers → `bench/results` + generated `benchmarks.md`; UI steps → `using-the-app.md`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gianlucamazza/xllama](https://github.com/gianlucamazza/xllama) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
