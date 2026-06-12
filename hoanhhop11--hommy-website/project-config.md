---
trigger: always_on
description: - **Ngôn ngữ phản hồi**: AI agent phải luôn phản hồi bằng tiếng Việt (vi-VN) trong mọi tương tác, trừ khi người dùng yêu cầu ngôn ngữ khác cho một trường hợp cụ thể.
---

## 🗣️ 1. Communication Rules

- **Ngôn ngữ phản hồi**: AI agent phải luôn phản hồi bằng tiếng Việt (vi-VN) trong mọi tương tác, trừ khi người dùng yêu cầu ngôn ngữ khác cho một trường hợp cụ thể.

## 🏗️ 2. Code Organization Rules

### Models - Tách theo tính năng

**Nguyên tắc:**
- Mỗi model chỉ quản lý 1 domain entity chính
- File > 500 dòng → **BẮT BUỘC** tách thành nhiều files
- Methods không liên quan đến domain hiện tại → tách file mới

**Ví dụ tách ChuDuAnModel.js (1648 dòng):**

❌ **SAI**: Gom tất cả trong `ChuDuAnModel.js`
```javascript
class ChuDuAnModel {
  // Tin đăng methods
  static async layDanhSachTinDang() {}
  static async taoTinDang() {}
  
  // Dự án methods
  static async layDanhSachDuAn() {}
  static async taoDuAn() {}
  
  // Cuộc hẹn methods
  static async layDanhSachCuocHen() {}
  
  // Báo cáo methods
  static async layBaoCaoHieuSuat() {}
}
```

✅ **ĐÚNG**: Tách thành nhiều files
```
server/models/
├── TinDangModel.js           # Chỉ methods về Tin đăng
├── DuAnModel.js             # Chỉ methods về Dự án
├── CuocHenModel.js          # Chỉ methods về Cuộc hẹn
├── BaoCaoHieuSuatModel.js   # Chỉ methods về Báo cáo
└── ChinhSachCocModel.js     # Chỉ methods về Chính sách cọc (đã có)
```

**Cấu trúc model chuẩn:**
```javascript
/**
 * Model cho [Tên Domain]
 * Quản lý [Mô tả domain]
 */
const db = require('../config/db');

/**
 * @typedef {Object} EntityName
 * @property {number} EntityID
 * @property {string} PropertyName
 */

class DomainModel {
  /**
   * [Mô tả method]
   * @param {number} id ID của entity
   * @param {Object} filters Bộ lọc
   * @returns {Promise<Array>}
   */
  static async methodName(id, filters = {}) {
    try {
      // Implementation
    } catch (error) {
      throw new Error(`Lỗi [mô tả]: ${error.message}`);
    }
  }
}

module.exports = DomainModel;
```

### Controllers - Tách theo tính năng

**Nguyên tắc tương tự Models:**
- Mỗi controller chỉ xử lý 1 domain entity
- File > 500 dòng → **BẮT BUỘC** tách
- Tách theo tính năng, không theo technical layer

**Ví dụ:**
```
server/controllers/
├── TinDangController.js      # Chỉ xử lý Tin đăng
├── DuAnController.js         # Chỉ xử lý Dự án
├── CuocHenController.js      # Chỉ xử lý Cuộc hẹn
└── BaoCaoHieuSuatController.js # Chỉ xử lý Báo cáo
```

### Routes - Nhóm theo domain

**Cấu trúc:**
```
server/routes/
├── tinDangRoutes.js          # Routes cho Tin đăng
├── duAnRoutes.js             # Routes cho Dự án
├── cuocHenRoutes.js          # Routes cho Cuộc hẹn
└── baoCaoRoutes.js           # Routes cho Báo cáo
```

**Pattern chuẩn:**
```javascript
const express = require('express');
const router = express.Router();
const DomainController = require('../controllers/DomainController');
const { authenticate, authorize } = require('../middleware/auth');

router.get('/', authenticate, DomainController.list);
router.get('/:id', authenticate, DomainController.getById);
router.post('/', authenticate, authorize(['role']), DomainController.create);

module.exports = router;
```

### Components - Mỗi component có folder riêng

**Cấu trúc:**
```
client/src/components/
├── ComponentName/
│   ├── ComponentName.jsx     # Component chính
│   ├── ComponentName.css     # Styles BEM
│   └── index.js              # Barrel export (optional)
```

**Pattern:**
```jsx
// ComponentName.jsx
import './ComponentName.css';

export default function ComponentName({ prop1, prop2 }) {
  return (
    <div className="component-name">
      <div className="component-name__header">...</div>
      <div className="component-name__body">...</div>
    </div>
  );
}
```

---

## 🎨 3. CSS Rules - BEM Naming Convention

### BEM Methodology - BẮT BUỘC

**Cấu trúc:** `block__element--modifier`

- **Block**: Component chính (ví dụ: `modal-duan`, `button`)
- **Element**: Phần tử con (ví dụ: `modal-duan__header`, `button__icon`)
- **Modifier**: Trạng thái/biến thể (ví dụ: `modal-duan--open`, `button--primary`)

### Ví dụ đúng/sai

**✅ ĐÚNG:**
```css
/* Block */
.modal-duan {}

/* Element */
.modal-duan__overlay {}
.modal-duan__container {}
.modal-duan__header {}
.modal-duan__title {}
.modal-duan__close {}

/* Modifier */
.modal-duan--open {}
.modal-duan--large {}
.modal-duan__close--disabled {}
```

**❌ SAI:**
```css
/* Không dùng nested selectors phức tạp */
.modal-duan .overlay {}  /* ❌ */
.modal-duan > .header {} /* ❌ */

/* Không dùng camelCase */
.modalDuan {}  /* ❌ */
.tieuDe {}     /* ❌ */

/* Không dùng kebab-case không BEM */
.modal-duan-overlay {}  /* ❌ Phải là modal-duan__overlay */
.header {}              /* ❌ Thiếu block name */
```

### Quy tắc BEM

1. **Block name**: Dùng tên component (tiếng Việt không dấu, lowercase, hyphen-separated)
   ```css
   /* Component: ModalTaoDuAn.jsx */
   .modal-tao-duan {}
   
   /* Component: QuanLyTinDang.jsx */
   .quan-ly-tin-dang {}
   ```

2. **Element**: Dùng `__` (double underscore)
   ```css
   .modal-tao-duan__header {}
   .modal-tao-duan__body {}
   .modal-tao-duan__footer {}
   ```

3. **Modifier**: Dùng `--` (double hyphen)
   ```css
   .modal-tao-duan--open {}
   .modal-tao-duan--disabled {}
   .modal-tao-duan__button--primary {}
   ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HoanhHop11/Hommy-Website](https://github.com/HoanhHop11/Hommy-Website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
