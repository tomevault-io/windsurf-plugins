---
trigger: always_on
description: 이 파일은 LLM이 이 프로젝트의 지식 저장소를 체계적으로 관리하기 위한 스키마 레이어다.
---

# AGENTS.md

이 파일은 LLM이 이 프로젝트의 지식 저장소를 체계적으로 관리하기 위한 스키마 레이어다.
사용자와 LLM이 시간이 지남에 따라 함께 진화시킨다.

---

## 프로젝트 개요

SID(Semantic ID) 기반 training-free 추천 시스템을 로컬 환경에서 실험하기 위한 Python 코드베이스다.

- **로컬 추론**: Apple Silicon MLX (`mlx-lm`, `mlx-embeddings`)
- **생성 모델**: `mlx-community/Qwen3.5-9B-OptiQ-4bit`
- **임베딩 모델**: `mlx-community/Qwen3-Embedding-4B-4bit-DWQ`
- **패키지 관리**: `uv`
- **CLI**: `typer` + `rich`

### 주요 모듈

| 모듈 | 경로 | 역할 |
|------|------|------|
| config | `src/sid_reco/config.py` | 환경 변수 + `.env` 기반 설정, 경로 해석 |
| llm | `src/sid_reco/llm.py` | `MLXTextGenerator` — 로컬 생성 추론 |
| embedding | `src/sid_reco/embedding.py` | `MLXEmbeddingEncoder` — 로컬 임베딩 추론 |
| mlx_runtime | `src/sid_reco/mlx_runtime.py` | MLX 런타임 probe 및 지원 환경 진단 |
| datasets | `src/sid_reco/datasets/` | 데이터셋 로더 및 전처리 (`foodcom.py`) |
| taxonomy | `src/sid_reco/taxonomy/` | neighbor context 임베딩/FAISS 검색, taxonomy dictionary 생성, item-level taxonomy structuring (`item_projection.py`) |
| sid | `src/sid_reco/sid/` | Phase 1 마지막 단계용 structured item serialization, MLX embedding, CPU residual K-means compiler, FAISS indexing, recommendation statistics 생성 (`serialization.py`, `embed_backend.py`, `compiler.py`, `indexing.py`, `stats.py`) |
| recommendation | `src/sid_reco/recommendation/` | Phase 2 recommendation pipeline: interest sketch, semantic retrieval, bootstrap rerank, confidence aggregation, grounding |
| cli | `src/sid_reco/cli.py` | `doctor`, `smoke-mlx`, `smoke-llm`, `smoke-embed`, `prepare-foodcom`, `build-neighbor-context`, `build-taxonomy-dictionary`, `structure-taxonomy-item`, `structure-taxonomy-batch`, `compile-sid-index`, `recommend` |

### 빌드 및 검증 명령

```bash
uv sync --all-groups          # 의존성 설치
uv run pytest                 # 테스트
uv run ruff check .           # 린트
uv run mypy src               # 타입체크
uv run sid-reco doctor        # 환경 상태 확인
uv run sid-reco smoke-mlx     # MLX 런타임 진단
uv run sid-reco recommend --help
uv run sid-reco build-neighbor-context --help
uv run sid-reco build-taxonomy-dictionary --help
uv run sid-reco structure-taxonomy-item --help
uv run sid-reco structure-taxonomy-batch --help
uv run sid-reco compile-sid-index --help
```

### Repo-local Codex Commands

이 저장소의 repo-local Codex 확장점은 `.agents/skills/`다.
공식 Codex App built-in slash command와는 별개로, enabled skill은 slash 목록에 나타날 수 있다.

- `/docs-manager` 또는 `/doc-manager` — 위키/ADR/인덱스 작업과 `README.md`, `AGENTS.md`, `.github/copilot-instructions.md`, `.harness/reference/local-adaptation.md` 동기화를 포함한 문서 반영 루틴
- `/spec` — 구현 전에 `SPEC.md`를 정리하는 spec-driven workflow
- `/plan` — `tasks/plan.md`, `tasks/todo.md` 기반 작업 분해
- `/build` — incremental-implementation + TDD 기반 구현 흐름
- `/test` — TDD / Prove-It 기반 검증 흐름
- built-in `/review` 또는 `$code-review-and-quality` — 5축 코드 리뷰
- `/code-simplify` — 동작 보존 단순화 검토
- `/ship` — 배포/릴리스 전 체크리스트

