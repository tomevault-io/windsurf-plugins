---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 언어 및 커뮤니케이션 규칙
- CRITICAL RULE: All internal thoughts, tool-use reasoning, and explanations before/after running Bash commands MUST be written in Korean (한국어)
- 작업 진행 상황, 스크립트 작성 전후의 설명, 터미널 로그 코멘트 등 혼잣말이나 설명 과정도 예외 없이 반드시 한국어로 출력
- 기본 응답 언어: 한국어
- 코드 주석: 한국어로 작성
- 커밋 메시지: 한국어로 작성
- 문서화: 한국어로 작성
- 변수명/함수명: 영어 (코드 표준 준수)

## 인수인계 문서(Notion) 작성 규칙 — 반드시 준수
이 프로젝트의 인수인계 문서를 Notion에 작성/수정할 때는 아래 규칙을 예외 없이 지킨다.
- 인수인계 문서는 메인 페이지(`https://www.notion.so/38754857e2e880838523c7896e2afab5`)와 그 하위 페이지에만 작성한다
- 문서를 작성/수정하기 전, 현재(최신) 코드를 기준으로 먼저 이 `CLAUDE.md`를 최신화한 뒤 그것을 바탕으로 Notion 페이지를 작성한다
- 모든 서술은 개조식으로 작성하고, 문장 끝에 마침표(`.`)를 찍지 않으며, 이모지를 사용하지 않는다
- 중앙점(`·`)을 사용하지 않는다 — 항목 나열은 쉼표(`,`)로 한다
- 콜아웃(callout) 박스는 전부 회색 바탕(`gray_bg`)으로 통일한다 (다른 색·아이콘 금지)
- 메인 페이지는 시스템 전반(개요·구조·파이프라인)만 정리하고, 세부 사항은 각 단계의 하위 페이지에서 확인하도록 구성한다
- 문서를 업데이트할 때 "업데이트 전 코드 대비 수정 사항"은 본문에 언급하지 않는다 (각 페이지는 최신 상태만 정리)
- 단, 이전 버전 시스템 코드 `iamtest_touch.py` 대비 바뀐 점은 'iamtest_touch.py 대비 변경 이력' 페이지에 모아서 반영한다
- 각 단계 하위 페이지는 "그 단계가 어떤 역할을 하며 내부에서 어떤 일이 벌어지는지"를 코드 기준으로 자세히 정리한다 (변경 이력이 아니라 현재 동작 설명에 집중)

## 이 프로젝트는 무엇인가

한국어 문장을 입력받아 형태소 분석(Kiwi) → 의존구문 분석(자체 학습 KLUE-DP 파인튜닝
SuPar 체크포인트) → 구/절 스팬 추출 → 형태소별 영어 글로싱(규칙 기반 + 사전 +
cross-encoder + LLM 폴백 체인) → 인라인 HTML 시각화(구/절을 색상 막대로 표시)
까지 한 번에 수행하는 Streamlit 앱이다.

## 실행 방법

```bash
streamlit run main.py
```

### API로 실행

`main.py`(Streamlit) 대신 FastAPI 서버로도 실행할 수 있다. 분석 파이프라인 로직은
`api/pipeline.py`가 담당하는데, 이는 `main.py`/`utils/loader.py`의 로직을 Streamlit
런타임 없이 그대로 옮긴 **수동 동기화 사본**이다(import가 아님) — 파이프라인 로직을
고칠 때는 두 파일 모두 반영해야 한다.

```bash
uv run uvicorn api.server:app --host 0.0.0.0 --port 8020
```
프로젝트 루트에서 실행할 필요는 없다(`api/pipeline.py`가 파일 위치 기준 절대경로를 씀).

- `POST /api/analyze` — 요청 바디 `{"text": "..."}`, 응답 `{"results": [...]}`
  (문장 단위로 분석해 배열로 반환, 각 원소는 `main.py`의 `run_analysis()`와 동일한 dict)
- `GET /api/health` — 리소스 로딩 완료 여부(`{"status": "ok"}`/`{"status": "loading"}`)
- 그 외 경로는 `web/` 디렉터리의 정적 프런트엔드(HTML/CSS/JS)를 그대로 서빙한다.

로드되는 모델 아티팩트 (경로는 `main.py` 상단 상수에 하드코딩, 작업 디렉터리 기준 상대 경로):
- `gloss_dict.pkl` — 엑셀 사전(`gloss_dictionary/`의 XLS 3개)에서 빌드된 글로싱 사전
- `klue_roberta_ce_listwise_llrd/` — cross-encoder 모델 + 토크나이저
- `supar_morph_dp/model.pth` — 자체 학습 KLUE-DP 파인튜닝 SuPar 의존구문 분석 모델
- `klue_roberta_large_ner/` — NNP(고유명사) 판별 로컬 백엔드용 모델(`NNP_BACKEND=local`일 때만 로드)

OpenAI 클라이언트는 `.env`의 `OPENAI_API_KEY`로 로드되며 LLM 폴백 글로싱
(`services/openai_client.py`)에 사용된다. `.env`의 `NNP_BACKEND`(`gemini`/`local`)로
고유명사 판별 백엔드를 고르며, `gemini`일 때는 `GEMINI_API_KEY`가 필요하다
(외부 API를 쓸 수 없는 배포 환경에서는 `local`로 설정).

