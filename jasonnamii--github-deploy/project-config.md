---
trigger: always_on
description: |
---


# GitHub Deploy (choi 디폴트 + pdkim 명시 모드 · Bash 자동실행)

**깃허브배포·퍼블리싱·웹배포·깃배포** 엔진. 디폴트는 `works.choi.build/{레포명}/`, 명시 호출 시 `works.pdkim.com/{레포명}/`. Private + noindex + HTTPS. **2도메인 체제 (디폴트 1택 + 옵션 1택).**


## Skill Boundaries

- **하는 것** — GitHub Pages 자동 배포 + **GA4 측정 ID 자동 주입** (mode별 분기).
- **안 하는 것** — 레포관리(→직접), DNS(→직접), jasonnamii.github.io 신규배포(→deprecated·리다이렉트만).

**📊 GA4 측정 ID 매핑 (verbatim · 절대 외우지 말 것 — deploy.sh _helper.py가 SSOT):**

| mode | 도메인 | 측정 ID |
|---|---|---|
| **choi (디폴트)** | works.choi.build | `G-1HB3T0BTW6` |
| **pdkim (명시)** | works.pdkim.com | `G-BRYWG9T5PQ` |

**Claude는 GA 스니펫을 절대 수동으로 HTML에 박지 않는다.** deploy.sh phase 3.5가 자동 주입. 형이 "GA 적용해줘"라고 해도 Claude의 역할 = `bash deploy.sh ...` 1콜로 끝. 형 vault grep 등으로 ID를 "찾아서" 박는 행위 = FAIL (모드별 ID 거꾸로 박힐 위험 100%).

**🚀 실행 환경 (필독):** 모든 bash는 **Claude가 직접 Bash 도구로 자동 실행**. `~/github-repos/skill-repos/github-deploy/scripts/deploy.sh`는 형 맥북 zsh 환경에서 `gh auth`·SSH 키·토큰을 그대로 사용. **1줄 명령 출력 후 "형이 맥북 터미널에 붙여넣어 실행" 안내 = FAIL**. 자동 실행이 디폴트, 수동 안내는 폐기.

**원칙:** SKILL.md는 분기·규칙만. 실행은 전부 `scripts/*.sh` 호출. LLM이 bash 본문 생성 ✗ → Bash 도구로 스크립트 호출만.

**v3.1 (2026-05-24) — 재배포 분기 단축 (-20s/재배포):**
1. **verify_root sleep 분기** — `DEPLOY_KIND=redeploy`면 초기 대기 20s→5s. 신규는 false-positive 회피용 20s 유지. 평균 -15s/재배포.
2. **clone 캐시 재사용** — `$WORK`(`/tmp/gh-deploy/{root}`)가 유효한 git repo면 `fetch + reset --hard origin/HEAD + clean -fdx` 1콜. 신규 clone 4~5s → 캐시 fetch 1~2s. 평균 -3s/배포.
3. **자동 fallback** — fetch/reset 실패 시 통째 `rm -rf` 후 신규 clone. 안전성 무손실.
4. **실측** — 동일 입력 재배포 28s → 8~10s.

**v2.2 (2026-05-02) — Bash 자동실행 전면화:** v2.1의 "샌드박스 직접 push 불가·형이 수동 실행" 정책 폐기. Claude Code의 Bash는 형 맥북 zsh를 그대로 쓰므로 `gh auth`·SSH·토큰 전부 동작. Claude가 deploy.sh를 **무조건 자동 호출**. 1줄 명령 출력 안내 전면 삭제.

**v3.0 (2026-05-23) — 외부공유 락 모드 신설 (잘못된 mode 배포 차단):**
1. **`scripts/external-locks.json` 신설** — slug별 `expected_mode`·`reason` 박제. 외부공유 중인 slug를 자동 보호.
2. **`check_external_lock()` 함수** — Phase 0 직전 호출. slug가 락에 등록돼 있고 호출 mode가 expected_mode와 다르면 `exit 9` abort.
3. **`--override-lock` 플래그** — 락 우회용. 외부공유 끊김을 감수할 때만 사용. 인자 어느 위치에 와도 인식.
4. **현재 락 보유 slug:** `madpop_with_kim` (expected=pdkim, 외부 공유 중).
5. **Failure Modes에 lock 항목 3종 신설** — abort 시나리오·override 사용 시점·락 관리 방법.

**v2.9 (2026-05-23) — F1 short-circuit 시에도 manifest 미러 갱신:**
1. **F1 sha256 match 분기 패치** — 동일 콘텐츠 재배포 시 `exit 0` 직전에 `manifest_update` 1회 호출. 콘텐츠 push는 스킵하되 로컬 manifest 미러의 `timestamp`·`last_kind=skip-same-sha`를 최신화.
2. **누적 drift 차단** — F1 빠짐이 반복돼도 manifest_lookup이 항상 최신 상태. 다음 신규/redeploy 시 원격 `_manifest.json`에 합류됨.
3. **Failure Modes에 F1+manifest 항목 신설** — 동일 콘텐츠 재배포 시 manifest가 누락되지 않는지 status 박제(`STATUS=skip-same-sha`)로 확인.

