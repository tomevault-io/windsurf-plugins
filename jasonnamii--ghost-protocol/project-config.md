---
trigger: always_on
description: |
---


<!-- Triggers
P1: 고스트프로토콜, ghost-protocol, ghost protocol, 기획핸드오프, 핸드오프패키지, PRD와기능정의, 기획패키지, 피그마기획, 기획Git패키지.
P2: 고스트프로토콜 실행, PRD랑 기능정의 리스트 만들어줘, 이 피그마로 PRD와 기능 리스트 뽑아줘, 컨셉을 개발자가 볼 PRD로, 이 PDF를 PRD와 기능 리스트로 바꿔줘, Git에 올릴 구조로, 고스트프로토콜 업데이트, 고스트프로토콜 갱신, {기능} 업데이트, 결정 반영해줘(기존 패키지 갱신 → Update Mode), QA 만들어줘, QA 기준 뽑아줘, 디자인·UX QA, 미결 점검, 미결 리마인드, 까먹은 결정 있어?(QA·미결 추적).
P3: planning handoff package, PRD + feature list, concept to PRD, figma to PRD, build planning package.
P5: Git 구조로, /docs/planning 으로, md+csv로, 엑셀로도, 7파일로(+qa).
NOT: 화면→기능 후보만(→feature-inventory), 정책·예외 설계 검증만(→planning-review), 범용기획 발산(→planning-skill), 정책기획(→policy-planning), UI비주얼(→ui-designer), 사업전략(→biz-skill).
GATE: "컨셉/화면/문서가 있고 → 개발자·디자이너가 바로 받아 착수할 기획 패키지(PRD + 기능정의)가 필요하다"는 상황에서 발동. 입력이 컨셉 한 줄이라도 있으면 실행한다(부족해도 멈추지 않음).
-->

# 고스트프로토콜 — ghost-protocol (입력 → PRD·기능정의 동시 산출 → Git 패키지)

**핵심 공식**: 고스트프로토콜은 컨셉·화면·기존 문서를 **개발 착수 가능한 PRD와 기능정의 리스트로 바꾸는 기획 핸드오프 스킬**이다. 핵심은 많이 쓰는 것이 아니라 **빠짐없이 닫는 것**이다. PRD는 제품 의도와 구현 범위를 정리하고, 기능정의 리스트는 개발자가 작업 단위를 볼 수 있게 만든다. 두 문서는 F-ID와 FR-ID로 연결한다. 가치는 "문서를 많이 만드는 것"이 아니라 **근거(화면에서 본 것)·의도(컨셉에서 온 것)·제안(AI가 보강한 것)·미결정(사람이 정할 것)을 절대 섞지 않는 것**에 있다.

> **이 스킬은 PRD 자동작성기도 기능정의 자동화기도 아니다.** 두 산출물을 ID로 연결하고 Git 패키지로 닫는 **단일 핸드오프 엔진**이다. PRD만 만들면 실패, 기능 리스트만 만들면 반쪽이다. 그리고 PRD는 **구현 지시서가 아니라 개발 착수 기준 문서**다 — 긴 문서가 목표가 아니라 개발 착수에 필요한 최소 완결성이 목표다.

## 내부 6엔진 (한 흐름으로 잇는다)

두 참고 스킬(feature-inventory·planning-review)을 **병렬 호출하지 않는다. 내부 엔진으로 흡수**해 한 흐름으로 잇는다.

```
ghost-protocol
├─ 1. 입력 해석 엔진     — 컨셉/이미지/피그마URL/PDF/기존문서/혼합 판별·정규화 → references/input-normalization.md
├─ 2. 컨텍스트 로드 엔진 — (요청 시) GitHub 원격 최신 코퍼스 fetch·읽기·레지스트리 → references/platform-context.md · scripts/corpus_tools.py
├─ 3. 기능 추출 엔진     — 화면 있으면 화면 기반 역추출, 컨셉만이면 컨셉 기반 초안. 15컬럼 → references/extraction-rules.md · feature-column-spec.md
│                          (feature-inventory 흡수 — 화면 전제는 상위 레이어에서 완화)
├─ 4. PRD 검증·작성 엔진 — 정책·예외·상태 내부 점검 + 레드플래그 + 컴플라이언스 → 기본 12섹션 PRD(+ 선택 섹션)
│                          → references/policy-exception-checklist.md · prd-template.md (planning-review 흡수, 웨이트 게이트 제거)
├─ 5. 결정 엔진         — 못감-정보부족 / 못감-결정부족 / 기획 확인 필요 + (요청 시) 플랫폼 정합성 → references/decision-log-template.md
└─ 6. Git 패키징·사후검토 — 7파일 동시 산출(+qa.md) + F↔FR + _registry + 미결 추적(track_open) + (요청 시) Git 대조 사후검토 + 커밋 안내 → references/git-output-rules.md · platform-context.md
```

핵심은 **feature-inventory → planning-review를 따로 부르지 않고**, 고스트프로토콜 안에서 입력 정규화 → (요청 시) 코퍼스 로드 → 기능 추출 → PRD 검증 → 결정 분리 → Git 패키징을 **한 번에·멈추지 않고** 잇는 것이다.

## Skill Boundaries

