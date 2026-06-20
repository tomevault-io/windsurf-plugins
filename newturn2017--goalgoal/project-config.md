---
trigger: always_on
description: This skill should be used when the user wants to turn a vague one-line idea into a clear, verifiable goal that Claude Code's `/goal` autonomous loop (or /loop, ouroboros seed) can run without spinning or converging wrong. Triggers — "목표 잡아줘", "goal 정의해줘", "/goal 돌릴 목표 만들어줘", "Loop에 넣을 목표 만들어줘", "아이디어 구체화해줘", "goal.json 만들어줘", "뭘 만들지 정리해줘", "goalgoal", "make a goal for the loop", "define the objective", "turn this idea into a goal". An ambiguous goal makes a repeating loop waste iterations, so th
---


# goalgoal

> 막연한 한 문장 아이디어를 인터뷰로 캐물어, **Claude Code `/goal`**(자율 반복 루프)이 헛돌지 않는 **검증 가능한 `goal.json` 목표**로 증류한다. 주 산출물은 `goal.json` 안의 `goal_command` — 사용자가 `/goal <goal_command>` 형태로 복붙해 그대로 투입한다.

이 스킬의 사용자는 코딩을 몰라도 되는 바이브코더다. 전문 용어는 항상 쉬운 말로 풀어 설명한다.

## 왜 필요한가

Claude Code `/goal`은 매 턴 끝에 **별도 평가자(Haiku 계열)가 완료조건 충족 여부를 yes/no로 판정**하는 자율 루프다. 목표가 모호하면 무한 반복하거나 잘못된 방향으로 수렴한다. "쇼핑몰 만들어줘" 같은 한 줄은 성공 기준도, 완료 정의도, 범위도 없어 기계가 반복 실행하기엔 미완성 입력이다. goalgoal은 그 간극 — "사람의 모호한 의도"와 "`/goal`이 그대로 돌릴 수 있는 목표" — 를 인터뷰로 메운다. `/goal`의 계약은 `references/loop-formats.md` 참조(추정 금지).

## Iron Law

**측정 불가능하거나 종료 조건이 없는 목표는 절대 `status: "ready"`로 내보내지 않는다.**

모호한 목표를 통과시키면 `/goal`이 무한 반복하거나 엉뚱한 결과로 수렴하고, 그 책임이 사용자에게 전가된다. 자신이 없으면 `confidence: "low"` + `status: "needs_review"` 게이트를 건다.

**이건 말뿐인 규칙이 아니라 코드로 강제된다.** Step 8의 `validate_goal.py`가 단일 진실 원천(SSOT)이며, 차단 이슈(errors)가 하나라도 있으면 `enforced_status`를 강제로 `needs_review`로 내리고 종료코드 1을 반환한다. 검증을 통과(`pass: true`)하기 전에는 사용자에게 "ready"라고 말하지 않는다.

## Demonstrable-in-conversation (핵심 철칙)

`/goal` 평가자는 **도구를 못 돌리고 파일을 직접 못 읽는다 — 대화에 드러난 출력만으로 판정**한다. 따라서 모든 성공기준과 `goal_command` 체크는 **Claude가 출력으로 증명할 수 있는 것**이어야 한다(❌ "사용자가 만족"·"보기 좋음" → ✅ "`swift build` exit 0"·"테스트 통과율 100%가 출력에 표시"). 증명 불가한 기준을 발견하면 그대로 두지 말고 "어떻게 출력으로 보여줄 수 있을까?"로 바꿔 묻는다. 근거와 전체 예시의 SSOT는 `references/loop-formats.md`("결정적 제약 3가지").

## 워크플로우

### Step 1 — 명확도 스코어링 + 도메인 분류 (prompt)

사용자의 한 문장 아이디어를 받는다. 인수로 없으면 AskUserQuestion으로 한 문장만 받는다.

**재개 확인 (먼저):** 현재 디렉토리에 `status: "needs_review"`인 `goal.json`이 있으면, "이전에 정하다 만 목표가 있어요. `open_questions`만 이어서 물을까요?"라고 제안한다. 동의하면 그 파일을 읽고 `open_questions`에 해당하는 질문만 다시 던진다(별도 세션 파일 없음 — 산출물 자체가 재개점이다).

**입력 검증:** 10자 미만이거나 의미 있는 명사·동사가 없으면 "한 문장 이상의 아이디어를 적어주세요"라고 안내하고 멈춘다(재시도 3회 한도). 2,000자 초과 입력(코드 덩어리 등)은 "핵심 아이디어만 한 문장으로 요약해주세요"로 리다이렉트한다.

