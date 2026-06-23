---
trigger: always_on
description: **This repo's main agent-assisted task:** generate kachō-e illustrations for
---

# AGENTS.md — generate bird illustrations for your location

**This repo's main agent-assisted task:** generate kachō-e illustrations for
the bird species at *your* location (the ones the bundled library is missing),
in the same style as the rest, and get them into the card.

A human can hand their AI coding agent (Claude Code, Cursor, etc.) this one
line:

> "Follow AGENTS.md to generate bird illustrations for my birds."

The agent should then work through the steps below, pausing only at the points
marked **ASK THE USER**. (Contributor build/test notes are at the very bottom.)

---

## 0. Prerequisites — ASK THE USER first

- **A billing-enabled Google Gemini API key.** Image generation is **not** on
  the free tier — a free key returns `HTTP 429 ... quota ... limit: 0`. Get one
  at <https://aistudio.google.com/apikey> and **enable billing** on its project.
  Cost is roughly **$0.04 per image** (2 images per species); a typical missing
  set runs a few dollars.
- **Python 3.10+** (always) and **Node 18+** (only to rebuild the card).
- Their **BirdNET-Go species list** — Step 1.

Don't generate anything until the user confirms a billing-enabled key. That 429
is the #1 thing that wastes a run.

---

## 1. Get the species list for your location

Pick one:

**A. Download the CSV from the BirdNET-Go dashboard** *(recommended — covers
every species likely at your location, not just what's been heard so far).*

1. Open your BirdNET-Go web UI (e.g. `http://homeassistant.local:8080`).
2. Go to the **Species** page — the per-location species list (each with an
   occurrence score). Use its **export / download** to save a **CSV**.
3. Any CSV with `Common Name` and `Scientific Name` columns works (other
   columns are ignored). Note the file path.

**B. Pull straight from the station** *(no download).* Skip the CSV and let the
pipeline read exactly what your station has detected:
`--from-birdnet http://<your-birdnet-go>:8080` (used in Step 3). Use this only
if the machine running the agent can reach your BirdNET-Go.

---

## 2. One-time setup

From the repo root:

```bash
python3 -m venv .venv
.venv/bin/pip install -r avian/scripts/requirements.txt certifi
```

**macOS only:** the system Python often can't verify HTTPS, which silently
breaks the Wikipedia + Gemini calls (`CERTIFICATE_VERIFY_FAILED`). Export a CA
bundle for **every** pipeline command in this session:

```bash
export SSL_CERT_FILE="$(.venv/bin/python -m certifi)"
```

Provide the Gemini key without putting it in shell history or git:

```bash
echo 'YOUR_GEMINI_KEY' > avian/scripts/.gemini_key   # already gitignored
export GEMINI_API_KEY="$(tr -d '[:space:]' < avian/scripts/.gemini_key)"
```

### Style references — do this; it's what keeps your birds matching the set

The bundled art was generated with ~10 public-domain Edo-period kachō-e
woodblock prints as a *style* reference (only the painting technique is
borrowed). They aren't committed (third-party scans). Fetch them:

```bash
mkdir -p avian/assets/references/styles
UA="habird-styleref/1.0 (+https://github.com/adamoberley/HABirdDashboard)"
while IFS='|' read -r fn url; do
  curl -sSL -A "$UA" -o "avian/assets/references/styles/$fn" "$url"
done <<'REFS'
01-sparrows-on-bamboo-Koson.jpg|https://upload.wikimedia.org/wikipedia/commons/a/a0/Tapuit_bij_bamboe%2C_RP-P-1999-489.jpg
02-cawing-crow-Koson.jpg|https://upload.wikimedia.org/wikipedia/commons/a/a3/Cawing_crow_by_Ohara_Koson.jpg
03-jays-on-berry-tree-Koson.jpg|https://upload.wikimedia.org/wikipedia/commons/b/b1/Koson_-_jays-on-berry-tree.jpg
04-kingfisher-Koson.jpg|https://upload.wikimedia.org/wikipedia/commons/1/19/IJsvogel%2C_AK-RAK-2000-9.jpg
05-owl-on-ginkgo-Koson.jpg|https://upload.wikimedia.org/wikipedia/commons/9/90/Scops_Owl%2C_Cherry_Blossoms%2C_and_Moon_by_Sh%C5%8Dson.jpg
06-goose-flying-in-moonlight-Koson.jpg|https://upload.wikimedia.org/wikipedia/commons/7/74/Gans_bij_volle_maan%2C_RP-P-1999-503.jpg
07-swallows-in-flight-Koson.jpg|https://upload.wikimedia.org/wikipedia/commons/4/4c/Drie_roodstuitzwaluwen_in_duikvlucht%2C_RP-P-1999-400.jpg
08-crane-in-small-water-Koson.jpg|https://upload.wikimedia.org/wikipedia/commons/1/1a/Vissende_kraanvogel_in_ondiep_water%2C_RP-P-2005-470.jpg
09-cockatoo-Yoshida.jpg|https://upload.wikimedia.org/wikipedia/commons/c/cf/Twee_kaketoes_op_tak_met_pruimenbloesem%2C_RP-P-2005-472.jpg
10-mandarin-ducks-Yoshida.jpg|https://upload.wikimedia.org/wikipedia/commons/9/99/Mandarijneenden%2C_RP-P-1999-568.jpg
REFS
# Shrink to ~1024px so each API request stays small:
.venv/bin/python - <<'PY'
from pathlib import Path
from PIL import Image
for f in sorted(Path("avian/assets/references/styles").glob("*.jpg")):
    im = Image.open(f).convert("RGB"); w, h = im.size
    if max(w, h) > 1024:
        s = 1024 / max(w, h); im = im.resize((round(w*s), round(h*s)), Image.LANCZOS)
    im.save(f, "JPEG", quality=88, optimize=True)
PY
```

(Generation still works without these, but the style drifts from the bundled
set — so don't skip it if you want them to match.)

---

## 3. Generate → cut out → masks → card

If you used the **CSV** (option A), convert it to the `Sci|Com` label format the
pipeline reads (skip if using `--from-birdnet`):

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adamoberley/HABirdDashboard](https://github.com/adamoberley/HABirdDashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-23 -->
