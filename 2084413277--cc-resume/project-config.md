---
trigger: always_on
description: Build single-page A4 Chinese-language resumes as HTML/CSS, then export to PDF via headless Edge/Chrome. Covers Sino-Western typography pairing, user-driven theme colors, strict A4 fit (210mm × 297mm), and pitfalls when iterating with the user.
---


# Single-Page A4 Chinese Resume — HTML → PDF

## When to use

The user wants a polished CV / 简历 they can print or attach to applications. They will iterate visually: the layout, color, fonts, bullet density, and section ordering will all change multiple times before they're satisfied. Optimize for fast, low-friction iteration over a "perfect first draft."

## Workflow at a glance

1. **Read the source** (`*.txt`, `*.md`, `*.docx`, `*.pptx`, `*.pdf`) and extract every fact verbatim. Don't paraphrase or invent.
2. **Extract the embedded photo** from the source if there is one — see [Extracting the portrait](#extracting-the-portrait) below.
3. **Build one HTML file per person.** Inline `<style>` — no external CSS file. Each resume is self-contained.
4. **Render to PDF** with headless Edge (Chrome works too):
   ```powershell
   & $edge --headless=new --disable-gpu --no-pdf-header-footer --no-sandbox `
     --print-to-pdf="<out>.pdf" "file:///<absolute-path-to-html>"
   ```
5. **Verify page count** by parsing the PDF — see "Page-count check" below.
6. **Iterate.** Each user remark is usually one of: change a fact, change a color, change a font, change a bullet structure, or "fit on one page."

## Extracting the portrait

Most source files (`.docx`, `.pptx`, the user's exported PDF resume) embed the candidate's portrait. **Always check for an embedded photo before falling back to a `PHOTO` placeholder** — the user expects the layout to look complete on first render.

`.docx` and `.pptx` are ZIP archives. The portrait lives at:
- `.docx` → `word/media/image1.{png,jpeg}` (and image2, image3, ...)
- `.pptx` → `ppt/media/image1.{png,jpeg}`
- `.pdf` → use `pdfimages -j input.pdf prefix` (poppler-utils) to dump every embedded image

### PowerShell (Windows)

```powershell
# Treat the file as a zip and extract media/
$src  = "E:\path\to\个人简历.pptx"
$temp = "$env:TEMP\portrait-extract"
if (Test-Path $temp) { Remove-Item $temp -Recurse -Force }
Add-Type -AssemblyName System.IO.Compression.FileSystem
[System.IO.Compression.ZipFile]::ExtractToDirectory($src, $temp)

# pptx → ppt/media; docx → word/media
$mediaDir = if (Test-Path "$temp\ppt\media") { "$temp\ppt\media" } else { "$temp\word\media" }
Get-ChildItem $mediaDir | Format-Table Name, Length

# Copy the largest image (usually the portrait) into your project
$largest = Get-ChildItem $mediaDir | Sort-Object Length -Descending | Select-Object -First 1
Copy-Item $largest.FullName "<your-cv-folder>\photo.jpg"
```

### Bash (macOS / Linux)

```bash
src="$HOME/path/to/resume.pptx"
tmp=$(mktemp -d)
unzip -q "$src" -d "$tmp"
mediadir=$(ls -d "$tmp"/{ppt,word}/media 2>/dev/null | head -n1)
ls -laS "$mediadir"
# largest image is usually the portrait
cp "$(ls -S "$mediadir"/* | head -n1)" "<your-cv-folder>/photo.jpg"
```

### Inserting the photo

Once `photo.jpg` (or `.png`) sits next to the resume HTML:

```html
<img class="header-photo" src="photo.jpg" alt="证件照" />
```

The template's `.header-photo` style (`object-fit: contain` / `cover`) will frame it correctly at 80×110 or whatever you've set.

**Self-contained alternative — inline as data URI** (so the HTML is a single distributable file with no asset dependency):

```powershell
$bytes = [System.IO.File]::ReadAllBytes("photo.jpg")
$b64   = [Convert]::ToBase64String($bytes)
"data:image/jpeg;base64,$b64" | Set-Clipboard
# paste into <img src="..." />
```

Trade-off: inline data URI bloats HTML (~250 KB for a typical portrait) but lets the user email a single `.html` without a missing-image broken icon.

### When there's no photo

If the source has no embedded image, **ask the user**: *"源文件里没有证件照，你提供一张吗？或者先用占位符？"* Don't silently render a placeholder — Chinese resumes default to having a portrait, and skipping it without checking is usually wrong.

If the user opts for a placeholder (e.g. they're still picking the photo), use a soft gradient with a ghosted "PHOTO" wordmark — looks intentional rather than broken:

```html
<div class="header-photo">PHOTO</div>
```
```css
.header-photo {
  background: linear-gradient(135deg, #d1d5db 0%, #e5e7eb 100%);
  display: flex; align-items: center; justify-content: center;
  font-family: "Newsreader", serif;
  font-style: italic;
  font-size: 8pt;
  color: #f3f4f6;
}
```

## Project layout

```
cv/
  resume_<personA>.html         ← one HTML per candidate
  resume_<personB>.html
  resume_<personA>.pdf / resume_<personB>.pdf
  photo_<personA>.jpg, photo_<personB>.png   ← passport-style portraits
  logos/                                     ← school / company logos as round masks
    <school1>.png, <school2>.png, ...
  fonts/                                     ← woff2 fonts, embedded via @font-face
    noto-sans-sc-{400,500,600,700}.woff2
    noto-serif-sc-{400,600,700}.woff2
    ibm-plex-sans-{400,500,600,700}.woff2
    newsreader-{400,400-italic,600}.woff2
  raw_<personA>.txt / raw_<personB>.md       ← original unedited content
```

Keep raw source files alongside the HTML — when the user says "go back to the original wording" you'll need them.

## Typography


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [2084413277/cc-resume](https://github.com/2084413277/cc-resume) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
