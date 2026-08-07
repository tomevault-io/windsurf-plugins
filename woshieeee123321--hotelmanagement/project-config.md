---
trigger: always_on
description: 全栈酒店管理系统，当前主线业务按 `房态 / 客单 / 现付账 / 报表 / 设置` 五大方向收敛。
---

# AGENTS.md — 酒店管理系统 (Hotel PMS) 编码规范

## 项目概况

全栈酒店管理系统，当前主线业务按 `房态 / 客单 / 现付账 / 报表 / 设置` 五大方向收敛。

- 房态：实时房态、远期房态总览、房间管理，以及后续的批量操作、营业日提示、续住/换房等接待增强
- 客单：订单全生命周期、预订修改、挂账单、联房/团队单、No Show 等
- 现付账：独立于房帐的即时消费收银，不与当前房帐财务客单混用
- 报表：前台交班、付款明细、消费明细、营收与经营分析报表
- 设置：酒店基础信息、班次、零售商品、用户账号、钟点房配置

说明：
- 当前仓库中的 `Members.jsx`、`More.jsx` 仅为占位页，不再作为独立产品方向扩写
- 前端继续沿用现有霓虹赛博视觉风格，不做风格重置

```text
HotelManagement/
├── pms-backend/        # Python FastAPI 后端
├── pms-frontend/       # React + Vite 前端
├── reference/          # 逆向需求与参考资料
├── jiagou.md           # 架构设计文档
└── AGENTS.md           # 本文件
```

## 技术栈（锁定版本，不要随意升级）

| 层 | 技术 | 备注 |
|---|---|---|
| 后端框架 | FastAPI 0.115+ | ASGI，async 优先 |
| ORM | SQLAlchemy 2.0+ | 声明式模型 |
| 数据校验 | Pydantic v2 | BaseModel 做 schema |
| 数据库 | SQLite (开发) → PostgreSQL (生产) | 通过 DATABASE_URL 切换 |
| 前端框架 | React 19 | 函数组件 + Hooks |
| 构建工具 | Vite 7 | ES modules |
| UI 组件库 | Material-UI (MUI) 7 | 唯一 UI 库，不引入其他 |
| 状态管理 | Zustand 5 | 轻量 store |
| 路由 | React Router DOM 7 | 嵌套路由 |
| HTTP 客户端 | Axios | 统一走 api.js |
| CSS 方案 | Emotion (MUI sx prop) | 不使用 CSS Modules / Tailwind |

## 启动命令

```bash
# 后端
cd pms-backend && pip install -r requirements.txt && uvicorn main:app --reload

# 前端
cd pms-frontend && npm install && npm run dev
```

---

## 一、通用规范

### 语言
- 前端：JavaScript (JSX)，不使用 TypeScript
- 后端：Python 3.12+
- UI 文案、注释、变量命名说明均使用中文（变量名本身用英文）
- API 错误消息使用中文（如 `"房间不存在"`）

### 命名约定
| 场景 | 风格 | 示例 |
|---|---|---|
| Python 函数/变量 | snake_case | `check_in`, `room_id` |
| Python 类 | PascalCase | `CheckInRequest` |
| SQLAlchemy 列名 | camelCase | `roomNumber`, `checkInTime` |
| JS 变量/函数 | camelCase | `loadRooms`, `guestName` |
| React 组件 | PascalCase | `RoomStatus`, `MainLayout` |
| 文件名 (前端组件) | PascalCase.jsx | `RoomStatus.jsx` |
| 文件名 (前端工具) | camelCase.js | `api.js`, `store.js` |
| 文件名 (后端) | snake_case.py | `main.py`, `models.py` |
| API 路径 | kebab-case 或小写 | `/api/rooms`, `/api/night-audit` |
| 数据库表名 | 复数小写 | `rooms`, `guests`, `orders` |

### 状态枚举（字符串常量，全大写）
- 房间状态：当前已使用 `VC / VD / OC / OD / OOO / RES`，后续新增锁房语义时需单独设计，不与维修房混用
- 订单状态：`ACTIVE` / `COMPLETED` / `CANCELLED`
- 支付状态：`PENDING` / `PAID` / `REFUNDED`

### 文档更新原则
- `AGENTS.md` 中的“模块开发指南”只写尚未完成或尚未业务闭环的未来蓝图
- 已有真实前后端实现的能力，不要重复写成后续任务
- 新需求的来源以 `reference/missionmap.md` 为准，`reference/devnote.md` 用于约束去重策略与风格延续

---

## 二、后端规范 (pms-backend/)

### 当前基础目录
```text
pms-backend/
├── main.py
├── database.py
├── models.py
├── schemas/
│   ├── room.py
│   ├── order.py
│   └── finance.py
├── routers/
│   ├── rooms.py
│   ├── orders.py
│   └── finance.py
├── services/
│   ├── room_service.py
│   ├── reservation_service.py
│   ├── order_service.py
│   └── finance_service.py
├── tests/
└── requirements.txt
```

