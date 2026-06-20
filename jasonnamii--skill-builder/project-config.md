---
trigger: always_on
description: 스킬을 생성·수정·검증·패키징하는 1턴 완결 게이트키퍼. 호스트 캐시 직접 편집·VM/호스트 이중환경 분기·N사본 동기·9룰 베놈 강제·.skill 패키징을 한 흐름으로 처리한다.
---


# skill-builder v5.0

**1턴 완결.** ⓪ 이중환경 게이트 → ① 호스트 캐시 직접 편집 → ② N사본 동기 → ③ 검증 → ④ (옵션) 패키징.

**v5.0 이중환경 모델 (2026-05-24):** v4.0의 "캐시-직접" 명제는 유지하되 **VM/호스트 환경 분리 가정 사고를 정정**. 실측 5건: ① VM Bash로 `/var/folders/...` 호스트 캐시 = **접근 불가**(VM 안에서 안 보임) ② VM에 보이는 건 `/sessions/.../mnt/.claude/skills/` bindfs **read-only** 마운트 1곳 ③ "7사본·sha 3종 혼재" 검증 = **VM에서 실행 불가**(N사본 다 못 봄) ④ bindfs 권한으로 cp·zip 직접 쓰기 `Operation not permitted` ⑤ zip `-x "_archive/*"` 패턴 무력화. 따라서 **편집 = 호스트 Edit 도구·검증·패키징 = VM Bash** 도구 분리가 §⓪ 진입 첫 게이트로 박힌다.

**v4.0 (archived):** "캐시 = 형 하드 동일파일" 명제 자체는 유지. 다만 그 캐시를 **어느 도구로 접근하느냐**가 환경별로 다르다는 분기 미명시 → v5.0에서 보강.

**v3.1 베놈 유지:** 모든 신규·수정 스킬에 **9룰 강제 적용**. SKILL.md 골격에 Prereq·NextPhase·OutputPath·RefIndex·Boundaries·WhenToUse·FailureModes 7섹션 강제 + metadata 표준화 + description 첫문장 평문화 강제.

## ⓪ 이중환경 분기 게이트 (v5.0 신설 · 진입 첫 단계)

**원칙**: 본 스킬의 모든 명령은 **두 환경 중 하나**에서 실행됨. 어느 환경에서 무엇을 할지 명시 안 하면 § ① 코드블록 전체가 빈 값을 반환하거나 read-only 권한 오류로 실패.

| 작업 | 환경 | 도구 | 경로 |
|------|------|------|------|
| **SKILL.md 편집** | 호스트(macOS) | Edit·Write·Read (호스트 도구) | `/var/folders/.../T/claude-hostloop-plugins/{HASH}/skills/{X}/SKILL.md` |
| **검증·sha256·grep** | VM(Linux 샌드박스) | bash (mcp__workspace__bash) | `/sessions/{SID}/mnt/.claude/skills/{X}/` (read-only bindfs) |
| **패키징(zip)** | VM | bash | 스테이징=`/tmp/`·결과는 `cat $ZIP > $OUT` redirect |
| **N사본 동기** | 환경 분기 (아래 §② 참조) | — | — |

**진입 게이트 절차**:
1. 형 발화에서 스킬명 추출
2. **호스트 캐시 경로 확인**(호스트 Read로 `/var/folders/.../skills/{X}/SKILL.md` 첫 5줄) — 성공 = 편집 가능
3. **VM mount 경로 확인**(VM bash `ls /sessions/*/mnt/.claude/skills/{X}/SKILL.md`) — 성공 = 검증·패키징 가능
4. 한쪽이라도 실패 시 STOP + 진단 메시지(스킬명 오타 / 플러그인 미설치 / mount 안 됨)

**금지**:
- VM bash로 `/var/folders/...` 접근 시도 ✗ (안 보임)
- 호스트 Edit으로 `/sessions/...` 접근 시도 ✗ (호스트에 없음)
- 호스트 캐시를 VM cp로 staging 시도 ✗ (bindfs 권한 거부)

## Skill Boundaries

- **하는 것** — SKILL.md 편집·references/scripts 갱신·9룰 강제 적용·N사본 동기·검증·(옵션) 패키징
- **안 하는 것** — 스킬 품질 진단(→ skill-doctor) · 자동변이/최적화(→ autoloop) · 플러그인 패키징(→ create-cowork-plugin) · UP 편집(→ up-manager) · git push (→ git-sync)

## When to Use

- 사용자가 "{스킬명} 수정해줘·고쳐줘·바꿔줘" — 단일 스킬 편집
- 신규 스킬 생성 요청 ("X 스킬 만들어줘")
- autoloop 루프 종료 후 handoff.json 감지 — 자동 패키징 모드
- skill-doctor 처방전 수신 — 처방대로 수정 후 N사본 동기
- **안 쓸 때** — 단순 질문, 스킬 진단만 원함(→ skill-doctor), 자동 최적화(→ autoloop)

## Prerequisites

본 스킬 진입 전 다음을 체크. 미충족 시 STOP + 안내.

