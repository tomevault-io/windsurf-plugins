---
trigger: always_on
description: >
---


# Line-Locator

Hai script phối hợp để định vị chính xác trước khi đọc:

| Script | Phạm vi | Câu hỏi trả lời |
|---|---|---|
| `findtree.py` | Cả cây thư mục | *File nào* chứa pattern này? |
| `findtool.py` | Một file duy nhất | *Dòng nào* trong file này? |

**Workflow:**
```
findtree.py (shortlist files) → findtool.py (locate lines) → view_range (read)
```

Script nằm tại: `skills/line-locator/scripts/`

---

## Output contract

**Tất cả pattern dùng Python regex.**

**findtool — success (stdout):**
```json
{"matches": {"processOrder": [247, 312]}}
{"line": 248}
{"matched": true}
```

**findtree — success (stdout):**
```json
{"matched_files": ["src/order.js", "tests/order.test.js"]}
```

**Failure — cả hai tool (stderr, exit 1):**
```json
{"ok": false, "error": "No line matching pattern 'foo' was found after line 0."}
```

**Parse nhanh:**
```python
import json, subprocess
out = subprocess.check_output([...])
data = json.loads(out)

# findtool
lines   = data["matches"]["pat"]   # -mr  → list[int]
line    = data["line"]             # -n / -b / -c / -o → int
matched = data["matched"]          # -e   → bool

# findtree
files   = data["matched_files"]    # list[str] relative paths
```

---

## findtree — tìm file

```bash
python findtree.py --root ROOT --pattern PAT [options]
```

| Option | Mặc định | Mô tả |
|---|---|---|
| `--root` | *(required)* | Thư mục gốc |
| `--pattern` | *(required)* | Regex pattern |
| `--include GLOB...` | *(tất cả)* | Chỉ tìm file khớp glob |
| `--exclude GLOB...` | *(không)* | Bỏ qua file/thư mục khớp glob |
| `--ignore-case` | off | Case-insensitive |
| `--max-results N` | *(không giới hạn)* | Dừng sau N file |
| `--show` | `matched` | `matched` / `errors` / `both` / `summary` / `all` |
| `--max-file-size` | *(không)* | Bỏ qua file lớn hơn (VD: `64K`, `10M`) |
| `--no-default-dir-excludes` | off | Tắt auto-skip `.git`, `node_modules`, v.v. |
| `--text` | off | Output text thay vì JSON |

**Auto-excluded dirs:** `.git`, `.hg`, `node_modules`, `venv`, `.venv`, `__pycache__`, `dist`, `build`, `target`, `.mypy_cache`, `.pytest_cache`, `.tox`, `coverage`

```bash
python findtree.py --root ./src --pattern "processOrder"
python findtree.py --root ./src --pattern "class\s+\w+Service"
python findtree.py --root . --pattern "TODO" --include "*.ts" "*.js" --exclude "dist/**"
python findtree.py --root . --pattern "deprecated_api" --max-results 1
```

---

## findtool — tìm dòng

```bash
python findtool.py --file FILE [--ignore-case] [-s] FLAG... [--text]
```

| Flag | Args | Mô tả | `result` key |
|---|---|---|---|
| `-mr` | `PAT [PAT ...]` | Tất cả dòng khớp một hoặc nhiều pattern | `{"matches": {"pat": [1,5]}}` |
| `-n` | `PAT LINE` | Dòng đầu tiên khớp **sau** LINE (0 = từ đầu file) | `{"line": 45}` |
| `-b` | `PAT LINE` | Dòng cuối cùng khớp **trước** LINE (999999 = cuối file) | `{"line": 12}` |
| `-e` | `PAT` | File có chứa pattern không? | `{"matched": true}` |
| `-c` | `OPEN CLOSE LINE N` | Dòng đóng khớp với OPEN thứ N trên LINE (scan xuôi) | `{"line": 89}` |
| `-o` | `OPEN CLOSE LINE N` | Dòng mở khớp với CLOSE thứ N trên LINE (scan ngược) | `{"line": 34}` |

Flag `-s` (chỉ dùng với `-c`/`-o`): bỏ qua nội dung string và comment khi đếm depth.

```bash
python findtool.py --file app.js -mr "processOrder"
python findtool.py --file app.js -mr "processOrder" "cancelOrder" "getUser"
python findtool.py --file app.js -n "function\s+\w+" 0
python findtool.py --file app.js -b "^import\s" 999999
python findtool.py --file app.js -e "TODO"
python findtool.py --file app.js -c "\{" "\}" 248 1
python findtool.py --file app.js -c "\{" "\}" 248 1 -s
python findtool.py --file app.js -o "\{" "\}" 298 1
```

---

## Pair matching (-c / -o)

Cả hai flag đều nhận `OPEN CLOSE LINE N` theo cùng thứ tự:

```
-c  OPEN  CLOSE  LINE  N  →  anchor tại OPEN thứ N trên LINE, scan xuôi  → trả về dòng CLOSE
-o  OPEN  CLOSE  LINE  N  →  anchor tại CLOSE thứ N trên LINE, scan ngược → trả về dòng OPEN
```

**Algorithm:** depth-tracking. Mỗi dòng tiếp theo: `depth += count(OPEN) - count(CLOSE)`. Khi `depth == 0` → trả về dòng đó.

**Flag `-s`:** trước khi đếm, mask toàn bộ string literals và comments (`//`, `#`, `/* */`, `'...'`, `"..."`, backtick, triple-quote). Dùng khi file có nhiều delimiter trong string/comment.

### Pairs phổ biến theo ngôn ngữ

| Ngôn ngữ | Block mở | Block đóng | Ghi chú |
|---|---|---|---|
| JS/TS/Java/Go/Rust/C/C++/C# | `\{` | `\}` | Dùng `-s` nếu file có `{` trong string |
| Bất kỳ — argument list | `\(` | `\)` | Áp dụng mọi ngôn ngữ |
| Bất kỳ — array/list | `\[` | `\]` | Áp dụng mọi ngôn ngữ |
| HTML/XML | `<div\b[^>]*>` | `</div>` | Thay `div` bằng tên tag |
| Ruby | `\bdo\b` | `\bend\b` | Hoặc `\bdef\b` / `\bend\b` |
| Lua / Pascal-like | `\bdo\b` | `\bend\b` | |
| Shell (bash) | `\bthen\b\|\bdo\b` | `\bfi\b\|\bdone\b` | Phức tạp hơn |
| Python | *(không có bracket)* | *(xem note bên dưới)* | Dùng `-n` tìm def tiếp theo |

> **Python note:** Python dùng indentation thay vì `{}`. Để tìm cuối hàm, dùng `-n "^(def\|class)\s" FUNC_LINE` tìm `def`/`class` tiếp theo cùng cấp, hoặc đọc từ `FUNC_LINE` đến dòng trước đó. Với argument list nhiều dòng: `-c "\(" "\)" FUNC_LINE 1` vẫn hoạt động.

```bash
# JS/TS function
python findtool.py --file service.ts -c "\{" "\}" 248 1 -s
# → {"line": 298}

# HTML div

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unkluco/line-locator](https://github.com/unkluco/line-locator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
