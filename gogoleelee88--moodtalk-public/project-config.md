---
trigger: always_on
description: 이 파일은 EFT 기반 개인 심리관리 AI 명상앱 개발 프로젝트에 대한 Claude Code 가이드입니다.
---

# CLAUDE.md

이 파일은 EFT 기반 개인 심리관리 AI 명상앱 개발 프로젝트에 대한 Claude Code 가이드입니다.

## 프로젝트 개요

**프로젝트명**: EFT 개인 심리관리 AI 명상앱
**목적**: EFT(감정자유기법) 메뉴얼을 기반으로 한 AI 기반 개인화 심리관리 애플리케이션 개발

## 핵심 기능 정의

### 1. 기본 EFT 기능
- **감정 체크인**: 현재 감정 상태 평가 (1-10 스케일)
- **EFT 탭핑 가이드**: 9개 탭핑 포인트 시각적 안내
- **셋업 구문 생성**: AI 기반 개인화된 셋업 구문 추천
- **세션 진행**: 단계별 EFT 세션 가이드

### 2. 시각적 가이드 시스템
- **애니메이션 탭핑 가이드**:
  - 3D 아바타 모델을 통한 탭핑 포인트 시각화
  - 순차적 포인트 하이라이트 (9개 EFT 포인트)
  - 탭핑 리듬 애니메이션 및 손가락 동작 시각화
  - 포인트별 효과 설명 팝업

- **카메라 기반 AR 가이드**:
  - 실시간 얼굴/신체 인식 (MediaPipe/TensorFlow.js)
  - 사용자 얼굴/몸에 탭핑 포인트 오버레이 표시
  - 손동작 인식으로 올바른 탭핑 검증
  - 실시간 피드백 시스템

### 3. 학습형 AI 상담사 시스템

#### 3.1 티어별 AI 모델 시스템 (2025.08.19 최종 업데이트)

**🔄 AI 시스템 진화:**
- **기존 계획**: 단일 AI 모델 시스템
- **현재 구현**: **티어별 다중 AI 모델 시스템**
- **변경 이유**: DialoGPT 토큰 제한 해결 + 비즈니스 모델 확립

**🔥 Engine A/B 병렬 비교 시스템 (2025.09.09 최종 완성):**
```javascript
const engineABComparisonSystem = {
  // ✅ DialoGPT 완전 폐기! → 무료 사용자도 최신 AI 2개 병렬 비교
  method: "asyncio.gather() 병렬 처리",
  
  engine_a: {
    model: "meta-llama/Meta-Llama-3-8B-Instruct",
    port: 8001,
    provider: "Meta (Facebook)",
    features: "뛰어난 한국어 지원, 논리적 사고",
    target: "정확성 중시 사용자"
  },
  
  engine_b: {
    model: "Qwen/Qwen2.5-7B-Instruct", 
    port: 8002,
    provider: "Alibaba Cloud",
    features: "빠른 응답 속도, 창의적 표현",
    target: "속도 중시 사용자"
  },
  
  // 🚀 병렬 처리 시스템
  parallelProcessing: {
    method: "await asyncio.gather(engine_a_task, engine_b_task)",
    responseComparison: "실시간 성능 비교 (처리 시간, 품질)",
    userExperience: "두 응답 모두 표시 + 더 빠른 모델 하이라이트",
    fallback: "한 모델 실패 시 다른 모델 응답 사용"
  },
  
  // 🔧 기술적 구현
  technicalStack: {
    client: "OpenAI SDK 호환 vLLM 클라이언트",
    server: "FastAPI + httpx.AsyncClient",
    protocol: "OpenAI ChatCompletion API (/v1/chat/completions)",
    timeout: "연결: 10초, 읽기: 120초"
  }
};

// 🎯 사용자 혜택 (2025.09.09)
// - ✅ 무료 사용자도 ChatGPT 수준 AI 2개 동시 비교
// - ✅ 투명한 성능 표시 (어떤 모델이 더 빠른지)
// - ✅ 자동 폴백 (서버 장애 시 다른 모델로 전환)
// - ✅ DialoGPT 대비 10배 이상 품질 향상
```

**⭐ 프리미엄 티어 (PREMIUM):**
```javascript
premiumModel: {
  model: "meta-llama/Llama-3.1-8B-Instruct",
    maxTokens: 8192,          // 8배 확장
    features: "전문 EFT 상담, 긴 대화, 개인화",
    target: "진지한 사용자",
    cost: "월 9,900원"
  },
  
  // 🏢 엔터프라이즈 티어 (ENTERPRISE)
  enterpriseModel: {
    model: "meta-llama/Llama-3.1-70B-Instruct", 
    maxTokens: "unlimited",   // 무제한
    features: "최고급 AI 상담, 실시간 학습",
    target: "기업, 상담센터",
    cost: "월 99,000원"
  }
};
```

