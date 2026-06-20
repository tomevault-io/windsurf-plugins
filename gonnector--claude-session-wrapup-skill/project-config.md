---
trigger: always_on
description: 세션 마무리, 세션 정리, 세션 래핑, 세션 요약, 배운 점 정리, lesson learned, wrapup, 오늘 뭘 배웠지, 세션 회고, 작업 마무리 시 자동으로 추천됩니다. 세션 종료 시 사용자와 AI 각각의 '배운 것'을 구조화된 JSONL로 자동 기록합니다.
---


# /wrapup

세션 종료 시 2계층(세션 요약 + Lesson-Learned)을 한 번에 기록하는 스킬.

- 스키마 상세: `references/schema.md` 참조
- 저장 스크립트: `scripts/save-wrapup.py`
- 통계 스크립트: `scripts/read-stats.py`

스크립트 경로 규칙: 이 스킬의 base directory를 `$SKILL_DIR`로 참조한다.
실행 시 `python "$SKILL_DIR/scripts/save-wrapup.py"` 형태로 **절대 경로** 사용.

**CRITICAL — AskUserQuestion 호출 규칙:**
`AskUserQuestion`은 built-in 도구이다. **ToolSearch로 찾지 말고 직접 호출할 것.**
ToolSearch에서 AskUserQuestion이 검색되지 않는 것은 정상 — deferred tool이 아니라 항상 사용 가능한 내장 도구이기 때문이다.
이 스킬의 모든 AskUserQuestion 호출(Step 0/3/5/6d/8)은 **ToolSearch 없이 직접 실행**한다.

**채널 모드(`--channels`) fallback:**
`--channels` 플래그가 활성화된 세션에서는 AskUserQuestion이 **비활성화**된다 (Claude Code v2.1.80+).
이유: AskUserQuestion이 터미널을 블로킹하면 채널 메시지 수신/응답이 불가하기 때문.
- AskUserQuestion 호출이 실패하거나 도구가 없으면 → **text-based fallback** 사용
- 선택지를 텍스트로 번호 목록으로 표시하고 사용자의 번호/텍스트 입력을 받는다
- 이 fallback은 UI 품질은 떨어지지만 기능적으로 동일한 결과를 보장한다
- Anthropic이 AskUserQuestion의 채널 릴레이를 구현하면 이 fallback은 제거 예정

## 언어별 변경 힌트 매핑

| 언어 코드 | language_name | language_label | change_hint |
|-----------|---------------|----------------|-------------|
| ko | 한국어 | 랩업 언어 | `/wrapup 언어 변경` |
| en | English | Wrap-up Language | `/wrapup change lang` |
| ja | 日本語 | ラップアップ言語 | `/wrapup 言語変更` |
| zh-cn / zh | 中文(简体) | 摘要语言 | `/wrapup 切换语言` |
| 기타 | (직접 입력) | Wrap-up Language | `/wrapup change lang` |

---

## 워크플로우

아래 10단계를 순서대로 실행한다.

### Step 0: 언어 설정 확인

**사용자 입력에 "언어 변경", "change lang", "change language", "言語変更", "切换语言" 중 하나라도 포함되어 있으면 → Step 0-B로 이동.**

**Step 0-A: 기존 설정 읽기 (일반 실행)**

```bash
python "$SKILL_DIR/scripts/settings.py" read
```

- 결과에 `"initialized": true` → `language`, `language_name`, `change_hint` 값을 기억한다 → Step 1로 **무음** 진행
- 결과가 `{}` 또는 파일 없음 → Step 0-C

**Step 0-B: 언어 변경 (키워드 트리거)**

AskUserQuestion으로 언어 선택 (4개 고정 옵션 + Other):
- 한국어 (ko)
- English (en)
- 日本語 (ja)
- 中文(简体) (zh-cn)

선택 후:
```bash
python "$SKILL_DIR/scripts/settings.py" write --lang {code} --name "{name}"
```

"{name}(으)로 언어가 설정되었습니다." 안내 → Step 1로 진행.

**Step 0-C: 최초 설정 (설정 파일 없을 때)**

시스템 언어 감지:
```bash
python "$SKILL_DIR/scripts/settings.py" detect
```

AskUserQuestion 1개:
"Select the language for wrapup records. (Detected: {detected_name})"
- {detected_name} 사용 (Recommended)
- English (en)
- 한국어 (ko)
- 日本語 (ja)
- 中文(简体) (zh-cn)

선택 후 write → `language`, `language_name`, `change_hint` 기억 → Step 1로 진행.

---

### Step 1: 세션 메타정보 수집

**단일 호출로 모든 메타정보를 수집한다:**

```bash
python "$SKILL_DIR/scripts/collect-meta.py"
```

