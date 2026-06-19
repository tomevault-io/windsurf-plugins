---
trigger: always_on
description: >
---


# Video Subtitle Extractor (YouTube + Bilibili)

Detect platform → download subtitles → clean → save raw → generate summary.

---

## Step 1 — Ensure yt-dlp is available

```bash
if ! command -v yt-dlp &>/dev/null; then
  echo "yt-dlp not found, installing..."
  pip install -q yt-dlp || pip3 install -q yt-dlp
fi
yt-dlp -U --quiet 2>/dev/null || true
```

If installation fails, stop and tell the user to install yt-dlp manually (`pip install yt-dlp` or `brew install yt-dlp`).

---

## Step 2 — Detect platform and download subtitles

Detect whether the URL is Bilibili or YouTube, then use the appropriate strategy.

```bash
URL="<user-provided URL>"
TMPDIR=$(mktemp -d)
SUB_FILE=""
SUBTITLE_LANG=""

# Detect platform
if echo "$URL" | grep -qE '(bilibili\.com|b23\.tv)'; then
  PLATFORM="bilibili"
  SITE_NAME="Bilibili"
  SITE_DOMAIN="bilibili.com"
else
  PLATFORM="youtube"
  SITE_NAME="YouTube"
  SITE_DOMAIN="youtube.com"
fi
```

### Bilibili branch

Bilibili subtitles require login cookies. Always use a cookies file — refresh from Chrome if missing or stale (>30 days):

```bash
if [ "$PLATFORM" = "bilibili" ]; then
  BILI_COOKIES="${BILIBILI_COOKIES_FILE:-$HOME/bilibili_cookies.txt}"

  NEED_REFRESH=false
  if [ ! -f "$BILI_COOKIES" ]; then
    NEED_REFRESH=true
  elif [ "$(find "$BILI_COOKIES" -mtime +30 2>/dev/null | wc -l | tr -d ' ')" -gt 0 ]; then
    echo "Bilibili cookies older than 30 days, refreshing..."
    NEED_REFRESH=true
  fi

  if [ "$NEED_REFRESH" = true ]; then
    echo "Reading cookies from Chrome (one-time keychain prompt)..."
    yt-dlp --cookies-from-browser chrome --cookies "$BILI_COOKIES" \
      --skip-download -i "https://www.bilibili.com/" 2>/dev/null
  fi

  COOKIE_ARGS="--cookies $BILI_COOKIES"

  # List available subtitle langs — capture stderr to detect login failure
  LIST_OUTPUT=$(yt-dlp --list-subs $COOKIE_ARGS "$URL" 2>&1)
  if echo "$LIST_OUTPUT" | grep -qi "login\|not logged\|需要登录\|please log"; then
    echo ""
    echo "❌ Bilibili cookies expired or invalid."
    echo "   Fix: delete the cookies file and retry — it will re-read from Chrome."
    echo "   rm \"$BILI_COOKIES\""
    rm -rf "$TMPDIR"
    exit 1
  fi
  AVAIL_LANGS=$(echo "$LIST_OUTPUT" | awk '/^[a-z]/{print $1}' | grep -v "^Language$")

  # Try ai-zh first, then any zh variant, then en
  for lang in ai-zh zh-Hans zh-CN zh en; do
    if echo "$AVAIL_LANGS" | grep -q "^${lang}$"; then
      yt-dlp \
        --write-sub \
        --sub-langs "$lang" \
        --skip-download \
        --retries 3 \
        -o "$TMPDIR/bili_%(id)s" \
        $COOKIE_ARGS \
        "$URL" 2>/dev/null
      SUB_FILE=$(find "$TMPDIR" -maxdepth 1 -name "*.${lang}.*" 2>/dev/null | head -1)
      if [ -n "$SUB_FILE" ]; then
        SUBTITLE_LANG="$lang"
        break
      fi
    fi
  done
fi
```

### YouTube branch

```bash
if [ "$PLATFORM" = "youtube" ]; then
  for lang in zh-Hans zh-CN zh en; do
    yt-dlp \
      --write-subs \
      --write-auto-subs \
      --sub-langs "$lang" \
      --skip-download \
      --sub-format vtt \
      --retries 3 \
      --sleep-requests 1 \
      -o "$TMPDIR/yt_%(id)s" \
      "$URL" 2>/dev/null
    SUB_FILE=$(find "$TMPDIR" -maxdepth 1 -name "*.${lang}.vtt" 2>/dev/null | head -1)
    if [ -n "$SUB_FILE" ]; then
      SUBTITLE_LANG="$lang"
      break
    fi
    sleep 1
  done
fi
```

### Fail if no subtitles

```bash
if [ -z "$SUB_FILE" ]; then
  echo "No subtitles found for this video."
  echo "  - No manually uploaded subtitles"
  echo "  - No auto-generated subtitles"
  echo "Cannot proceed without a transcript."
  rm -rf "$TMPDIR"
  exit 1
fi
```

---

## Step 3 — Clean subtitle file → plain text + timestamped text

Produce two outputs from the same subtitle file:

- `cleaned.txt` — no timestamps, for LLM summary input (Steps 7–9)
- `timestamped.txt` — `[mm:ss] text` format, for saving to vault and chapter inference

```bash
EXT="${SUB_FILE##*.}"

python3 - "$SUB_FILE" "$EXT" "$TMPDIR" <<'EOF'
import sys, html, re

sub_file, ext, tmpdir = sys.argv[1], sys.argv[2], sys.argv[3]

def parse_time(t):
    t = t.strip().replace(',', '.')
    parts = t.split(':')
    if len(parts) == 3:
        return int(parts[0]) * 3600 + int(parts[1]) * 60 + float(parts[2])
    return int(parts[0]) * 60 + float(parts[1])

def fmt(secs):
    secs = int(secs)
    return f"[{secs // 60:02d}:{secs % 60:02d}]"

lines = open(sub_file, encoding='utf-8').read().splitlines()

cleaned_out = []
ts_out = []
seen = set()
current_secs = 0

if ext == 'srt':
    i = 0
    while i < len(lines):
        line = lines[i].strip()
        if re.match(r'^\d+$', line):            # sequence number
            i += 1
            continue
        m = re.match(r'^(\d{2}:\d{2}:\d{2}[,\.]\d+) -->', line)
        if m:
            current_secs = parse_time(m.group(1))
            i += 1
            continue
        text = html.unescape(re.sub(r'<[^>]+>', '', line)).strip()
        if text and text not in seen:
            seen.add(text)
            cleaned_out.append(text)
            ts_out.append(f"{fmt(current_secs)} {text}")
        i += 1
else:  # vtt
    i = 0
    while i < len(lines):
        line = lines[i].strip()
        if not line or line.startswith('WEBVTT') or line.startswith('NOTE') \
                or line.startswith('Kind:') or line.startswith('Language:'):
            i += 1

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xiapuyang/youtube-summary](https://github.com/xiapuyang/youtube-summary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
