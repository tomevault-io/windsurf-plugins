---
trigger: always_on
description: > 이 워크스페이스의 Claude는 **임의 사용자 환경에서 NGC 기반 vLLM 컨테이너와 서빙전략을 생성하는
---

# CLAUDE.md — vLLM 업스트림 추적 · 컨테이너 버전관리 에이전트

> 이 워크스페이스의 Claude는 **임의 사용자 환경에서 NGC 기반 vLLM 컨테이너와 서빙전략을 생성하는
> 이식 가능한 코드 에이전트**다(업스트림 vLLM 릴리즈 추적 → 컨테이너 버전관리 포함).
> 사용자의 노드 구성·네트워크·경로 같은 환경 구체값은 `manifest.yaml`에서 읽는다(테라포밍 스킬이 생성).
> 이 파일은 세션마다 로드되는 "항상 들고 있어야 할 사실"만 담는다.
> 다단계 절차(전파 워크플로)는 `.claude/rules/workflow.md`에 있다.
> 근거: Ouroboros 인터뷰 → Seed (비추적 `seed/` 참조 — 배포본엔 부재 가능).

## 정체성

- **업스트림(진실의 원천)**: vLLM — https://github.com/vllm-project/vllm
  - "버전" = GitHub Release 태그. **pre-release도 추적 대상**.
- **관리 대상**: 커스텀 Docker 컨테이너 = NGC PyTorch 베이스 + vLLM(**prebuilt wheel 또는 소스빌드**) + 주변 의존성.
- **토폴로지**: 사용자 환경에 따라 단일노드 또는 분산(다노드)으로 결정된다. 노드 수·역할(main/sub)·인터커넥트는
  헌법에 박지 않고 `manifest.yaml`의 `topology`·`nodes[]`에서 읽는다(테라포밍이 자동탐지+인터뷰로 채움).
- **브랜치**: `single-node` = 단일노드 전용 · `multi-node` = 분산 전용. 두 브랜치는 버전 핀이 독립.

## 레이어드 적응 원칙 (기초레이어 우선 — 가장 먼저)

- **헌법(철학)=기초레이어=단일 진실원천.** 기초레이어 규정이 바뀌면 상위레이어(스킬·workflow·recipe·산출물)는
  그 변경에 **적응패치**해 정합을 회복한다(역방향 금지 — 상위 편의가 헌법을 흔들지 않음).
- 운영: 기초레이어 변경 시 **상위 전 레이어 conformance 스윕**(부분수정 방치 금지 — 정합 회복까지가 1건).
  근거: `docs/plan/plan_2026062711_1`(첫 적용 = 모델구동 패치 + KV 이식성).

## 핵심 사실 (항상 보유)

- **레이어 커플링 규칙(가장 중요)**: 대상 vLLM 버전의 `pyproject.toml` `[build-system].requires`에
  **명시된 torch 버전**을 먼저 확인하고, 그 torch 버전과 NGC 컨테이너의 `NVIDIA_PYTORCH_BUILD_VERSION`
  **접두어**가 일치하는 NGC PyTorch 베이스 태그를 선정한다.
  (접미어 `+해시`·빌드메타는 무시. 예: vLLM 0.21.0 → torch 2.11.0 → `nvcr.io/nvidia/pytorch:26.03-py3`)
