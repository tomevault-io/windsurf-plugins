---
trigger: always_on
description: 이 프로젝트는 **Windows에서 Claude Code Agent Teams의 split pane 모드를 설정**하는 자동화 가이드입니다.
---

# Claude Code Agent Teams + Split Pane 자동 설정 가이드

이 프로젝트는 **Windows에서 Claude Code Agent Teams의 split pane 모드를 설정**하는 자동화 가이드입니다.
사용자가 "설정 시작해줘"라고 하면, 아래 절차를 순서대로 실행합니다.

---

## 핵심 원칙

### 환경 감지 우선 (CRITICAL)

**모든 단계에서 먼저 이미 설치/설정되어 있는지 확인하고, 되어 있으면 건너뜁니다.**
건너뛸 때는 사용자에게 "[Step N/9] {항목}이 이미 설치되어 있습니다. 건너뜁니다. ✅" 안내.

### 실행 유형 — 반드시 구분하세요 (CRITICAL)

이 가이드에는 두 종류의 코드 블록이 있습니다. **절대 혼동하지 마세요.**

| 표시 | 의미 | Claude 행동 |
|------|------|------------|
| `🤖 실행:` 뒤의 bash 블록 | Claude Code가 직접 실행할 명령어 | Bash 도구로 실행 |
| `📋 사용자에게 전달:` 뒤의 텍스트 블록 | 사용자에게 보여줄 안내문 | AskUserQuestion으로 텍스트 전달만. **절대 실행하지 마세요** |

**⚠️ "📋 사용자에게 전달:" 블록 안의 명령어(wsl --install, claude auth login 등)는 사용자가 직접 입력할 명령어입니다. Claude Code가 실행하면 안 됩니다.**

### 안내 메시지 톤
- 한국어 존댓말, 친근하고 명확
- 단계 번호와 현재 진행 상황 표시
- 구체적으로 무엇을 해야 하는지 설명
- 완료 확인 방법 안내

### 선택 기능 처리 (CRITICAL)
- Optional 단계 진입 전 AskUserQuestion으로 "이 기능을 설정하시겠습니까?" 질문
- **사용자가 질문에 답하지 않는 모드(dontAsk, bypassPermissions 등)를 사용 중이라면, 선택 단계를 건너뛰고 필수 단계만 완료하세요**
- 각 선택 기능이 무엇이고 왜 유용한지 간단히 설명한 후 질문

---

## 시작: 환경 스캔

사용자가 설정을 요청하면, **먼저 현재 환경을 전체 스캔**합니다.

**🤖 아래 명령어들을 순서대로 실행하세요 (실행):**
```bash
# 1. 플랫폼 확인
# process.platform === "win32" 인지 확인
# → Windows가 아니면: "이 가이드는 Windows 전용입니다" 안내 후 중단

# 2. WSL2 확인
wsl --status

# 3. Ubuntu 확인
wsl -d Ubuntu -- echo "ok" 2>/dev/null

# 4. 기본 패키지 확인 (Ubuntu가 있을 때만)
wsl -d Ubuntu -- bash -c "tmux -V 2>/dev/null; git --version 2>/dev/null; curl --version 2>/dev/null | head -1"

# 5. Node.js 확인
wsl -d Ubuntu -- bash -c "export NVM_DIR=\"\$HOME/.nvm\" && [ -s \"\$NVM_DIR/nvm.sh\" ] && . \"\$NVM_DIR/nvm.sh\" 2>/dev/null; node --version 2>/dev/null"

# 6. Claude Code 확인
wsl -d Ubuntu -- bash -c "export NVM_DIR=\"\$HOME/.nvm\" && [ -s \"\$NVM_DIR/nvm.sh\" ] && . \"\$NVM_DIR/nvm.sh\" 2>/dev/null; claude --version 2>/dev/null"

# 7. Oh My Tmux 확인
wsl -d Ubuntu -- bash -c "test -d ~/.tmux && echo 'installed' || echo 'not found'"

# 8. teammateMode 확인
wsl -d Ubuntu -- bash -c "cat ~/.claude/settings.local.json 2>/dev/null || find ~ -maxdepth 3 -path '*/.claude/settings.local.json' -exec cat {} \; 2>/dev/null | head -1"
```

스캔 결과를 사용자에게 보고:

```
환경 스캔 결과:

  WSL2:          ✅ 설치됨 / ❌ 미설치
  Ubuntu:        ✅ 설치됨 / ❌ 미설치
  tmux:          ✅ x.x / ❌ 미설치
  git:           ✅ x.x / ❌ 미설치
  Node.js:       ✅ vXX.x.x / ❌ 미설치
  Claude Code:   ✅ x.x.x / ❌ 미설치
  Oh My Tmux:    ✅ 설치됨 / ❌ 미설치
  teammateMode:  ✅ tmux / ❌ 미설정

→ Step {첫 번째 미설치 항목}부터 진행합니다.
```

이미 모든 항목이 설치된 경우 → "모든 Core Setup이 완료되어 있습니다!" 안내 후 Part 2로 바로 진행.

---

## Part 1: Core Setup (필수)

이 단계들만으로 Agent Teams + split pane이 완전히 작동합니다.

### Step 1: WSL2 설치 (👤 사용자 조작 필요)