---

## 개발 워크플로 스킬 레이어

이 저장소에는 범용 엔지니어링 워크플로 스킬 레이어도 함께 존재한다.

- 활성 스킬 위치: `.agents/skills/`
- 하네스 support 자산 루트: `.harness/`
- 체크리스트 참조: `references/`
- upstream 문서 스냅샷: `.harness/reference/`
- 훅 스냅샷: `.harness/hooks/`
- archived command prompt 초안: `.harness/reference/command-drafts/`
- archived persona markdown: `.harness/reference/agent-personas/`

운영 원칙:

1. `docs/sources/`, `docs/wiki/`, ADR/교차 참조/인덱스 작업은 항상 `docs-manager`와 이 `AGENTS.md` 스키마가 우선한다.
2. 코드 구현, 테스트, 리뷰, 릴리스 흐름은 imported agent skills를 사용할 수 있다.
3. imported skill의 일반 예시가 이 저장소 구조와 충돌하면 `.harness/reference/local-adaptation.md` 규칙을 우선한다.
4. 위키 문서는 계속 한국어로 유지한다.

---

## 3-레이어 아키텍처

이 프로젝트의 지식 저장소는 3개 레이어로 구성된다.

```
.
├── AGENTS.md                          ← 스키마 레이어 (이 파일)
└── docs/
    ├── sources/                       ← 원문 소스 레이어 (불변)
    │   ├── papers/                    SID·추천 관련 논문
    │   ├── datasets/                  데이터셋 원본 문서·스키마
    │   ├── models/                    모델 공식 문서·벤치마크
    │   └── experiments/               실험 로그·결과 데이터
    └── wiki/                          ← 위키 레이어 (LLM 소유)
        ├── INDEX.md                   전체 페이지 목록
        ├── summaries/                 소스별 요약
        ├── entities/                  구체적 대상 (모델, 데이터셋 등)
        ├── concepts/                  추상 개념 (SID, Training-Free 등)
        ├── comparisons/               비교·분석
        ├── overviews/                 주제별 합성 개요
        ├── decisions/                 ADR (Architecture Decision Record)
        └── logs/                      인제스트 건별 로그
```

### 레이어 1: 원문 소스 (`docs/sources/`)

**진실의 원천(Source of Truth)**이다.

- 변경 불가(immutable) — LLM은 읽기만 하고 수정하지 않는다
- 사용자가 큐레이션한 소스 문서를 저장한다: 논문, 데이터셋 문서, 모델 자료, 실험 로그
- 하위 카테고리:
  - `papers/` — 학술 논문 PDF 또는 텍스트 요약
  - `datasets/` — 데이터셋 스키마, README, 출처 문서
  - `models/` — 모델 카드, 벤치마크 발췌, 공식 문서
  - `experiments/` — 실험 설정, 결과 데이터, 로그 (기록 후 불변)

### 레이어 2: 위키 (`docs/wiki/`)

**LLM이 완전히 소유**하는 마크다운 파일 디렉터리다.

- LLM이 페이지 생성, 업데이트, 교차 참조 유지를 담당한다
- 사용자는 읽기만 한다
- 모든 페이지에 YAML 프론트매터가 필수다

### 레이어 3: 스키마 (이 파일)

LLM에게 위키 구조, 컨벤션, 워크플로를 알려주는 설정 문서다.
사용자와 LLM이 시간이 지남에 따라 함께 진화시킨다.

---

## 위키 페이지 타입

### 1. 요약 (Summary)

소스 1개를 읽고 핵심 내용을 정리한 페이지다.

**프론트매터:**

```yaml
---
title: "<소스 제목 또는 핵심 주제>"
date: YYYY-MM-DD
type: summary
sources:
  - docs/sources/<category>/<filename>
---
```

**본문 구조:**

```markdown
# <제목>

## 출처 정보
- 저자, 발행일, 출처 유형 등

## 핵심 주장/발견
- 소스의 주요 논점, 결과, 발견 사항

## 방법론
- 사용된 접근법, 실험 설계 (해당되는 경우)

## 한계
- 저자가 밝힌 한계 또는 분석 상 한계

## Related

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pko89403) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
