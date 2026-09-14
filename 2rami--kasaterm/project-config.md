---
trigger: always_on
description: 이 레포는 **"AI 에이전트와 함께 일하는 1인 개발자를 위한, 터미널을 코어로 둔 작업 OS"** 다. 한 모노레포에 3층이 쌓여 있고, **나누지 않는다** — 세 층이 같이 진화 중이라 레포를 찢으면 cross-repo 오버헤드만 폭발한다(1인 개발). 분리는 경계 정리의 *결과*지 시작이 아니다.
---

# 정체성 — 3층 작업 OS (모노레포)

이 레포는 **"AI 에이전트와 함께 일하는 1인 개발자를 위한, 터미널을 코어로 둔 작업 OS"** 다. 한 모노레포에 3층이 쌓여 있고, **나누지 않는다** — 세 층이 같이 진화 중이라 레포를 찢으면 cross-repo 오버헤드만 폭발한다(1인 개발). 분리는 경계 정리의 *결과*지 시작이 아니다.

| 층 | 코드네임 | 역할 | 상태 |
|---|---|---|---|
| ① 엔진 | **kasaterm** | 터미널 — wgpu 셀 렌더 · PTY · 한글 IME · multipane | 거의 안정기 |
| ② 작업환경 | **kasaspace** | 파일트리 · git 관리 · pane 간 에이전트 연결 | 진행 중 |
| ③ 오케스트레이션 | **blueclaudearchive** | 여러 Claude를 학생처럼 거느리는 하네스 GUI (아로나 모드) | 무게중심 |

**새 기능은 자기 층에 둔다** — 렌더/입력=①, 파일·git·에이전트 배선=②, 학생·재화·협업·경험=③. 무게중심은 ②③로 올라가는 중이라 ①은 "③를 떠받칠 만큼만" 만지면 된다. 레포 분리 트리거(미래): 터미널 엔진을 남이 임베드할 라이브러리로 배포 / ③를 터미널 없이 독립 제품으로 팔 때 — 지금은 둘 다 아님.

---

kasaterm (자체 tmux GUI 터미널 + Claude 런처). 사용자: 거노 (디자이너→개발 입문). 패키지/바이너리명 = **`kasaterm`**.

