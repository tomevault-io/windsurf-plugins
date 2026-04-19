---
trigger: always_on
description: ﻿# Changelog - Subtitle Draft GMTPC
---

﻿# Changelog - Subtitle Draft GMTPC

Tất cả thay đổi đáng chú ý của project sẽ được ghi lại trong file này.

**⚠️ Lưu ý:** Thứ tự entries từ TRÊN xuống DƯỚI = từ MỚI đến CŨ
- Entry đầu tiên (trên cùng, ngay sau header) = MỚI nhất
- Entry cuối cùng (dưới cùng) = CŨ nhất
- LUÔN chèn entry mới lên ĐẦU file (ngay sau phần header này)

---
## [2026-04-11 04:30:00 PM Saturday]

### Added - Tab Text to Subtitle (✂️ Text to Subtitle)
- **Tab mới**: Tự động chia văn bản dài thành phụ đề ASS format
- **Panel 1**: Input văn bản gốc (Full Text Input)
- **Panel 2**: Settings + Output ASS
- **Max Characters**: Giới hạn ký tự tối đa mỗi subtitle (default: 500), tự tìm dấu chấm câu gần nhất để cắt trọn vẹn
- **CPS (Characters Per Second)**: Tính toán duration mỗi dòng dựa trên CPS (default: 17.0)
- **Punctuation mode**: Count all characters / Ignore punctuation - liên kết với CPS calculation
- **Gap (ms)**: Khoảng cách thời gian giữa các dòng phụ đề (default: 200ms)
- **Thuật toán split**: Ưu tiên cắt tại dấu câu (. ! ? 。) để câu trọn vẹn, fallback khoảng trắng, fallback cứng tại giới hạn
- **Auto-convert**: Debounce 300ms sau khi ngừng typing
- **Stats**: Hiển thị số segments, max chars/segment, tổng chars, CPS trung bình, tổng thời lượng
- **Search support**: Ctrl+F search trong panel input/output
- **Settings persistence**: Lưu MaxChars, CPS, Gap vào Properties.Settings
- **SearchManager**: Thêm `_searchTextToSub` cho tab mới

---
## [2026-04-09 02:00:00 PM Thursday]

### Fixed - Word Split Rules Search Reapply
- **Search tự động reaply** khi rules thay đổi (TextChanged event)
- **Search hoạt động sau Load/Reset**: Thêm `ReapplySearchAfterLoad()`
- Load file .txt xong → tự động tìm lại với search text hiện tại
- Reset rules → tự động tìm lại
- LoadKaraokeEngSplitRules: Set `_isKaraokeEngUpdating = true` để tránh trigger TextChanged loop

---
## [2026-04-09 01:45:00 PM Thursday]

### Fixed - Word Split Rules Search Highlight
- **Highlight hoạt động**: Focus TextBox rules → Select text → Scroll → Restore focus về search box sau 100ms
- **Toast notification**: Hiện số kết quả tìm được (vd: "🔍 3 kết quả cho 'white'")
- **Không tìm thấy**: Hiện "❌ Không tìm thấy '...'"
- Search giờ hoạt động đúng và có highlight rõ ràng

---
## [2026-04-09 01:30:00 PM Thursday]

### Fixed - Word Split Rules Search Issues
- **Không còn jump cursor**: Bỏ `Focus()` trong `SelectRuleSearchMatch()`
- Chỉ `Select()` text để highlight, không focus TextBox rules
- Cursor không bị nhảy lung tung khi search
- Set `_rulesSearchIndex = 0` khi tìm thấy kết quả đầu tiên

---
## [2026-04-09 01:15:00 PM Thursday]

### Added - Word Split Rules Search
- Thêm ô tìm kiếm trong panel Word Split Rules
- TextBox search + 2 nút ◀ Prev / ▶ Next
- **Auto-search**: Tìm ngay khi nhập liệu
- **Enter**: Tìm kết quả tiếp theo (wrap-around)
- **Escape**: Xóa search text và focus về TextBox rules
- Case-insensitive search
- Auto-select và scroll đến kết quả

### Changed - MainWindow.xaml
- Thêm RowDefinition cho search row trong Word Split Rules panel
- Grid chứa: TxtKaraokeEngRulesSearch + BtnSearchPrev + BtnSearchNext

### Changed - MainWindowTabKaraokeEnglish.cs
- Thêm search fields: `_rulesSearchIndex`, `_rulesSearchText`, `_rulesSearchPositions`
- `TxtKaraokeEngRulesSearch_TextChanged`: Auto-search khi nhập
- `TxtKaraokeEngRulesSearch_KeyDown`: Enter/Escape handling
- `BtnKaraokeEngRulesSearchPrev/Next_Click`: Navigation
- `FindAllRuleSearchMatches()`: Tìm tất cả vị trí
- `SelectRuleSearchMatch()`: Select và scroll đến match
- `GetLineFromPosition()`: Tính số dòng từ character position
- Thêm `using System.Windows.Input;`

### Build Result
- **Build**: SUCCESS ✅ (0 errors, 2 warnings - unused fields)

---
## [2026-04-09 01:00:00 PM Thursday]

### Fixed - Word Split Rules Case Sensitivity
- **Input case được bảo toàn** trong output, bất kể rule viết thế nào
- VD: Input="HOWLING", rule="how/ling" → Output="HOW/LING" ✅
- VD: Input="Howling", rule="how/ling" → Output="How/ling" ✅
- VD: Input="howling", rule="HOW/LING" → Output="how/ling" ✅
- Thêm hàm `AdjustSyllableCase()`: Tự động adjust syllables theo input case
- Hỗ trợ: ALL CAPS, lowercase, Title Case
- Thêm `using System.Linq;` vào KaraokeVietnameseService.cs

---
## [2026-04-09 12:45:00 PM Thursday]

### Fixed - Context Menu Placement
- Sửa vị trí context menu Load xuất hiện ngay dưới nút Load
- Thêm HorizontalOffset và VerticalOffset = 0
- Không còn hiện xa tít mù khơi nữa 😄

---
## [2026-04-09 12:30:00 PM Thursday]

### Fixed - Search Behavior
- Bỏ real-time search khi TextChanged
- Enter lần đầu: Tìm từ kết quả đầu tiên
- Enter nhiều lần: Tiếp tục tìm kết quả tiếp theo, không bị reset
- Tự động reset search khi text thay đổi
- F3 vẫn hoạt động để tìm kết quả tiếp theo

### Added - Word Split Rules Buttons (Karaoke English)
- Thêm 3 nút trong panel Word Split Rules: 🔄 Reset, 💾 Save, 📂 Load
- **Reset**: Load lại rules mặc định từ thư mục app
- **Save**: Browse chọn nơi lưu file .txt
- **Load**: Context menu với 2 tùy chọn:
  - 🔄 Load Default: Từ thư mục app
  - 📂 Load from File: Browse chọn file .txt
- Toast notification cho mỗi thao tác

### Changed - MainWindow.xaml
- Thêm RowDefinition cho buttons row trong panel Word Split Rules
- WrapPanel chứa 3 nút Reset/Save/Load

### Changed - MainWindowTabKaraokeEnglish.cs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ghostminhtoan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