의존성은 `uv`로 관리한다(`uv.lock`, `pyproject.toml`, `requirements.txt`가 동기화되어
있음). Python `>=3.12.0`(`.python-version` 참조). 테스트 스위트나 별도 lint 설정은 없으며,
`ruff`가 의존성 목록에 포함되어 있을 뿐이다.

## 아키텍처

### 파이프라인 (`main.py`가 구동)

0. **NNP(고유명사) 판별** — `kiwi_morphs` 이전에 원문 문장에서 고유명사 스팬을 뽑아
   `detect_nnp(sentence) -> [(start, end, "NNP"), ...]` 형태로 반환한다. `.env`의
   `NNP_BACKEND`로 아래 두 구현 중 하나를 선택(`utils/loader.load_nnp_handler`/
   `api/pipeline.load_all_resources`에서 로드):
   - **`services/gemini_client.GeminiHandler`**(기본값) — Gemini API로 스팬을 탐지한다.
     이름 뒤 호칭성 의존명사(씨/군/양/님/옹)는 스팬 끝에서 잘라내 과잉병합을 막는다.
   - **`services/local_ner_client.LocalNERHandler`** — 외부 API를 쓸 수 없는 배포
     환경(예: 울산대)용. `klue_roberta_large_ner/`(soddokayo/klue-roberta-large-klue-ner
     파인튜닝)로 로컬 추론하며, PS/LC/OG 태그만 채택한다. klue-dev/train-morph.conllu로
     검증한 결과 안전장치 없이 쓰면 오히려 베이스라인보다 정확도가 낮아져(-0.1~0.2%p),
     신뢰도 임계값(0.80)·호칭/익명 접미사 트림(씨/군/양/님/옹/모)·공백 교차 스팬 제외·
     로마자 전용 스팬 제외·1글자 스팬 제외를 거친 스팬만 채택하도록 되어 있다.
   두 구현 모두 실패(키 없음/API 오류/모델 로드 실패) 시 빈 리스트를 반환해 원본 Kiwi
   분석 그대로 진행되도록 한다(교정 실패가 파이프라인을 막지 않음). 새 백엔드를
   추가할 때도 이 `detect_nnp` 인터페이스를 그대로 지켜야 호출부(`main.py`/
   `api/pipeline.py`) 수정 없이 교체할 수 있다.
1. **`core/analyzer.kiwi_morphs`** — Kiwi로 원문을 토크나이즈하여 `(tokens, xpos, spans, lemmas)`를
   얻는다. 0단계에서 얻은 NNP 스팬이 있으면 `pretokenized` 인자로 넘겨 그 구간의
   품사를 NNP로 고정한다. 그 외에는 Kiwi 출력을 그대로 사용한다(과거 끝 글자 `요`를
   `JX`로 강제 분리하던 특수 규칙은 제거됨 — Kiwi는 체언/부사 뒤 `요`를 이미 `요/JX`로
   분리하고 용언 종결형 뒤 `요`는 `어요/네요/세요`처럼 하나의 EF로 통합하므로, 인위적
   분리는 글로싱 사전 매칭만 깨뜨렸다).
2. **`core/analyzer.parse_dep`** — KLUE-DP 파인튜닝 SuPar 모델(`supar_morph_dp/model.pth`)로
   형태소 토큰 시퀀스에 의존구문 분석을 적용하여 `(words, upos, arcs, rels)`를 얻는다.
3. **`span_extraction.morpheme_span_extractor_klue_phase3.MorphemeSpanExtractorKLUE`** — 형태소 단위 arc/rel을
   그대로 사용하여 구/절 스팬을 추출한다(`(tokens, xpos, arcs, rels)` → `list[Span]`).
   어절 단위 압축 없이 형태소 인덱스 공간에서 직접 스팬을 계산한다.
4. **`services/glossing.gloss_sequence_from_tokens`** — 아래 설명하는 계층적 폴백 체인을 통해
   모든 형태소 토큰에 대한 영어 글로스와 메타데이터를 생성한다. 0단계에서 병합된
   NNP 토큰은 `decompose_compound_nnp`가 Kiwi 기본분석으로 재분해를 시도해, 정확히
   2조각(머리+꼬리)이고 꼬리가 사전에 독립 명사로 있으면 "머리글로스 꼬리글로스"로
   합성한다(예: "제주지부" → "jeju branch office") — 병합 자체는 유지하면서
   병합 때문에 사라지는 의미만 글로싱 단계에서 복원한다.
5. **`visualization/renderer.render_phrase_bars`** — 글로싱/스팬 결과를 인라인 HTML로 렌더링한다
   (`config.PHRASES`/`config.CLAUSES`/`SpanLabels` 팔레트에 따라 구/절 종류별로 색상 막대 표시).

### 구/절 스팬 추출 (`span_extraction/morpheme_span_extractor_klue_phase3.py`)

- `MorphemeSpanExtractorKLUE(tokens, xpos, arcs, rels).extract()` — 형태소 인덱스 기반으로
  직접 스팬을 추출한다. ROOT 탐색 후 NPS(주어 명사 나열) 선행 추출 →

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SungmoonJeong/korean_syntax_analysis](https://github.com/SungmoonJeong/korean_syntax_analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
