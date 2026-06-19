---
trigger: always_on
description: 转写结果父目录；实际输出保存到该目录下的 transcribe-results 子目录
---


# Transcribe — 音视频转写 Skill

将音视频内容转写为带时间戳、说话人标注的 Markdown 文字稿。

## 支持的输入方式

| 方式 | 示例 |
|------|------|
| 在线视频 URL | 支持 yt-dlp 可解析的视频平台链接 |
| 本地音视频文件路径 | `/path/to/video.mp3` |
| 已公开的 HTTP 文件 URL | `https://example.com/audio.mp4` |

**输出**：保存到 Step 0.5 确认的目录下：

| 文件 | 内容 |
|------|------|
| `{ts}-{name}-{taskId}.md` | 转写全文（带时间戳、说话人标注）+ 摘要总结 |
| `{ts}-{name}-{taskId}_meta.json` | task_id、原文件名、本地路径、OSS 路径、听悟临时 URL（兜底用） |

---

## Step 0：依赖自检（每次调用前执行）

**第一步：确定 skill 根目录的绝对路径**

skill 根目录是 SKILL.md 所在的目录。用以下命令获取并记录，后续所有步骤都以此为基准：

```bash
SKILL_ROOT=$(python3 -c "from pathlib import Path; print(Path('scripts/pipeline.py').resolve().parent.parent)")
```

若上述命令因 CWD 不同而不准确，改用脚本的绝对路径推算：

```bash
SKILL_ROOT=$(python3 -c "from pathlib import Path; import sys; print(Path(sys.argv[1]).resolve().parent.parent)" "$SKILL_ROOT/scripts/pipeline.py")
```

或直接用已知的安装路径：

```
~/.agents/skills/transcribe
~/.claude/skills/transcribe
~/.openclaw/skills/transcribe
~/.hermes/skills/media/transcribe
```

**第二步：运行依赖自检**

```bash
bash "$SKILL_ROOT/scripts/setup.sh"
```

幂等脚本：会在 skill 根目录创建 `.venv`，安装 Python 依赖，并检查 `ffmpeg` 与 `yt-dlp`。后续 Python 和 yt-dlp 命令必须使用：

```bash
PYTHON_BIN="$SKILL_ROOT/.venv/bin/python3"
YT_DLP_BIN="$SKILL_ROOT/.venv/bin/yt-dlp"
```

**第三步：检查 `.env` 文件**

检查 `$SKILL_ROOT/.env` 是否存在：
- **存在** → 继续
- **不存在** → 停止，提示用户：
  ```
  未找到 .env 文件，请在 skill 根目录下创建：
    cp "$SKILL_ROOT/.env.example" "$SKILL_ROOT/.env"
  然后编辑 .env 填入真实凭证（参考 .env.example 中的字段说明）
  ```

**第四步：解析 cookie 路径并优先复用本地缓存**

处理在线视频 URL 前，先检查 `$SKILL_ROOT/cookies/` 下是否已有当前来源 cookie：
- **已有且非空** → 直接使用，不要再从浏览器导出；
- **没有或后续 yt-dlp 仍因登录/风控失败** → 再提示用户选择导出或手动提供。

若没有可用 cookie，提示用户：
- 先在本机浏览器登录目标来源，然后允许脚本从浏览器导出 cookie；
- 或手动提供 Netscape 格式 cookie 文件，并保存到 `cookies/<source-alias>.cookies.txt`。

```bash
bash "$SKILL_ROOT/scripts/prepare_cookies.sh" "<视频URL>" chrome
```

脚本会根据 URL 自动生成来源别名，必要时自动创建 `$SKILL_ROOT/cookies/`，并使用固定 cookie 文件：

```
$SKILL_ROOT/cookies/<source-alias>.cookies.txt
```

默认情况下，来源别名由 URL host 自动生成。

重要：不要读取或修改 `.env` 来处理 cookie 复用；`.env` 只用于敏感凭证和基础配置。

新窗口或新会话处理在线视频 URL 时，先检查 `$SKILL_ROOT/cookies/` 下已有的 `*.cookies.txt` 文件。同一内容来源可能同时存在主域名、短链域名、移动端域名、分享域名等多个入口，这些 URL 生成的默认来源别名可能不同，但实际登录态 cookie 往往可以复用。模型需要根据 URL 语义和已有 cookie 文件名做一次兼容判断：如果目标 URL 明显属于已有 cookie 覆盖的同一来源，不要重新导出 cookie，优先复用已有文件。

复用方式：使用已有 cookie 文件名去掉 `.cookies.txt` 后得到的别名设置 `TRANSCRIBE_COOKIE_ALIAS`，再运行 `prepare_cookies.sh`。例如已有：

```
$SKILL_ROOT/cookies/source-a.cookies.txt
```

则复用时运行：

```bash
TRANSCRIBE_COOKIE_ALIAS=source-a bash "$SKILL_ROOT/scripts/prepare_cookies.sh" "<视频URL>" chrome
```