### 未来扩展目录目标
在保持现有结构不破坏的前提下，未来新增模块按以下方向扩展：

```text
routers/
├── auth.py
├── reports.py
├── settings.py
├── cashier.py
├── night_audit.py
├── channels.py
└── hardware.py

schemas/
├── auth.py
├── report.py
├── setting.py
├── cashier.py
├── night_audit.py
├── channel.py
└── hardware.py

services/
├── auth_service.py
├── report_service.py
├── setting_service.py
├── cashier_service.py
├── night_audit_service.py
├── channel_service.py
└── hardware_service.py
```

### API 设计
- 所有端点以 `/api/` 开头
- RESTful 风格：GET 查询、POST 创建、PUT/PATCH 更新、DELETE 删除
- 路由函数使用 `Depends(get_db)` 注入数据库 Session
- 请求体用 Pydantic BaseModel 校验，不要在路由函数里手动校验字段
- 响应统一返回 dict 或 Pydantic model，不要直接返回 ORM 对象
- 错误使用 `HTTPException`，status_code 遵循 HTTP 语义，detail 用中文
- 后续新增模块推荐前缀：
  - `/api/auth`
  - `/api/night-audit`
  - `/api/cashier`
  - `/api/reports`
  - `/api/settings`
  - `/api/channels`
  - `/api/hardware`

### 数据库 & 模型
- 模型定义在 `models.py`，一个文件管理所有表
- 外键关系必须定义 `relationship` 双向绑定
- 主键统一用自增 Integer `id`
- 时间字段用 `DateTime`，默认值用 `datetime.now`（注意不是 `datetime.now()`）
- 字符串枚举直接用 String 列 + 注释说明可选值，不用 Python Enum
- 新增表时在 `models.py` 底部追加，保持导入顺序一致
- 已有局部字段不代表业务已闭环，新增能力仍需补足专属模型，不能只靠备注字段硬撑

### 业务逻辑
- 简单 CRUD 可以直接写在路由里
- 涉及多表操作、事务、外部调用的逻辑抽到 `services/` 层
- 数据库事务：一个请求一个 Session，service 层不要自己创建 Session
- 不要在路由层 catch 通用异常来返回 200，让 FastAPI 的异常处理机制工作
- 夜审、报表、渠道同步、硬件对接、现付账、认证鉴权逻辑必须进 service 层
- 在增加和修改功能时，前端与后端要对齐，也就是说永远不要出现前端仅供演示的情况而没有实际做后端。如果需要演示，请在做出相应后端后进行实际业务级别的演示。
- 在完成TASKS.md中的任务时，可以在根目录中查看前置任务的完成与测试情况，所有完成的任务都有一个对应的T0?Test.md文件，?替换成第几个任务，里面详细写了任务完成情况以及测试结果

---

## 三、前端规范 (pms-frontend/)

### 当前目录结构
```text
pms-frontend/src/
├── main.jsx
├── App.jsx
├── api.js
├── store.js
├── theme.js
├── layouts/
│   └── MainLayout.jsx
├── pages/
│   ├── RoomStatus.jsx
│   ├── FutureRooms.jsx
│   ├── Orders.jsx
│   ├── Payments.jsx
│   ├── Reports.jsx
│   ├── RoomManage.jsx
│   ├── Members.jsx      # 当前占位，不再扩写主线需求
│   └── More.jsx         # 当前占位，不再扩写主线需求
├── components/
│   ├── ReservationOrderDialog.jsx
│   └── WaveBackground.jsx
└── assets/
```

### 未来页面扩展方向
后续新增正式页面时，优先补齐以下产品主线页面：

```text
pages/
├── Login.jsx
├── Cashier.jsx
├── Reports.jsx      # 从占位页升级为正式报表中心
└── Settings.jsx
```

### 组件编写
- 只用函数组件 + Hooks，不用 class 组件
- 组件用 `export default function ComponentName()` 导出
- 一个文件一个组件，文件名 = 组件名
- 页面组件放 `pages/`，可复用组件放 `components/`
- props 不需要 PropTypes（项目不用 TypeScript 也不用 PropTypes）
- 组件内部状态用 `useState`，跨组件共享用 Zustand store
- 副作用用 `useEffect`，依赖数组必须写完整

### 样式规范
- 统一使用 MUI 的 `sx` prop，不写外部 CSS 文件
- 颜色、间距、圆角等从 `theme.js` 取值，不要硬编码新的设计 token
- 保持现有霓虹赛博视觉语言，后续新页面只做同风格扩展，不做整站换肤
- 设计系统色板（严格遵守，不要引入新颜色）：

| 用途 | 色值 |
|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [woshieeee123321/HotelManagement](https://github.com/woshieeee123321/HotelManagement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-01 -->