**v2.8 (2026-05-22) — manifest 합집합 분기 신설 (choi+pdkim 양쪽 조회):**
1. **Phase 0.5 신설** — slug 정규화 후 choi/pdkim 양쪽 `_manifest.json` 미러 합집합 조회. 4케이스 자동 판정 (`new_global`·`redeploy_same`·`exists_other`·`exists_both`).
2. **`_manifest.json` 박제 위치** — 각 루트 레포(`works-choi-live`·`works-pdkim`) 루트에 1개씩. 로컬 미러는 `~/github-repos/skill-repos/github-deploy/.cache/manifest-{mode}.json`.
3. **반대 모드/양쪽 hit 시 ⚠ 경고 + 진행** — 자동 중단 ✗ (배포 흐름 보존). `MANIFEST_VERDICT` + `MANIFEST_OTHER_URL`을 `.deploy-status.txt`에 박제하여 형이 사후 확인 가능.
4. **slug 키 = 정규화된 REPO 인자** — 소문자·하이픈·영숫자만. 형이 명시 박제(`KISAS-TF-Agenda` → `kisas-tf-agenda`).

**v2.7 (2026-05-21) — GA4 매핑 SSOT 박제 + 수동 박기 금지 룰 신설:**
1. **GA4 ID 매핑을 Skill Boundaries에 verbatim 박제** — choi=G-1HB3T0BTW6 / pdkim=G-BRYWG9T5PQ. Claude가 vault grep 등으로 ID 추측·수동 박기 금지. deploy.sh phase 3.5(_helper.py GA4_IDS dict)가 SSOT.
2. **Failure Modes에 GA4 3종 신설** — ① 수동 박기 금지 ② "기존 gtag 있으면 스킵" 가드와의 충돌 ③ ID 매핑 외우지 말 것.
3. **WRONG/CORRECT에 GA 사건 케이스 추가** — vault grep으로 찾은 G-BRYWG9T5PQ를 choi에 박은 실제 사고 1건 박제.

**버전 히스토리 (요약):**
- **v3.1 (2026-05-24)** — 재배포 verify 대기 20s→5s + clone 캐시 재사용. 동일 입력 재배포 28s→8~10s.
- **v2.5 (2026-05-10)** — verify polling(sleep 20+5s×12), Phase 0 병렬화, `_helper.py` 통합. 배포 120s→80s, 재배포 라우팅 8s→3s.
- **v2.4 (2026-05-09)** — F1 sha256 short-circuit, F2 `.deploy-status.txt` 9필드 박제, F3 stdout flush, F4 timeout≠실패 가이드. 동일파일 재배포 1초.
- **v2.3 (2026-05-06)** — Phase 0 라우팅 게이트(cache+tree+head 3중 조회), 검증 단순화(루트 URL 1회 HEAD), `.deploy-cache.json` 자동 갱신.

**계정·레포 구조 (고정):**
- OWNER = `jasonnamii`
- choi 배포처 (디폴트) = `jasonnamii/works-choi-live` 루트 레포의 `/{레포명}/`
- pdkim 배포처 (명시) = `jasonnamii/works-pdkim` 루트 레포의 `/{레포명}/`
- 레거시(리다이렉트 전용) = `jasonnamii/jasonnamii.github.io`
- **스크립트 경로 (형 맥북 표준):** `~/github-repos/skill-repos/github-deploy/scripts/`

---

## When to Use

- 사용자가 "배포해줘", "올려줘", "자동으로 배포", "바로 배포", "리다이렉트 걸어줘" 같은 표현으로 발동
- 도메인 작업이 필요한 시점
- **안 쓸 때** — 레포관리(→직접), DNS(→직접), jasonnamii.github.io 신규배포(→deprecated·리다이렉트만).


## Prerequisites

| # | 체크 | 미충족 시 |
|---|------|-----------|
| 1 | 대상·입력 명확 (스킬 발동 의도 확인) | 1줄 확인 후 진입 |
| 3 | scripts/ 실행 권한 | 권한 보정 후 재시도 |


## 🚀 라우팅 (Bash 자동실행 직행)

**디폴트 = choi.** pdkim은 명시 트리거가 있을 때만. **모든 트리거에서 Claude가 즉시 Bash 도구 호출.**

| 트리거 | 액션 (Claude 자동) |
|---|---|
| **배포·build·deploy·깃배포·재배포** + 파일/레포명 (도메인 미지정) | Bash 도구 → `bash ~/github-repos/skill-repos/github-deploy/scripts/deploy.sh {repo} {src}` |
| **"pdkim으로 배포"·"pdkim 모드"·"김피디 배포"·`--mode=pdkim`** | Bash 도구 → `... deploy.sh {repo} {src} --mode=pdkim` |
| **리다이렉트·마이그레이션** | Bash 도구 → §레거시 마이그레이션 스크립트 |
| jasonnamii.github.io 신규 배포 언급 | "리다이렉트 전용. choi 또는 pdkim으로." 안내만 (실행 ✗) |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jasonnamii/github-deploy](https://github.com/jasonnamii/github-deploy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