**🤖 환경 감지 (실행):**
```bash
wsl --status
```
- 정상 출력 (기본 버전, 커널 버전 등 표시) → **건너뛰기** ✅
- 에러 또는 미설치 → 아래 진행

**이 단계는 관리자 권한이 필요합니다. Claude Code가 직접 실행할 수 없습니다.**

**📋 사용자에게 전달 (실행 금지 — AskUserQuestion으로 텍스트만 전달):**

```
[Step 1/9] WSL2를 설치해야 합니다.

아래 단계를 따라주세요:

1. Windows 검색창에 "PowerShell" 입력
2. "Windows PowerShell"을 우클릭 → "관리자로 실행"
3. 열린 창에 아래 명령어를 붙여넣고 Enter:

   wsl --install

4. 설치가 완료되면 컴퓨터를 재부팅해주세요

5. ⚠️ 재부팅 후 두 가지 경우가 있습니다:

   [경우 A] Ubuntu 설정 화면이 자동으로 나타나는 경우:
   → 사용자 이름(영문 소문자)과 비밀번호를 설정하세요
   → 설정 완료 후 그 창은 닫아도 됩니다

   [경우 B] 아무 화면도 안 나타나는 경우:
   → 정상입니다! 다음 단계에서 Ubuntu를 별도로 설치합니다

6. 재부팅 후 터미널에서 아래 명령어로 이 세션을 이어서 진행할 수 있습니다:

   claude --resume

7. 세션이 복구되면 "완료"라고 알려주세요
```

**🤖 사용자가 완료를 알리면 (실행):** `wsl --status`로 WSL 설치 확인 후 다음 단계.

---

### Step 2: Ubuntu 설치 (👤 사용자 조작 필요)

**🤖 환경 감지 (실행):**
```bash
wsl -d Ubuntu -- echo "ok" 2>/dev/null
```
- `ok` 출력 → **건너뛰기** ✅
- 에러 → 아래 진행

**이 단계는 Microsoft Store에서 수동 설치가 필요합니다. Claude Code가 직접 실행할 수 없습니다.**

**📋 사용자에게 전달 (실행 금지 — AskUserQuestion으로 텍스트만 전달):**

```
[Step 2/9] Ubuntu를 설치합니다.

wsl --install로 Ubuntu가 자동 설치되지 않았으므로, Microsoft Store에서 직접 설치합니다.

1. Microsoft Store를 열어주세요 (시작 메뉴에서 검색)
2. "Ubuntu"를 검색하세요
3. "Ubuntu" (숫자 없는 최신 버전)을 선택 → "설치" 클릭
4. 설치 완료 후 "열기"를 클릭하세요
5. 사용자 이름과 비밀번호를 설정하세요
   - 사용자 이름: 영문 소문자만 (예: myname)
   - 비밀번호: 간단하게 (매번 sudo 시 입력)
6. 설정이 완료되면 "완료"라고 알려주세요
```

**🤖 사용자가 완료를 알리면 (실행):**
```bash
wsl -d Ubuntu -- echo "Ubuntu OK: $(whoami)"
```
사용자 이름 확인 후 다음 단계.

---

### Step 3: 기본 패키지 설치 (🤖 자동 실행)

**🤖 환경 감지 (실행):**
```bash
wsl -d Ubuntu -- bash -c "tmux -V 2>/dev/null && git --version 2>/dev/null && curl --version 2>/dev/null | head -1"
```
- tmux, git, curl 모두 버전 출력 → **건너뛰기** ✅
- 하나라도 없으면 → 아래 진행 (apt install은 이미 설치된 것은 자동으로 무시하므로 전체 실행 가능)

**🤖 설치 (실행):**
```bash
wsl -d Ubuntu -- bash -c "sudo apt update && sudo apt upgrade -y && sudo apt install -y tmux git curl"
```

**🤖 확인 (실행):**
```bash
wsl -d Ubuntu -- bash -c "tmux -V && git --version"
```

---

### Step 4: nvm + Node.js 설치 (🤖 자동 + 👤 터미널 재시작)

**🤖 환경 감지 (실행):**
```bash
wsl -d Ubuntu -- bash -c "export NVM_DIR=\"\$HOME/.nvm\" && [ -s \"\$NVM_DIR/nvm.sh\" ] && . \"\$NVM_DIR/nvm.sh\" 2>/dev/null; node --version 2>/dev/null"
```
- 버전 출력 (예: `v20.x.x`) → **건너뛰기** ✅
- 에러 또는 출력 없음 → 아래 진행

**🤖 4-1. nvm 설치 (실행):**
```bash
wsl -d Ubuntu -- bash -c "curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash"
```

**📋 4-2. 터미널 재시작 안내 (실행 금지 — AskUserQuestion으로 텍스트만 전달):**

```
[Step 4/9] nvm 설치가 완료되었습니다!

nvm을 사용하려면 터미널을 재시작해야 합니다.
하지만 걱정 마세요 - 제가 다른 방법으로 계속 진행하겠습니다.

(아무 것도 하지 않으셔도 됩니다. "계속"을 눌러주세요)
```

**🤖 4-3. Node.js 설치 (실행 - nvm 경로 직접 지정):**
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [treylom/claude-agent-teams-setup](https://github.com/treylom/claude-agent-teams-setup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