- **하는 것** — 어떤 입력이든(컨셉/이미지/피그마/PDF/기존문서/혼합) 받아 → 제품·기능 모델 정리 → 화면·컨셉에서 기능 후보(15컬럼) 역추출 → 정책·예외·상태 내부 점검 + 레드플래그·컴플라이언스 스캔 → **기본 12섹션 PRD와 기능정의 리스트를 동시 생성** → FR↔F ID 연결 → 미결정 분리 → README·PRD·features.md·features.csv·decisions.md·qa.md·changelog.md **7파일**을 Git 패키지로 산출(엑셀은 옵션). qa.md는 PRD·features에서 도출한 합격 기준(디자인 완성도·사용자 경험/모션 포함)을 담는 별도 검증 문서이고, 열린 미결은 닫힐 때까지 추적한다.
- **안 하는 것** — 화면만 보고 기능 후보만 뽑는 단일 작업(→`feature-inventory`), 이미 있는 기획의 정책·예외 설계 검증만(→`planning-review`), 범용 기획 발산·아이디어 단계(→`planning-skill`), 정책기획·공약(→`policy-planning`), 화면 UI 비주얼 디자인(→`ui-designer`·`apple-canvas`), 사업전략(→`biz-skill`), 약관·개인정보 등 **법무 문서 생성**(감지·라우팅만, 생성은 →`app-and-jang`·`ip-skill`). 그리고 **개발 구현 방식 지시**(DB 테이블 구조·API 엔드포인트·컴포넌트 세부 설계·픽셀 UI)는 쓰지 않는다.

---

## When to Use

- 컨셉 문장·와이어프레임·피그마 URL·PDF·기존 기획서가 있고, **개발자·디자이너가 바로 받아 착수할 기획 패키지(PRD + 기능정의)**가 필요할 때
- "고스트프로토콜 실행", "PRD랑 기능정의 리스트 만들어줘", "이 피그마로 PRD와 기능 리스트 뽑아줘"로 발동
- 화면은 있는데 PRD가 없거나, 컨셉만 있는데 개발 가능한 기준 문서로 닫아야 할 때
- **안 쓸 때** — 화면 보고 기능 후보 목록 1개만 빠르게(→`feature-inventory`), 이미 있는 기획의 구멍만 메우기(→`planning-review`), 아직 아이디어 발산 단계(→`planning-skill`)
- **이미 만든 패키지를 갱신할 때** → 아래 **Update Mode**(첫 생성과 같은 스킬, 다른 진입)

## Update Mode — 기존 패키지 갱신 (첫 생성 이후)

첫 생성으로 끝이 아니다. 컨셉 초안은 미결정이 많고, 사람이 **결정을 내리고 기능을 더하면서** 여러 번 갱신된다. 그 입구가 **`decisions.md` 답안지**다(별도 입력 파일 없음 — 결정 답안지가 곧 갱신 입력칸). → 상세 `references/update-cycle.md`·`decision-log-template.md`.

- **발동**: "고스트프로토콜 업데이트 {feature}", "고스트프로토콜 갱신", "{feature} 업데이트", "결정 반영해줘". 대상 패키지가 `mnt/outputs/planning/{feature}/`(또는 repo)에 이미 있으면 Update Mode.
- **사용자 흐름**: decisions.md에서 결정은 객관식(`▶ 내 결정: A/B`), 추가·수정·뺄것·새입력은 ➕ 빈칸을 채운다 → "고스트프로토콜 업데이트 {feature}".
- **스킬 처리(한 흐름)**:
  0. **묵힌 미결 환기**: `python3 scripts/track_open.py --root mnt/outputs/planning --today {오늘}` 1회 → 3일+ 안 닫힌 미결을 응답 맨 위에 먼저 띄운다(이 패키지뿐 아니라 형제 패키지 것도). "3일 전에 정하기로 해놓고 잊은 것"을 지금 환기(`update-cycle.md §6`).
  1. 7파일 읽고 decisions.md에서 **답이 채워진 항목만** 추림(없으면 무엇을 갱신할지 1줄 확인).
  2. 종류별 반영. **결정(D-n→안)은 PRD §8/§9·features 정책 칸을 미정→확정, decisions의 그 항목은 ✅ 결정됨 표로 이동. 기능 추가는 새 F/FR ID(기존 최대+1, 재사용 금지). 수정/범위변경/새입력**은 해당 위치만(`update-cycle.md §3`).
  3. **근거·의도·제안·미결정 분리(절대규칙 4)·F↔FR 연결(절대규칙 6) 유지.** PRD·features를 직접 손대지 말고 decisions.md 답안을 통해서만 반영(직접 편집은 ID·분리가 깨짐).
  4. changelog.md 새 버전 항목(날짜+변경+영향 ID), `version`·`updated` 올림, `status` 재판정(미결정 닫히면 draft→review).
  5. decisions.md ✅ 결정됨 정리(열린 목록에서 제거, 닫힌 항목의 📅 줄은 날짜로 흡수), ➕ 처리분 비움, **새로 연 미결엔 📅 opened={오늘}**. 영향 큰 결정·범위변경은 반영 전 한 줄로 영향 짚고 확인.
  6. **qa.md 재생성**(닫힌 결정·확정 화면이면 묶인 보류 QA를 미검증으로 풀고 합격선 확정, 새 FR엔 QA 추가 — `update-cycle.md §6`) + 재검증(validate) + (코퍼스 활성 시 정합성 재대조) + (요청 시 커밋/푸시 명령).

## QA 기준 · 미결 추적 (별도 레이어 — 무엇·왜)

핸드오프는 "개발 착수"에서 끝나지 않는다. 두 가지를 더 둔다(실행 절차는 위 Phase 0·6·7·Update Mode, 깊이는 `qa-template.md`·`update-cycle.md §6`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jasonnamii/ghost-protocol](https://github.com/jasonnamii/ghost-protocol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