如果无法从 URL 生成来源别名，不使用默认 cookie。可临时指定来源别名：

```bash
TRANSCRIBE_COOKIE_ALIAS=source-a bash "$SKILL_ROOT/scripts/prepare_cookies.sh" "<视频URL>" chrome
```

随后按当前 URL 获取来源 cookie：

```bash
COOKIE_ALIAS=$("$PYTHON_BIN" - "<视频URL>" "$SKILL_ROOT" <<'PY'
import sys
from pathlib import Path
sys.path.insert(0, str(Path(sys.argv[2]) / "scripts"))
from config import cookie_alias_from_url
print(cookie_alias_from_url(sys.argv[1]))
PY
)

COOKIE_PATH="$SKILL_ROOT/cookies/${COOKIE_ALIAS}.cookies.txt"

if [ -n "$COOKIE_ALIAS" ] && [ -s "$COOKIE_PATH" ]; then
  COOKIES_OPT=(--cookies "$COOKIE_PATH")
  echo "Cookie 文件已就绪：$COOKIE_PATH"
else
  COOKIES_OPT=()
  echo "未找到当前来源 cookie，yt-dlp 以匿名模式运行"
fi
```

后续所有 yt-dlp 命令统一使用 `"${COOKIES_OPT[@]}"`，无需手动拼路径。
若已设置 `COOKIES_OPT` 但 yt-dlp 仍报登录、403、412、风控或 cookie 失效类错误，停止当前转写，提示用户重新登录浏览器后再次运行 `prepare_cookies.sh` 导出，不要反复匿名重试。

---

## Step 0.5：输出路径确认（每次调用前执行）

默认原则：**除非用户明确要求保存到某个目录，否则不要在命令里显式传 `--output-dir`**。让 `pipeline.py` / `get_result.py` 自己按下列规则解析输出目录。

Codex 运行时可能有自己的 workspace/output 目录约定；本 skill 不主动使用该目录。用户未指定输出位置时，优先使用 skill 安装目录的上一层创建 `transcribe-results`，避免不同项目 workspace 启动导致结果分散。

读取 `$SKILL_ROOT/.env` 中的 `TRANSCRIBE_OUTPUT_PARENT_DIR`：

- **不为空且路径存在** → 输出到：
  ```
  <TRANSCRIBE_OUTPUT_PARENT_DIR>/transcribe-results
  ```
  若该目录还不存在，脚本会自动创建。

- **为空或路径不存在** → 根据 skill 安装位置自动推导：
  ```
  ~/.agents/skills/transcribe   -> ~/.agents/transcribe-results
  ~/.claude/skills/transcribe   -> ~/.claude/transcribe-results
  ~/.openclaw/skills/transcribe -> ~/.openclaw/transcribe-results
  ~/.hermes/skills/media/transcribe -> ~/.hermes/skills/media/transcribe-results
  ```

Hermes 若通过 `metadata.hermes.config` 配置了 `transcribe.output_parent_dir`，需要在运行前把该值同步到 `.env` 的 `TRANSCRIBE_OUTPUT_PARENT_DIR`。不要因为平台存在默认 output 目录就主动覆盖本 skill 的默认目录策略。

只有当用户明确说“保存到/输出到某个目录”时，才使用命令行 `--output-dir PATH` 临时覆盖上述规则。

---

## Step 1：执行转写

> 以下命令中 `$SKILL_ROOT` 为 skill 根目录绝对路径（Step 0 已确定）。
> 若 Step 0 解析出了 `COOKIES_OPT`，每条 yt-dlp 命令都使用它；未配置 cookie 时该数组为空。

### 方式 A：在线视频 URL（yt-dlp 可解析的平台）

```bash
TITLE=$("$YT_DLP_BIN" "${COOKIES_OPT[@]}" --get-title "<视频URL>")
"$YT_DLP_BIN" "${COOKIES_OPT[@]}" -x --audio-format mp3 -o - "<视频URL>" | "$PYTHON_BIN" "$SKILL_ROOT/scripts/pipeline.py" --stdin "${TITLE}.mp3"
```

### 方式 B：本地文件

```bash
"$PYTHON_BIN" "$SKILL_ROOT/scripts/pipeline.py" "/path/to/file.mp3"
```

### 方式 C：已有公开 HTTP URL（文件可被听悟服务器直接访问）

```bash
"$PYTHON_BIN" "$SKILL_ROOT/scripts/pipeline.py" "https://example.com/audio.mp3"
```

### 方式 D：yt-dlp 取直链 → 先上传 OSS 再转写

适用场景：视频平台 CDN 直链有时效或需要 Headers，听悟无法直接拉取，需经 OSS 中转。

```bash
TITLE=$("$YT_DLP_BIN" "${COOKIES_OPT[@]}" --get-title "<视频URL>")
DIRECT_URL=$("$YT_DLP_BIN" "${COOKIES_OPT[@]}" -g "<视频URL>" | head -1)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kkzeros/transcribe](https://github.com/kkzeros/transcribe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