[!] 작업 시작 전 반드시 [.memory/MEMORY.md](file://./.memory/MEMORY.md) 를 먼저 읽어라 — 거노님 개발 성향·todos·피드백, 그리고 **맨 위 핸드오프 블록**(직전 세션이 어디서 멈췄는지)부터 파악. 렌더러·색·아키텍처 배경, 렌더버그 카탈로그, 코드 수정 주의점은 전부 거기 토픽 파일에 있다.

## 커밋·push 는 묻지 말고 한다 (2026-08-25 지시)

이 레포에서는 **커밋도 push 도 승인을 기다리지 않는다.** 글로벌 규칙의 「push 는 물어볼 것」을
여기서만 뒤집는 것이니, 다른 레포로 이 습관을 들고 가지 마라.

이유는 미반영분이 쌓이는 비용이다. pane 여럿이 같은 워킹트리를 쓰므로, 안 올린 커밋은 다른
기계·다른 세션에서 없는 것과 같고 남이 그 위에 덮어쓴다. 작업 하나가 끝나면 커밋하고 올리는
데까지가 한 단위다.

**그래도 물어봐야 하는 것 셋** — push 가 자유로워진 것과 뜻이 다르다.

- **브랜치 전환·checkout** — 워킹트리를 여럿이 함께 쓴다. 네가 옮기면 남의 pane 이 통째로 딸려간다.
- **force push·history 재작성**(`rebase` 뒤 강제 push, `reset --hard` 후 push) — 일반 push 는
  되돌릴 수 있지만 이건 남의 커밋을 지운다.
- **남의 브랜치·`main` 밖으로 올리는 것** — 올릴 자리가 평소와 다르면 한 번 확인한다.

## 커밋 공동저자 — 나쵸네코를 함께 단다

이 레포의 커밋은 **공동저자 줄을 둘** 단다. 기존 모델 줄은 그대로 두고, 그 아래에

```
Co-Authored-By: NachoNekoBot <322779791+NachoNekoBot@users.noreply.github.com>
```

를 덧붙인다. GitHub 은 공동저자도 기여자로 세는데, 계정에 연결된 이메일일 때만 센다 —
모델 줄의 `noreply@anthropic.com` 은 어느 계정에도 안 걸려 목록에 안 뜬다(2026-08-30 확인).
이 주소는 이 프로젝트의 기계용 계정이라 목록에 뜬다.

모델 줄을 지우지 마라. 코드를 실제로 쓴 것이 무엇인지가 기록에서 사라진다.

**공개 레포다**(`2rami/kasaterm`, public). push 가 자유로워졌다고 담는 내용까지 자유로운 게
아니다 — 키·토큰은 커밋에도 로그에도 넣지 말고, 주석·문서·커밋 메시지의 개인 호칭 금지도 그대로다.

## 자율 테스트 우선

사용자에게 "테스트 해보세요"라고 떠넘기지 말고 **너가 직접** 실행·확인·수정 사이클을 돌려라.

### ⛔ 검증용 앱을 띄우고 거두는 법 — 이 블록을 어기면 사용자 세션이 통째로 날아간다

2026-08-15 실측: 검증용 앱을 띄웠다가 `pkill -f "target/debug/kasaterm"` 으로 거뒀더니 **사용자 창의 pane 9개에서 claude 가 전부 종료**됐다(전부 "Resume this session with:" 를 남기고 셸로 돌아갔다). 원인은 둘 중 하나이고 **둘 다 아래 규칙 하나로 막힌다** — ①`pkill -f` 의 패턴이 의도보다 넓게 잡혔거나 ②새로 띄운 앱이 같은 `session.json` 을 읽어 **같은 세션 id 로 `claude --resume` 을 다시 열어** 먼저 열려 있던 쪽을 밀어냈거나. 사용자는 자기 창이 왜 비었는지 알 방법이 없다.

**띄울 때 — 세 개를 반드시 함께 준다.**

```bash
KASATERM_SESSION_FILE=/tmp/<네이름>-session.json \
KASATERM_SETTINGS_FILE=/tmp/<네이름>-settings.json \
KASATERM_WINDOW_FILE=/tmp/<네이름>-window.json \
KASATERM_AUTORESTORE=fresh \
KASATERM_STUDENTS_DIR=/tmp/<네이름>-students \
KASATERM_AUTOQUIT_MS=120000 \
./target/debug/kasaterm > /tmp/<네이름>-app.log 2>&1 &
APP=$!                     # 거둘 때는 이 PID 만: kill $APP
```

- **`KASATERM_SESSION_FILE`·`KASATERM_SETTINGS_FILE` 은 선택이 아니다.** 안 걸면 검증용 앱이 사용자의 `~/.config/kasaterm/session.json` 을 읽고, **실행 중 5초마다 자기 상태로 덮어쓴다**. 설정 파일 쪽은 사용자가 손수 적은 계정 라벨을 하네스 값으로 덮은 전례가 있다. 실데이터가 있어야 화면이 성립하면 원본을 스크래치로 **복사**해 그걸 가리켜라 — 빈 파일을 가리키면 검증하려던 UI 자체가 안 뜬다.
- **`KASATERM_AUTORESTORE=fresh`** — 저장된 세션을 복원하지 않고 빈 창으로 뜬다. 사용자 pane 의 claude 세션과 같은 id 를 다툴 경로가 사라지고, 캡처가 복원 모달만 찍는 일도 없어진다.
- **`KASATERM_STUDENTS_DIR`** 로 그림 폴더를 격리한다 — 업로드·삭제를 검증하면서 사용자가 실제로 쓰는 `~/.config/kasaterm/students/` 를 건드리지 않는다.
- **`KASATERM_WINDOW_FILE`** 도 빠뜨리지 마라. 안 걸면 검증용 앱이 자기 창 크기를 사용자의 `window.json` 에 적어 두고, 다음에 사람이 앱을 열 때 엉뚱한 크기로 뜬다(2026-08-31 실제로 덮었다 — 세션은 격리해 무사했는데 이 하나가 목록에 없었다).
- **격리 창구가 없는 것도 안다.** `session_characters.json`(세션↔학생 바인딩)과 `caps.json` 은 언제나 `~/.config/kasaterm/` 을 쓴다. 다만 전자는 **읽고 병합해** 쓰므로 검증 세션의 학생이 항목으로 늘 뿐 사람의 바인딩을 지우지는 않고, 후자는 같은 터미널이면 같은 값이라 무해하다. 그래도 깨끗하게 두고 싶으면 검증 전에 복사해 두고 끝나면 되돌려라.
- ⚠️ **`KASATERM_SOCKET_PATH` 를 띄울 때도 주고, `kasaterm-cli` 를 부를 때도 줘라.** CLI 는 `$KASATERM_SOCKET_PATH > $CMUX_SOCKET_PATH > /tmp/cmux.sock` 순으로 붙고 **포트는 안 본다** — 리그를 다른 포트로 띄웠어도 CLI 에 이 변수를 안 주면 그 명령이 **사용자 앱으로 간다**(2026-09-03 실측: `split right` 이 사용자 창에 pane 을 만들었고, `send` 가 도는 학생의 입력창에 글자를 밀어넣었다). 앱 부팅 줄과 CLI 호출 양쪽에 같은 경로를 걸어라:
  `export KASATERM_SOCKET_PATH=/tmp/<네이름>/rig.sock` 를 먼저 하고 그 셸에서 둘 다 부르는 것이 가장 안전하다.
- **화면을 판정할 때 `kasaterm-cli capture <pane>` 을 믿지 마라 — 원본 글자판이다.** 그건 `capture_pane_offscreen` 이라 `t.cells` 를 그대로 찍어, 렌더러가 화면을 만들며 하는 일(스크롤백 당김·입력창 붙잡기 같은 재구성)이 **안 보인다**. 렌더 변경을 눈으로 확인하려면 `kasaterm-cli capture --window <path>`(창 프레임) 이나 `KASATERM_AUTOCAPTURE_MS`/`_PATH` 를 써라. 2026-09-03 에 이걸 몰라 "코드가 안 돈다"고 한동안 오판했다(계측을 심어 보니 값은 맞게 돌고 있었다).
- **포트는 지정하지 마라.** 8765 가 사용자 앱 것이므로 새 앱은 알아서 다른 포트를 고른다. 그 번호는 로그에서 읽어라:
  `P=$(grep -o "HTTP MCP on 127.0.0.1:[0-9]*" /tmp/<네이름>-app.log | tail -1 | grep -o "[0-9]*$")`
- **색을 검증한다면 `NO_COLOR` 를 먼저 걷어내라.** claude code 의 셸 도구는 이 변수를 켜 두는데, 거기서 앱을 띄우면 앱을 거쳐 **pane 의 셸까지** 물려간다. 그러면 셸이 스스로 색을 끄고(PowerShell 7 은 `$PSStyle.OutputRendering` 이 `PlainText` 로 내려간다) 화면이 죄다 흑백으로 나온다 — 렌더러는 멀쩡한데 없는 버그를 쫓게 된다(2026-08-31 실제로 한 번 속았다). 의심되면 pane 안에서 `$PSStyle.OutputRendering` 과 `$env:NO_COLOR` 를 찍어 봐라: `Host` 와 빈 값이면 정상이다. claude 마커(`CLAUDE_MARKER_ENV`)와 달리 앱이 지워 주지 않는다 — 사용자가 일부러 켰을 수도 있는 값이라 터미널이 함부로 뺏으면 안 된다.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [2rami/kasaterm](https://github.com/2rami/kasaterm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
