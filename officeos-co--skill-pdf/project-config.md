---
trigger: always_on
description: Create, read, and manipulate PDF documents via a file-proxy service. Supports creating PDFs from scratch, extracting text, merging and splitting files, adding pages and text, rotating pages, and managing page structure.
---

# PDF

Create, read, and manipulate PDF documents via a file-proxy service. Supports creating PDFs from scratch, extracting text, merging and splitting files, adding pages and text, rotating pages, and managing page structure.

All commands go through `skill_exec` using CLI-style syntax.
Use `--help` at any level to discover actions and arguments.

## Documents

### Create PDF

```
pdf create --filename "report.pdf"
```

| Argument   | Type   | Required | Description                         |
| ---------- | ------ | -------- | ----------------------------------- |
| `filename` | string | yes      | Output filename (e.g. report.pdf)   |

Returns: `file_id`, `filename`, `created_at`.

### Open PDF

```
pdf open --file_id "abc123"
```

| Argument  | Type   | Required | Description               |
| --------- | ------ | -------- | ------------------------- |
| `file_id` | string | yes      | ID of a previously stored file |

Returns: `file_id`, `filename`, `page_count`, `title`, `author`, `creator`, `producer`, `creation_date`, `modification_date`, `encrypted`.

### Get info

```
pdf get_info --file_id "abc123"
```

| Argument  | Type   | Required | Description |
| --------- | ------ | -------- | ----------- |
| `file_id` | string | yes      | File ID     |

Returns: `file_id`, `page_count`, `title`, `author`, `subject`, `keywords`, `creator`, `producer`, `creation_date`, `modification_date`, `page_sizes`, `encrypted`.

### Save PDF

```
pdf save --file_id "abc123"
```

| Argument  | Type   | Required | Description     |
| --------- | ------ | -------- | --------------- |
| `file_id` | string | yes      | File ID to save |

Returns: `file_id`, `filename`, `size_bytes`, `saved_at`.

## Content

### Extract text

```
pdf extract_text --file_id "abc123" --pages "1-5,8,10-12"
```

| Argument  | Type   | Required | Description                                          |
| --------- | ------ | -------- | ---------------------------------------------------- |
| `file_id` | string | yes      | File ID                                              |
| `pages`   | string | no       | Page range expression: `1-3,5,7-9` (1-indexed, omit for all) |

Returns: `text` (full extracted text), `pages` (array of `{page, text}`).

### Add page

```
pdf add_page --file_id "abc123" --width 595 --height 842 --position 2
```

| Argument   | Type   | Required | Default | Description                                                 |
| ---------- | ------ | -------- | ------- | ----------------------------------------------------------- |
| `file_id`  | string | yes      |         | File ID                                                     |
| `width`    | int    | no       | 595     | Page width in points (595 = A4)                             |
| `height`   | int    | no       | 842     | Page height in points (842 = A4)                            |
| `size`     | string | no       |         | Named size: `A4`, `A3`, `Letter`, `Legal` (overrides width/height) |
| `position` | int    | no       |         | Insert at page index (0-based, omit to append)              |

Returns: `page_index`, `page_count`, `file_id`.

### Add text

```
pdf add_text --file_id "abc123" --page_index 0 --text "Confidential" --x 50 --y 750 --font_size 12 --color "#FF0000"
```

| Argument      | Type   | Required | Default   | Description                               |
| ------------- | ------ | -------- | --------- | ----------------------------------------- |
| `file_id`     | string | yes      |           | File ID                                   |
| `page_index`  | int    | yes      |           | Zero-based page index                     |
| `text`        | string | yes      |           | Text to add                               |
| `x`           | int    | no       | 50        | X position in points from left            |
| `y`           | int    | no       | 750       | Y position in points from bottom          |
| `font_size`   | int    | no       | 12        | Font size in points                       |
| `font`        | string | no       | `Helvetica` | Font family: `Helvetica`, `Times-Roman`, `Courier` |
| `color`       | string | no       | `#000000` | Hex text color                            |
| `bold`        | boolean| no       | false     | Bold text                                 |
| `italic`      | boolean| no       | false     | Italic text                               |
| `opacity`     | number | no       | 1.0       | Opacity (0.0–1.0)                         |

Returns: `text_element_id`, `page_index`, `file_id`.

### Add image to page

```
pdf add_image --file_id "abc123" --page_index 0 --image_url "https://example.com/logo.png" --x 50 --y 700 --width 150
```

| Argument      | Type   | Required | Description                          |
| ------------- | ------ | -------- | ------------------------------------ |
| `file_id`     | string | yes      | File ID                              |
| `page_index`  | int    | yes      | Zero-based page index                |
| `image_url`   | string | yes      | URL or base64 data URI               |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [officeos-co/skill-pdf](https://github.com/officeos-co/skill-pdf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
