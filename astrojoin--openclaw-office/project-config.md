---
trigger: always_on
description: Read, create, and edit Word, PowerPoint, Excel, and OneDrive files; send and read Outlook email; manage Outlook calendar — all via Microsoft Graph API.
---


# OpenClaw Office

Use this skill when the user needs to work with Microsoft 365 cloud files (OneDrive, Word, PowerPoint, Excel), Outlook email, or Outlook calendar.

## Prerequisites

1. **Azure app registered** with Device Code flow enabled. Client ID in `config.json`.
2. **Authenticated** — run `python3 scripts/auth.py login` first. Opens a device-code flow; user visits URL and enters code.
3. **Token valid** — tokens auto-refresh; check with `python3 scripts/auth.py status`.

If auth is missing or expired, tell the user to run `auth.py login` before continuing.
4. **Python packages** — install with `pip3 install --break-system-packages python-docx python-pptx msal requests pillow lxml`:
   - `python-docx` — Word (.docx) creation and editing
   - `python-pptx` — PowerPoint (.pptx) creation and editing
   - `msal` — Microsoft Authentication Library (OAuth2 device-code flow)
   - `requests` — HTTP client for Graph API calls
   - `pillow` — Image handling for `add_picture` in Word/PowerPoint
   - `lxml` — XML parsing (dependency of python-docx/python-pptx)

## Architecture

```
auth.py          → OAuth2 device-code flow + token refresh (MSAL)
graph_client.py  → Low-level Graph API HTTP client + Excel workbook/session
onedrive.py      → Cloud file operations (list, upload, download, move, copy, delete, search)
word.py          → Offline .docx CRUD (python-docx, bytes-in/bytes-out)
powerpoint.py    → Offline .pptx CRUD (python-pptx, bytes-in/bytes-out)
mail.py          → Outlook mail (list, search, read, send, reply, forward, move, delete)
outlook_calendar.py → Outlook calendar (list, create, update, delete, accept/decline)
```

Key principle: `word.py` and `powerpoint.py` are **purely offline** editors — they operate on bytes, no network calls. `onedrive.py` is the cloud bridge: it downloads bytes, passes them to word/pptx for editing, then uploads the result. Excel operations go through `graph_client.py` workbook/session endpoints (server-side editing).

## Capabilities

> ⚠️ **This section is API reference only.** Do NOT copy-paste these code blocks into `exec`. For actual execution, use the **Workflows** section below — it has the correct `exec` inline pattern with triggers.

### 1. Authentication

```bash
python3 scripts/auth.py login    # Device-code flow
python3 scripts/auth.py status   # Check token validity
python3 scripts/auth.py token    # Print current access token
python3 scripts/auth.py logout   # Delete stored tokens
```

Programmatic: `auth.get_access_token()` returns a valid token or `None`.

### 2. OneDrive Files

```python
from onedrive import OneDrive
od = OneDrive()

od.list("/Documents")            # List folder contents
od.info("/Documents/file.docx")  # File metadata
od.search("report")              # Search across OneDrive
od.download("/Documents/f.docx") # → bytes
od.upload("/Documents/new.docx", data_bytes)  # Upload (<4MB simple, >4MB session)
od.move("/f.docx", "/Archive")   # Move file
od.copy("/f.docx", "/Backup")    # Copy file
od.rename("/old.docx", "new.docx")
od.delete("/unwanted.docx")
od.create_folder("/Projects", "Q3")
```

### 3. Word (.docx)

```python
from onedrive import OneDrive
od = OneDrive()

# Read
text = od.docx_read("/Documents/report.docx", mode="plain")       # Plain text
text = od.docx_read("/Documents/report.docx", mode="structured")  # JSON with styles

# Create
od.docx_create("/Documents/new.docx", operations=[
    {"method": "add_heading", "args": ["Title", 1]},
    {"method": "add_paragraph", "args": ["Body text"]},
    {"method": "add_table", "kwargs": {"rows": 3, "cols": 2, "data": [["A","B"],["C","D"]], "style": "Table Grid"}},
])

# Edit (download → modify → upload)
od.docx_edit("/Documents/report.docx", operations=[
    {"method": "replace_text", "args": ["old text", "new text"]},
    {"method": "add_paragraph", "args": ["Appended paragraph"]},
    {"method": "add_heading", "args": ["Section 2", 2]},
    {"method": "add_page_break"},
])
```

**Word operations:** `add_paragraph`, `add_heading`, `add_table`, `add_page_break`, `add_picture`, `remove_paragraph`, `replace_text`. Paragraph-level ops use `paragraph_index` in kwargs. Table ops use `table_index`.

### 4. PowerPoint (.pptx)

```python
from onedrive import OneDrive
od = OneDrive()

# Read
text = od.pptx_read("/Documents/deck.pptx", mode="plain")
text = od.pptx_read("/Documents/deck.pptx", mode="structured")

# Create
od.pptx_create("/Documents/new.pptx", operations=[
    {"method": "add_slide", "kwargs": {"layout": "Title Slide", "title": "My Deck"}},
    {"method": "add_slide", "kwargs": {"layout": "Title and Content", "title": "Agenda", "body": "Item 1\nItem 2"}},
])

# Edit
od.pptx_edit("/Documents/deck.pptx", operations=[
    {"method": "add_textbox", "kwargs": {"slide_index": 0, "text": "Note", "left": 1, "top": 4}},
    {"method": "replace_text", "args": ["Draft", "Final"]},
    {"method": "add_notes", "kwargs": {"slide_index": 0, "text": "Speaker notes here"}},
    {"method": "add_table", "kwargs": {"slide_index": 1, "rows": 3, "cols": 3, "data": [["X","Y","Z"]]}},
    {"method": "add_picture", "kwargs": {"slide_index": 0, "path": "/tmp/chart.png", "left": 1, "top": 2}},
])
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Astrojoin/openclaw-office](https://github.com/Astrojoin/openclaw-office) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
