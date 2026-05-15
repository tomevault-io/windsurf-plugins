---
trigger: always_on
description: Multiturn Review System - Context limit 초과 시 프롬프트 분할 및 결과 합성 시스템
---


# 멀티턴 리뷰 시스템 (Multiturn Review System)

## 시스템 개요

멀티턴 리뷰 시스템은 LLM API의 컨텍스트 제한(context limit)을 초과하는 대용량 코드 리뷰 요청을 자동으로 처리하는 시스템입니다. 컨텍스트 제한 초과 에러가 감지되면 자동으로 작동하여 프롬프트를 여러 청크로 분할하고, 각 청크를 순차적으로 처리한 후 결과를 지능적으로 합성하여 하나의 통합된 리뷰 결과를 제공합니다.

### 주요 특징

- **자동 감지**: error_patterns.yml 기반 컨텍스트 제한 초과 에러 자동 감지
- **지능적 분할**: 토큰 정보와 텍스트 길이를 고려한 최적 프롬프트 분할
- **순차 처리**: OpenRouter 동시성 문제 해결을 위한 안정적인 순차 API 호출
- **LLM 합성**: Summary는 LLM을 활용한 지능적 합성, fallback 로직 제공
- **비용 투명성**: 전체 청크 처리 비용과 합성 비용의 정확한 계산 및 표시

## 핵심 컴포넌트 및 아키텍처

```
CLI Error Handler
    ↓
MultiturnReviewExecutor (메인 조정자)
    ├── PromptSplitter (프롬프트 분할)
    └── ReviewSynthesizer (결과 합성)
            ├── SynthesisAPIClient (API 호출)
            └── SynthesisPromptManager (프롬프트 관리)
```

### 1. MultiturnReviewExecutor

**역할**: 멀티턴 리뷰 프로세스의 메인 조정자
**위치**: `selvage/src/multiturn/multiturn_review_executor.py`

```python
class MultiturnReviewExecutor:
    def execute_multiturn_review(
        self,
        review_prompt: ReviewPromptWithFileContent,
        token_info: TokenInfo,
        llm_gateway: BaseGateway,
    ) -> ReviewResult:
        # 1. 프롬프트 분할
        # 2. 순차 API 호출
        # 3. 결과 합성
```

**주요 기능**:
- PromptSplitter를 통한 user_prompts 분할
- 분할된 청크들의 순차적 LLM API 호출
- ReviewSynthesizer를 통한 결과 합성 조정

### 2. PromptSplitter

**역할**: 토큰 제한에 맞춰 프롬프트를 최적으로 분할
**위치**: `selvage/src/multiturn/prompt_splitter.py`

**분할 전략**:
1. **토큰 정보 기반 분할**: actual_tokens와 max_tokens 비율로 청크 수 계산
2. **텍스트 길이 기반 균등 분배**: 그리디 알고리즘으로 각 청크의 크기 균등화
3. **안전 여유분 확보**: max_tokens의 80%를 안전 기준으로 설정

```python
# 분할 비율 계산 예시
safe_max_tokens = max_tokens * 0.8
raw_split_ratio = actual_tokens / safe_max_tokens
split_ratio = max(2, math.ceil(raw_split_ratio))
```

### 3. ReviewSynthesizer

**역할**: 여러 리뷰 결과를 하나의 통합된 결과로 합성
**위치**: `selvage/src/multiturn/review_synthesizer.py`

**합성 전략**:
- **Issues**: 단순 합산 (정보 손실 방지)
- **Summary**: LLM 합성 시도 → 실패 시 fallback (가장 긴 summary 선택)
- **Recommendations**: 단순 합산 + 완전 동일 항목만 중복 제거
- **Score**: 첫 번째 결과의 점수 사용

### 4. SynthesisAPIClient

**역할**: 모든 LLM 프로바이더에 대한 통합 합성 API 호출
**위치**: `selvage/src/multiturn/synthesis_api_client.py`

**지원 프로바이더**:
- OpenAI (Instructor 사용)
- Anthropic Claude (일반/thinking 모드 지원)
- Google Gemini (직접 SDK 사용)
- OpenRouter (JSON Schema 지원)