다음 4축의 명확도를 0~1로 채점한다: **목표 / 성공기준 / 종료조건 / 범위·스택**. 그리고 **도메인**(code·data·infra·automation·creative·analysis), **규모**(spike·mvp·production), 그리고 **`risk_tier`**(none/low/**elevated**)를 판정한다. 파일 변경·삭제, 시스템·권한 접근, 사용자 콘텐츠 읽기, 외부 전송 신호가 있으면 elevated — 이 경우 Step 7에서 `safety` 블록이 필수가 된다.

**추정 먼저, 질문은 나중 (핵심 효율 장치):** 사용자는 코딩을 모르는 바이브코더고, 매번 4~16개를 물으면 지친다. 좋은 한 문장은 이미 절반을 함축한다 — "매일 쓰는 사소한 일을 자동화하는 macOS 메뉴바 앱"이면 도메인=code, 결과물=메뉴바 앱, 스택≈Swift/SwiftUI, 맥락≈개인용, 규모≈mvp가 거의 자명하다. 그러니 **추론 가능한 축은 LLM이 먼저 채워 "잠정 추정"으로 들고**, 인터뷰는 **추론 불가능한 축**(주로 성공기준·종료조건 — 사용자 머릿속에만 있는 것)에 집중시킨다. `question-pool.md`에서 `infer_first`로 표시된 질문이 추정 대상이다.

단, 추정은 **반드시 사용자 확인을 거친다**(Iron Law: 4축은 사용자가 확인해야 한다). 추정 + 명시적 확인은 "조용한 가정"이 아니다 — 틀리면 바로 고친다. 자신 없는 추정은 confidence를 낮추는 신호로 삼는다.

4축이 모두 명확하면(이미 또렷한 아이디어) 인터뷰를 건너뛰고 "이미 충분히 구체적이에요. 바로 goal.json 만들까요?"라고 확인한 뒤 Step 7로 간다.

### Step 2 — 질문 가지치기 (rag)

`references/question-pool.md`의 **20개 마스터 질문 풀**을 연다. 세 단계로 줄인다: ①`infer_first` 질문 중 한 문장에서 추론된 것은 **추정값으로 흡수**(1차의 확인 질문으로 합침), ②도메인 매핑·`skip_if`로 무관·중복 제거, ③남은 것을 묶음으로 구성. 결과는 보통 **4~8개**(추정이 잘 먹히면 더 적게). 이미 또렷하거나 추정된 축의 질문은 다시 묻지 않는다.

### Step 3 — 1차 인터뷰: 추정 확인 + 핵심 미지축 (api_mcp)

1차 묶음의 목적은 **추출이 아니라 확인**이다. AskUserQuestion 한 번에 다음을 담는다(질문 ≤4, 옵션 각 ≤4):

1. **추정 확인 질문 (1개)** — Step 1에서 추론한 잠정값을 한 묶음으로 보여주고 맞는지 묻는다. 예: *"이렇게 이해했어요 — 결과물=macOS 메뉴바 앱, 언어=Swift, 규모=최소 동작(MVP), 사용자=본인. 맞나요?"* 옵션: `다 맞아요` / `규모만 달라요` / `결과물·스택이 달라요` / `많이 달라요(다시 설명할게요)`. "많이 달라요"면 Other로 재설명을 받아 추정을 갱신한다.
2. **성공기준 측정 (1개, 절대 추정 금지)** — 성공을 **대화 출력으로 어떻게 증명**할지. 이건 사용자 머릿속에만 있어 추론 불가다. demonstrable 기준(숫자·%·통과/실패·`exit 0`·"메뉴바에 표시")으로 유도한다.
3. **종료조건 (1개)** — "성공적으로 끝났을 때 vs 억지로라도 멈춰야 할 때"를 한 질문으로. JSON에선 `done_definition` / `stop_condition`으로 분리한다.
4. (선택) **범위 밖 (1개)** — 추정이 과해 보이면 "이번엔 안 할 것"을 물어 범위를 못 박는다.

진행률을 질문 머리말에 "1/N"처럼 보여주고, 각 질문에 **"잘 모르겠어요"** 선택지를 항상 둔다("Other" 자유입력은 자동 제공됨). 추정 확인이 `다 맞아요`로 떨어지면 도메인 추출 질문 대부분이 사라져 전체 질문 수가 크게 준다.

### Step 4 — 분기 + 중간 미리보기 (prompt)

1차 답변을 파싱한다. 추정 확인이 **틀린 것으로 나오면 그 축만 보정 질문을 끼우고**(맞은 축은 그대로 확정), 나머지 처리 규칙은 `references/question-pool.md`의 "운영 규칙"을 따른다:

- **Other 자유입력**은 다음 묶음 첫 질문을 "말씀하신 내용이 이 중 어디에 가깝나요?"로 바꿔 **재앵커링**한다.
- **"잘 모르겠어요" 누적**을 센다(연속 3회 또는 총 10회 초과 시 인터뷰 중단 → 합리적 기본값 + `confidence:low`).
- **모순**을 감지한다(예: "모바일 앱" + "CLI 배포"). 발견하면 종료 전 재확인 질문을 끼운다.

그리고 **"지금까지 이렇게 정리됐어요"** 중간 초안을 사람이 읽는 한국어로 짧게 보여준다 — 사용자가 자기 의도가 반영되는지 보며 끝까지 참여하게 만든다.

### Step 5 — 2차 인터뷰: 남은 도메인 디테일 + 검증·범위 (api_mcp, 필요 시)

추정 확인으로 **이미 채워진 도메인 축은 건너뛴다.** 남은 미지 항목만 AskUserQuestion 한 묶음으로 묶는다: 도메인 특화 중 추정 안 된 것(0~2개) + `verification_method` 1개 + `out_of_scope`/`risks` 1개. 도메인별 후보와 `max_iterations` 기본값은 `references/question-pool.md`. 추정이 다 맞고 1차에서 검증·범위까지 나왔으면 이 단계 자체를 생략한다.

### Step 6 — 3차 인터뷰: 조건부 (api_mcp)

`done_definition` 또는 `constraints`가 **아직 모호할 때만** 3차 묶음을 호출한다. 앞 단계에서 이미 또렷해졌으면 이 단계를 **생략**한다 — 전형적으로 총 질문 수는 4~8개로 압축된다.

### Step 7 — goal.json 생성 (generate)

수집한 답변으로 `goal.json`을 만든다. 스키마·필드 정의는 `references/goal-schema.md`, `goal_command` 작성법은 `references/loop-formats.md`. 순서:

1. **구조화 필드 채우기**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NewTurn2017/goalgoal](https://github.com/NewTurn2017/goalgoal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
