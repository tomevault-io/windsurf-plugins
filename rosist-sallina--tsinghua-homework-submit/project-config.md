---
trigger: always_on
description: Correctly handle Tsinghua Learn homework workflows: locate the assignment, download the prompt, read the instructions, submit the right file, and verify the result.
---


# Tsinghua Homework Submit Skill

Use this skill for the full Tsinghua Learn homework workflow, not just uploading a file.

Goals:

1. Find the correct course and assignment page.
2. Download the assignment prompt or attached files.
3. Read the assignment instructions and due date.
4. Submit the correct local file reliably.
5. Avoid cross-course mistakes like uploading the wrong homework to the wrong class.

## When to use

Use this skill when:

- The user is already logged into Tsinghua Learn in Chrome.
- The course page, assignment page, or submission page is already open in the real browser, or can be reached from an existing tab.
- You need the complete workflow: download prompt + read instructions + submit solution.

## Core rules

1. Confirm the course first, then the assignment title, then the local file path, and only then submit.
2. Never guess the answer file only from “most recently created PDF”.
3. If the page says “upload failed” or “unsupported file type”, do not trust it immediately; first determine whether:
   - it is a frontend-only error, or
   - the server actually rejected the file.
4. Before submitting, always do a triple check:
   - current course name
   - current assignment title
   - absolute path of the local file to submit

## Recommended workflow

### Step 1: Find the Tsinghua Learn tab

```bash
alma browser tabs
```

Look for tabs like:

- `https://learn.tsinghua.edu.cn/...`
- titles that are course names, such as `Operating Systems`

If you are not already on an assignment page, navigate from the course page.

### Step 2: Confirm what page you are on

```bash
alma browser read <tabId>
alma browser read-dom <tabId>
```

Verify all of the following:

- whether this is the course homepage, assignment list page, assignment detail page, or submission page
- current course name
- current assignment title
- due date
- whether the page shows things like assignment attachments, answer attachments, or a submit button

## Locating a specific assignment from the assignment list

Assignment list pages are usually like:

- `/f/wlxt/kczy/zy/student/beforePageList?...`

You can extract the table with:

```bash
alma browser eval <tabId> "(() => {
  const rows=[...document.querySelectorAll('table tr')]
    .map(tr => [...tr.querySelectorAll('td')].map(td => td.innerText.trim().replace(/\\n+/g,' | ')))
    .filter(r => r.length);
  return JSON.stringify(rows, null, 2);
})()"
```

If you want the detail link for each row:

```bash
alma browser eval <tabId> "(() => {
  const rows=[...document.querySelectorAll('table tr')].map(tr => [...tr.querySelectorAll('td')]);
  const items=rows.filter(tds => tds.length >= 6).map(tds => ({
    title: tds[1]?.innerText.trim(),
    href: tds[1]?.querySelector('a')?.href || ''
  }));
  return JSON.stringify(items, null, 2);
})()"
```

Then open the target assignment detail page.

### Step 3: Read the assignment details and instructions

Assignment detail pages are usually like:

- `/f/wlxt/kczy/zy/student/viewZy?...`

Read them with:

```bash
alma browser read <tabId>
alma browser read-dom <tabId>
```

If `alma browser read` does not extract the content cleanly, fall back to:

```bash
alma browser eval <tabId> "document.body.innerText"
```

Extract at least:

- course name
- assignment title
- assignment instructions shown on the page
- due date
- names of attached prompt files

### Step 4: Download the assignment prompt correctly

Do not guess download URLs with curl first.

Instead, first inspect the DOM on the assignment detail page and collect the attachment links:

```bash
alma browser eval <tabId> "(() => JSON.stringify(
  [...document.querySelectorAll('a')]
    .map(a => ({text:(a.innerText||'').trim(), href:a.href}))
    .filter(x => x.text || x.href.includes('downloadFile')),
  null, 2
))()"
```

Common cases:

- the attachment filename itself is a link
- there is a separate `Download` link nearby
- the real download URL may include an `_csrf` parameter

After you have the link, prefer opening it in the browser so the site can use the existing login session:

```bash
alma browser open "<attachment-url>"
```

If the site says the file cannot be previewed and will be downloaded automatically, continue and confirm.

Then find the newly downloaded local file:

```bash
find ~/Downloads -maxdepth 1 -type f -mmin -5 -print0 | xargs -0 ls -lt | head
```

### Step 5: Read the downloaded prompt file

Once the file is on disk, read the local file itself instead of guessing from the preview page.

Examples:

```bash
Read /Users/.../Homework4.md
Read /Users/.../HW4.txt
```

If it is a PDF, prefer checking whether there is also a markdown, doc, or txt version first. If not, use an appropriate PDF-reading path afterward.

## Submitting homework: two strategies

### Strategy A: Prefer the page’s native submission flow

First open the submission page:

- `/f/wlxt/kczy/zy/student/tijiao?...`

If the detail page has a submit button, use that first. Otherwise, inspect page scripts or functions such as `goBtnF()` to find the target submission URL.

Once on the submission page, confirm these important elements:

- form: `#form_sn1`
- file input: `#fileupload`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rosist-Sallina/tsinghua-homework-submit](https://github.com/Rosist-Sallina/tsinghua-homework-submit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