| # | 체크 | 미충족 시 |
|---|------|-----------|
| 1a | **호스트 캐시 경로** 존재 (호스트 Read로 `/var/folders/.../T/claude-hostloop-plugins/*/skills/{X}/SKILL.md` 첫 5줄) | 호스트 캐시 없음 → 진단: ① 스킬명 오타? ② 플러그인 미설치? ③ 호스트 경로 hash 변경? (해시 폴더 ls로 재확인) |
| 1b | **VM mount 경로** 존재 (VM bash `ls /sessions/*/mnt/.claude/skills/{X}/SKILL.md`) | VM mount 없음 → 진단: ① 세션 재시작 필요? ② 플러그인 캐시 미동기? |
| 2 | 9룰 적용 의도 확인 (신규 스킬) | 9룰 골격 자동 삽입. 수정 모드는 누락 룰만 보강 |
| 3 | autoloop handoff.json 있나? (선택) | 있으면 ① 캐시 백업 단계로 핸드오프 사용 |
| 4 | 형 발화에 패키징 트리거 hit? (`패키징해줘·.skill로·줘·달라·제공해줘`) | hit = §④ 자동 진행 / miss = §① §② §③까지만 |

## ⛔ 절대 규칙 (8)

| # | 규칙 |
|---|------|
| 1 | **게이트키퍼** — 스킬 편집 전 본 스킬 발동. 미발동 = FAIL |
| 2 | **이중환경 분리** — 편집 = 호스트 Edit 도구·검증/패키징 = VM bash. 환경 혼동 시 빈 결과·권한 오류로 실패. §⓪ 게이트 우선 통과 |
| 3 | **호스트 캐시 직접 편집** — `/var/folders/.../T/claude-hostloop-plugins/{HASH}/skills/{X}/SKILL.md`를 **호스트 Edit·Write 도구**로 직접 수정. VM bash로 호스트 경로 접근 ✗ · VM staging·cp -r ✗ |
| 4 | **N사본 동기 강제(환경 분기)** — 호스트 캐시 편집 후 호스트 측 다중 plugin 디렉터리 동기는 별도 절차. VM 측은 read-only mount 1곳 자동 반영. §② 참조 |
| 5 | **9룰 베놈 강제** — 신규 스킬은 7섹션(Boundaries·WhenToUse·Prereq·OutputPath·RefIndex·NextPhase·FailureModes) 골격 자동 삽입. 수정 모드는 누락 룰 보강. 미적용 = FAIL |
| 6 | **편집 직후 sha256 검증 강제(VM bash)** — VM mount 경로에서 shasum -a 256. 호스트 다중 사본 동기 시는 별도 sha 비교 절차 |
| 7 | **패키징 권한 우회 강제** — bindfs로 outputs에 zip 직접 쓰기 ✗ → `/tmp/`에서 zip 생성 후 `cat $ZIP > $OUT` redirect |
| 8 | **패키징은 옵션** — 형이 "패키징해줘·.skill로·zip해줘·줘·달라·제공해줘" 명시 시만. 무명시 = §편집·§동기·§검증만으로 완료 |

## ① 호스트 캐시 직접 편집 (호스트 Edit 도구 전용)

**호스트 캐시 경로 패턴**:
```
/var/folders/{2글자}/{랜덤}/T/claude-hostloop-plugins/{HASH}/skills/{SKILL_NAME}/SKILL.md
```

**경로 확인 (호스트 Read 도구로 — VM bash ✗)**:
- 호스트 Read 도구로 첫 5줄 읽기 시도
- 성공 = 편집 가능. 실패 = Prerequisites 1a로 돌아가 진단

**편집 방식 (전부 호스트 도구)**:
- **Edit** — 1~3곳 부분수정. 가장 단순·안전 (디폴트)
- **Write** — 전면 재작성 시만 (신규 스킬·v숫자 마이그레이션). 부분수정에 Write 사용 = FAIL
- **morph edit_file** — 다중수정·반복치환·구조보정 (UP 모프 모드 hit 시만)

**편집 시 주의**:
- `replace_all: false`가 디폴트. 동일 문자열 다중 hit 시 명시 트리거 필요
- 큰 블록 교체보다 작은 단위 다발이 안전(섹션별 Edit 1콜씩)
- 9룰 7섹션 누락 시 Edit로 섹션 추가·기존 본문 보존

**금지**:
- `mcp__workspace__bash`로 `/var/folders/...` 접근 ✗ (VM에서 호스트 캐시 안 보임. exit 1만 반환)
- VM 안에서 `cp -r` 캐시 → `/tmp/` staging ✗ (read-only bindfs·권한 거부)
- `python3 << EOF` 류 heredoc으로 호스트 캐시 직접 쓰기 ✗ (VM 안에서 호스트 경로 없음)

상세 — `→ references/edit-protocol.md`.

## ② N사본 동기 (환경 분기)

### VM 환경 (현 세션 디폴트)

- VM에 보이는 SKILL.md = `/sessions/{SID}/mnt/.claude/skills/{X}/SKILL.md` **1곳뿐**(read-only bindfs)
- 호스트 캐시 편집 시 이 mount는 **자동 반영**(마스터 = 호스트 캐시)
- **VM 안에서는 추가 동기 작업 불필요**. 검증만 진행(§③)

### 호스트 환경 (다중 plugin 디렉터리 존재 시)

호스트에는 plugin UUID × user UUID 조합마다 별개 사본 존재 가능. 다음 위치들 확인:

```
~/Library/Application Support/Claude/local-agent-mode-sessions/skills-plugin/.../skills/{X}/
~/Library/Application Support/Claude/local-agent-mode-sessions/{SID}/.../cache/.../skills/{X}/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jasonnamii/skill-builder](https://github.com/jasonnamii/skill-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