### 5. SynthesisPromptManager

**역할**: 합성 작업별 프롬프트 관리
**위치**: `selvage/src/multiturn/synthesis_prompt_manager.py`

**지원 작업**:
- summary_synthesis: Summary 합성 전용 프롬프트
- recommendation_synthesis: Recommendations 합성 전용 프롬프트

## 상세 워크플로우

### 1. 에러 감지 및 분석
```python
# cli.py:L354-356
if error_response.is_context_limit_error():
    return _handle_context_limit_error(
        review_prompt, error_response, llm_gateway
    )
```

**에러 패턴 매칭**: `selvage/resources/error_patterns.yml`에 정의된 패턴으로 context_limit_exceeded 에러 감지

**프로바이더별 에러 패턴**:
- **OpenAI**: "context_length_exceeded"
- **Anthropic**: "prompt is too long", "input length"
- **OpenRouter**: "maximum context length"

### 2. 토큰 정보 추출
```python
token_info = TokenInfo.from_error_response(error_response)
```

**추출 정보**:
- `actual_tokens`: 실제 사용한 토큰 수
- `max_tokens`: 최대 허용 토큰 수

### 3. 프롬프트 분할 실행
```python
user_prompt_chunks = self.prompt_splitter.split_user_prompts(
    user_prompts=review_prompt.user_prompts,
    actual_tokens=token_info.actual_tokens,
    max_tokens=token_info.max_tokens,
)
```

**분할 과정**:
1. 분할 비율 계산 (안전 여유분 20% 확보)
2. 각 UserPromptWithFileContent의 추정 크기 계산
3. 크기순 정렬 후 그리디 알고리즘으로 균등 분배

### 4. 순차 API 호출
```python
review_results = self._execute_sequential_reviews(
    user_prompt_chunks, review_prompt.system_prompt, llm_gateway
)
```

**순차 처리 이유**: OpenRouter의 동시성 문제로 인해 병렬 처리에서 순차 처리로 전환 (병렬 버전은 deprecated)

### 5. 결과 합성
```python
synthesizer = ReviewSynthesizer(llm_gateway.get_model_name())
merged_result = synthesizer.synthesize_review_results(review_results)
```

**합성 과정**:
1. 성공한 결과들만 추출
2. Issues 단순 합산
3. Summary LLM 합성 (실패 시 fallback)
4. Recommendations 합산 및 중복 제거
5. 비용 계산 (기존 + 합성 비용)

### 6. Summary LLM 합성 상세
```python
# ReviewSynthesizer._synthesize_summary_with_llm()
synthesis_data = {
    "task": "summary_synthesis",
    "summaries": summaries,
}
system_prompt = self.prompt_manager.get_system_prompt_for_task("summary_synthesis")
structured_response, estimated_cost = self.api_client.execute_synthesis(
    synthesis_data, SummarySynthesisResponse, system_prompt
)
```

### 7. 비용 계산
```python
total_cost = self._calculate_total_cost(
    successful_results, summary_synthesis_cost, None
)
```

**비용 구성**:
- 각 청크 처리 비용의 합산
- Summary 합성 비용 (LLM 호출 시)
- Recommendations 합성 비용 (구현되어 있으나 현재 미사용)

## 에러 처리 및 토큰 추출 메커니즘

### error_patterns.yml 구조
```yaml
providers:
  openai:
    patterns:
      context_limit_exceeded:
        keywords: ["context_length_exceeded"]
        message_patterns:
          - regex: "This model's maximum context length is (\\d+(?:,\\d+)*) tokens\\. However, your messages resulted in (\\d+(?:,\\d+)*) tokens"
            extract_tokens:
              max_tokens: 1 # 첫 번째 캡처 그룹
              actual_tokens: 2 # 두 번째 캡처 그룹
```

### TokenInfo 모델
```python
@dataclass
class TokenInfo:
    actual_tokens: int | None
    max_tokens: int | None
    
    @classmethod
    def from_error_response(cls, error_response: ErrorResponse) -> "TokenInfo":

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [selvage-lab/selvage](https://github.com/selvage-lab/selvage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