**기존 계획 (초기 설계):**
```javascript
// Level 1: 규칙 기반 (즉시 응답, 0ms)
if (간단한_인사 || 기본_감정) {
  return 규칙기반_응답();
}

// Level 2: Transformers.js (중간 복잡도, ~2초)
else if (일반적_상담) {
  return transformersJS_응답();
}

// Level 3: 자체 AI (복잡한 상담, 맞춤형)
else {
  return 학습된_EFT_전문_AI_응답();
}
```

**현재 구현 (2025.09.09 최종):**
```javascript
// Level 1: 규칙 기반 (즉시 응답, 0ms)
if (간단한_인사 || 기본_감정 || 위험상황_감지) {
  return 규칙기반_응답(); // 안전장치 + 기본 응답
}

// Level 2: 고도화된 클라이언트 AI (일반 상담, ~2초)
else if (일반적_상담 || 서버_장애) {
  return aiCompanion_지능형_분석(); // 상황+감정+맥락 종합 분석
}

// Level 3: Engine A/B 병렬 비교 시스템 (메인 AI, ~3초)
else {
  // 🔥 DialoGPT 완전 대체!
  const [llama3_result, qwen25_result] = await Promise.all([
    call_engine_a("meta-llama/Meta-Llama-3-8B-Instruct"),   // 포트 8001
    call_engine_b("Qwen/Qwen2.5-7B-Instruct")              // 포트 8002
  ]);
  
  return {
    both_responses: [llama3_result, qwen25_result],
    faster_model: determine_faster_model(),
    comparison_time: total_processing_time,
    fallback_available: true
  };
}
```

**혁신적 변경 근거 (2025.09.09):**
1. **무료 사용자 경험 혁신**: DialoGPT → 최신 AI 2개 병렬 비교
2. **투명한 성능 공개**: 실시간으로 어떤 모델이 더 빠른지 표시
3. **안정성 극대화**: 한 모델 실패 시 자동으로 다른 모델 사용
4. **품질 혁신**: 구식 DialoGPT 대비 10배 이상 품질 향상
5. **사용자 선택권**: 두 최신 AI 응답을 모두 비교해서 확인 가능

#### 3.2 AI 학습 인프라 구축

**데이터 수집 시스템:**
```javascript
// 사용자 동의 기반 데이터 수집
const dataCollectionConsent = {
  anonymizedData: true,        // 개인정보 제거
  conversationLogs: true,      // 대화 내용 (익명)
  emotionPatterns: true,       // 감정 패턴 분석
  effectivenessMetrics: true,  // 효과성 측정
  optOutAnytime: true         // 언제든 철회 가능
};

// 수집 데이터 구조
const sessionData = {
  sessionId: "sess_12345",
  userId: "anonymized_hash", 
  timestamp: Date.now(),
  conversation: [
    {role: "user", content: "오늘 너무 스트레스받아요"},
    {role: "ai", content: "힘드시겠어요. 어떤 상황인가요?"},
    {role: "user", content: "상사가 계속 야근을 시켜요"}
  ],
  emotionAnalysis: {
    primary: "stress",
    intensity: 8,
    triggers: ["work", "authority_conflict"]
  },
  userFeedback: {
    responseQuality: 4,      // 1-5 평점
    empathyLevel: 5,         // 공감도
    helpfulness: 3,          // 도움 정도
    continueSession: true    // 대화 지속 의향
  },
  eftRecommendation: "stress_reduction_sequence_1",
  effectiveness: {
    beforeMood: 3,
    afterMood: 6,
    improvement: 3,
    techniqueUsed: "crown_chest_breathing"
  }
};
```

**전문가 검증 시스템:**
```javascript
// 전문가 검토 워크플로우
const expertReviewProcess = {
  // 1단계: 자동 필터링
  autoFilter: {
    inappropriateContent: "자동 제거",
    lowQualityResponses: "품질 점수 < 3",
    privacyIssues: "개인정보 포함 세션"
  },
  
  // 2단계: 전문가 검토
  expertReview: {
    psychologist: "심리상담 전문가 검토",
    eftSpecialist: "EFT 기법 전문가 검토", 
    criteriaChecking: {
      empathy: "공감적 응답 여부",
      accuracy: "EFT 기법 정확성",
      safety: "심리적 안전성",
      effectiveness: "실제 도움 정도"
    }
  },
  
  // 3단계: 품질 라벨링
  qualityLabeling: {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gogoleelee88/moodtalk-public](https://github.com/gogoleelee88/moodtalk-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