- **빌드트랙 따름정리**: torch 2.10대 → prebuilt wheel · torch 2.11+ → NGC alpha와 prebuilt `_C`의 C++ ABI 충돌(하드 ABI 벽) → **소스빌드 1차 트랙**. 트랙 판정 = 스킬 `upstream-version-watch` §0.5, 최종 중재 = 스모크.
- **커플링 보강 원칙**: prefix-매칭은 필요조건일 뿐 — source-build에서 alpha 베이스가 stable-ABI 심볼 결여 시 **더 새 NGC 베이스 승격 정당**(전방호환). 절차·키잉 상세 = `.claude/rules/workflow.md` S3 · 스킬 §4.6.
- **이미지 네이밍 불변식**: `easy-vllm:{vllm}-cu{cuda}-{arch}-{track}`(예 `0.23.0-cu132-aarch64-source`). 모델-키잉 금지(과거 난립 원인) — 한 이미지가 모든 모델을 서빙. 태그 산정 = `render_dockerfile.py`.
- **아치-enablement 변종 트랙 따름정리 (도커 패치 범위 상한 = 포크 핀)**: stock vLLM이 **구조적으로 불가**할 때(arch-wall — 예 GB10 sm_121 DeepSeek-V4: 어텐션 major∈[9,10] + MXFP4 오라클 MARLIN-only) 도커 패치 범위는 **deps-패치 → 소스-게이트 패치 → vLLM 소스-repo 오버라이드(포크 SHA 핀) → (포크도 불가 시) 체크포인트-교체** 사다리로 확장한다. repo 오버라이드는 `VLLM_REPO`/`VLLM_REF` build-arg(기본=stock)로 같은 Dockerfile이 분기 → `…-source-sm12x` 같은 **superset 변종 이미지**(기존 모델 유지+추가, 제거 ✗·모델-키잉 ✗). 거버넌스 = NGC 베이스 오버라이드와 동일 1급 HITL 핀-오버라이드(참조-그라운디드·증거·testlog·HITL → `resolved.json` `source_build_variants`). 규율: **단일 변종-트랙**(난립 금지)·**승격 전 기존모델 회귀 재스모크**(포크는 stock과 diverged). 절차 = `.claude/rules/workflow.md` S3.
- **변종이미지 build-plane ≠ serve-plane 따름정리**: 변종 이미지는 언제든 발생 가능 → **이미지 정체성(IMAGE_TAG·`VLLM_REPO`/`VLLM_REF` 빌드-arg)=클러스터-와이드 Band2**(멀티노드 클러스터가 한 이미지 — 슬레이브 compose 보간에도 전달), **모델 serve config(CONFIG_FILE·트리플렛)=Band3**. ∴ **슬레이브 Band2-only는 *serve-plane* 불변식이지 *build-plane*이 아니다**(슬레이브는 모델 트리플렛 없이도 변종 이미지를 빌드·기동해야 함 — `multinode_serve_smoke.sh`가 IMAGE_TAG/REPO/REF만 슬레이브에 전달, CONFIG_FILE 미전달로 Band2-only 보존). 절차 = workflow.md S2.5.
- **산출물 통로 불변식 (single/multi 혼재 차단)**: 빌드/렌더 산출물(Dockerfile·compose·requirements·모델 configs·envs·**manifest 실값**)은 **`output/<topology>/`(single|multi)** 에 둔다. **통로 껍데기(`.gitkeep`)만 추적·생성물 비추적** → 단일/멀티 산출물이 켜켜이 쌓여도 경로 격리로 서로 침범 못 함. **topology 는 브랜치가 결정**(single-node=single, multi-node=multi) → manifest 실값도 `output/<topology>/manifest.yaml` 통로 분리, **브랜치 빈번 전환 시 재작성 0**(전환=그 통로 manifest를 읽음). 빌드 = `docker compose -f output/<topology>/docker-compose.yaml …`. 예외: multi 손작성 컨테이너 정의는 `*.template` 졸업 전까지 추적(정본 — Plan 2서 ignore 강등). 근거: `docs/plan/plan_2026062312_1`(통로)·`plan_2026062315_1`(manifest 이관).
- **serve-time env 통로 불변식 (결함#2 codify)**: serve 변수치환값(`NAS_MODEL_PATH`·`TIKTOKEN_HOST_PATH`) **해소 우선순위 = env-주입 > manifest 정본 > 리터럴 default**(포인터 원칙 연장 — `check_smoke_model.py`·render materialize 동일). materialize 절차(`render_dockerfile.py --materialize-env` → `output/<topology>/.env`) 상세 = 스킬 `upstream-version-watch` §2.5.
- **KV 절대클램프 따름정리 (이식성)**: 최종 recipe 의 KV 는 **측정된 GPU당 `kv-cache-memory-bytes`(절대값)** 로 제어한다 — 이게 **이식성**을 준다(gmu-derived KV 는 호스트 VRAM 차이로 비이식·OOM). **단 `gpu-memory-utilization` 도 함께 emit**(E2E 실증): vLLM 은 클램프 시 gmu 를 *KV 사이징*에만 무시(cache.py)하고 **startup free-memory 검증(free ≥ gmu×total)+총-cap 엔 여전히 사용** → 통합메모리(GB10 free/total≈0.91)는 기본 0.92 가 startup OOM 이라 **gmu ≤ 0.90 명시 필수**. ∴ gmu=startup/cap 게이트, clamp=KV·이식성. 시뮬레이터의 3종 산출물 = "**타겟 GPU 구동가능 환경의 시뮬레이션**". 이식성 = 선언된 절대 필요량 이상 GPU서 동일 구동(작은 GPU 자동맞춤 ✗, GPU당 값이라 TP 의존). 상세 = 스킬 `vllm-recipe-explorer` §5 · `plan_2026062711_1` Part 2.
- **모델구동 런타임 패치 따름정리**: 구동 불가 모델(포크-시대 VL processor 불일치 등)의 런타임 호환은 **stock 이미지 + 런타임 패치**로 해결한다(빌드타임 파생-이미지 금지 — 이미지 네이밍 불변식). 패치 내용 = **메인 저작 빌딩블럭**(서브 저작 ✗·하향 배달), **확률론·휘발**(에이전트가 참조-그라운디드로 환경·bump 마다 재유도 · 비추적 `<model>_patch.py` · 카탈로그 없음 · carry-forward ✗). **결정론 메커니즘 = arming**(serve_runner/단일진입의 `<model>_patch.py` 자동탐지 .pth → 모든 프로세스 적용; 메인 저작). 정본 = 방법론(스킬)+지식(docs). 단 source-build 패치는 *빌드타임*이라 추적 `Dockerfile.source-build`에 동결(평면별 지속성 비대칭). 상세 = 스킬 `vllm-recipe-explorer` §5 · `plan_2026062711_1` Part 1·3.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tbvjvsladla/easy_vllm_simulator](https://github.com/tbvjvsladla/easy_vllm_simulator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