반환 JSON 필드:
- `date` — 현재 시각 (ISO 8601)
- `project` — 프로젝트 경로
- `agent` — 에이전트명 (소문자) 또는 `null` (비에이전트 세션). `CLAUDE_AGENT_NAME` 환경변수에서 자동 추출
- `session_id` — 세션 UUID
- `session_name` — 세션명 (`/rename`으로 설정한 이름)
- `timing` — 세션 시간 측정 (`session_start`, `wrapup_start`, `segment_start`, `elapsed_minutes`, `is_continuation`, `wrapup_number`)
- `stats` — 누적 통계 (user_lessons / ai_lessons / session_summaries)
  - `stats.session_summaries.total` — 현재 프로젝트 랩업 수 (에이전트 세션이면 해당 에이전트 폴더 기준)
  - `stats.session_summaries.global_total` — 전체 프로젝트 합산 랩업 수 (기존 경로 + 모든 에이전트 폴더)
  - `stats.session_summaries.total_elapsed_minutes` — 전체 누적 협업 시간 (분)

`session_name`이 빈 문자열이면 **중단** → "세션명이 설정되지 않았습니다. `/rename 세션명`으로 설정 후 다시 시도해주세요." 안내

### Step 2: 대화 컨텍스트 분석 → 2계층 드래프트 생성

**언어 규칙: 모든 내용(info, qa, conclusions, actions, lesson 제목/요약 등)은 Step 0에서 확인된 언어(`{language_name}`)로 작성한다. 대화가 다른 언어로 진행되었더라도 설정 언어로 번역하여 기록한다.**

**Auto Memory 중복 확인 (선행 작업):**

드래프트 생성 전, 현재 프로젝트의 auto memory 디렉토리를 Glob/Read 도구로 스캔한다:
- 경로: `~/.claude/projects/{project-slug}/memory/*.md`
- `{project-slug}`는 Step 1의 `project` 값에서 경로 구분자(`\`, `/`, `:`)를 `-`로 치환한 값
- auto memory 파일이 존재하면 내용을 읽어 세션 중 기록된 항목 목록을 파악한다
- 이후 Lesson-Learned 추출 시 auto memory에 이미 기록된 **사실(fact)과 동일한 내용**은 lesson에서 `[📝 auto memory]` 태그를 붙여 중복임을 표시한다
- auto memory가 사실만 기록했다면, lesson은 **맥락(context)과 발견 과정** 중심으로 경량화한다
- auto memory 디렉토리가 없거나 파일이 없으면 이 단계를 건너뛴다

전체 대화 컨텍스트에서 아래 항목을 추출한다.

**계층 1 — 세션 요약:**
- A. **정보 요약**: 세션에서 조사/파악된 핵심 정보 (불릿 포인트)
- B. **Q&A 정리**: 사용자 질문 + AI 답변 쌍
- C. **협의 결론**: 토론/비교된 안건의 결론 + 이유(rationale)
- D. **작업 내역**: 세션 중 실제로 수행한 작업 목록 (구현, 파일 수정, 버그 수정, 테스트, 문서 작업 등). Q&A나 토론이 아닌 실제 실행된 행위.
- E. **액션 아이템**: 후속 할 일 목록 (priority: high/medium/low)

**계층 2 — Lesson-Learned:**

**사용자 학습 탐지 — Q: 이 대화에서 사용자가 무언가를 알게 됐는가?**
- What/Who/When/Where 질문→AI 답변 패턴 → type: `user_fact_question`
- Why/How/비교/분석 질문→AI 답변 패턴 → type: `user_concept_question`
- 통찰/깨달음 표현 → type: `user_insight_feedback`
  - [확실한 신호] "새로 알게 됐다", "깨달았다", "처음 알았어", "이건 몰랐는데"
    "이렇게 되는 거구나", "그래서 ~이었구나", "결국 ~이구나"
  - [후속 발화와 함께] -군(요)/-네(요) 어미, "아!"/"오!" — 단독으로는 백채널과 구분 불가
- 기존 관점/전제가 틀렸음을 발견 → type: `user_perspective_shift`
  "이렇게 생각하면 안 되겠다", "내가 잘못 알고 있었네", "완전히 다르게 봐야겠어"

**AI 학습 탐지 — Q: 이 대화에서 AI가 무언가를 알게 됐거나, 방식을 바꿨는가?**
- 에러/오답 → 수정 패턴 → type: `ai_trial_error`
- 새로운 정보/맥락 발견 → type: `ai_research_discovery`
- 접근 방식 전환 → type: `ai_strategy_pivot`
- 사용자가 AI에게 가르치거나 방향을 지정 → type: `ai_user_guided`
  - 명시적 교정: "아니야", "틀렸어", "실제로는 ~야"
  - 도메인 주입: "내 상황은 ~이야", 배경/전문 지식 제공
  - 가이드라인: "~스타일로", "~는 하지 마"
  - 방식 교정: 출력 포맷/어조/수준 재지시

양쪽 모두 해당하면 양쪽 다 기록 (같은 내용이지만 관점이 다름).

각 lesson에 `category`와 `tags`를 부여한다.

**계층 3 — 후속 작업 추천:**

세션에서 완료한 작업을 바탕으로, Dylan에게 도움이 될 만한 연계 태스크를 3개 추천한다.

추천 조건:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gonnector/claude-session-wrapup-skill](https://github.com/gonnector/claude-session-wrapup-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
