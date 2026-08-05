---
trigger: always_on
description: PVE 集群扫描与管理平台 — Django 5 + Vue 3 全栈项目。
---

# pve-cluster-scan

PVE 集群扫描与管理平台 — Django 5 + Vue 3 全栈项目。

## 项目架构

```
pve-cluster-scan/
├── config/                 # Django 项目配置
│   ├── settings.py         #   - DRF / JWT / CORS / Vite 端口配置
│   ├── urls.py             #   - auth 路由 + catch-all → Vue SPA
│   └── asgi.py             #   - ASGI 入口（uvicorn）
├── apps/
│   ├── core/               # 通用工具（Vite 模板标签 / 上下文处理器）
│   │   ├── context_processors.py  # - vite_context（注入 vite_host/vite_port）
│   │   ├── templatetags/
│   │   │   └── vite_tags.py       # - vite_asset（生产模式 manifest 查找）
│   │   └── views.py
│   ├── accounts/           # 用户认证 & 操作日志 & AI 助手
│   │   ├── models.py       #   - User / PasswordResetCode / UserLog / UserLLMConfig / ChatConversation / ChatMessage
│   │   ├── serializers.py  #   - Login / Register / User / PasswordReset
│   │   ├── views.py        #   - 登录 / 注册 / 用户信息 / 密码重置 / 操作日志 / AI 聊天流式端点
│   │   ├── urls.py         #   - /api/auth/ 路由（含 logs/ + chat/ + llm-configs/ + system-prompts/）
│   │   ├── llm_service.py  #   - LangChain LLM 封装（build_llm / stream_chat）
│   │   ├── chat_context.py #   - PVE 数据上下文注入（关键词匹配 → 动态查询）
│   │   └── admin.py
│   ├── clusters/           # 集群管理（CRUD + Agent 列表）
│   │   ├── models.py       #   - Cluster（含 agent_token）
│   │   ├── serializers.py  #   - List / Create / Detail / AgentBrief
│   │   ├── views.py        #   - 集群 CRUD + Agent 查询
│   │   ├── urls.py         #   - /api/clusters/ 路由
│   │   ├── tests.py        #   - 10 个测试用例
│   │   └── admin.py
│   ├── agent_api/          # Agent 通信 & 多 Agent 管理
│   │   ├── models.py       #   - AgentInstance / ScanTask
│   │   ├── serializers.py  #   - Register / Heartbeat / ScanUpload / Tasks / Unregister / Version
│   │   ├── views.py        #   - 注册 / 心跳 / 扫描上传 / 任务下发 / 卸载 / 版本查询 / 安装脚本
│   │   ├── urls.py         #   - /api/agent/ 路由（7 个端点）
│   │   ├── install_script.py  # install.sh 模板生成
│   │   ├── tests.py        #   - 47 个测试用例（含 7 个过期数据清理测试）
│   │   └── admin.py
│   ├── dashboard/          # Dashboard 查询 API
│   │   ├── views.py        #   - stats / alerts / trends / nodes
│   │   ├── urls.py         #   - /api/dashboard/ 路由（4 个端点）
│   │   └── tests.py        #   - 30 个测试用例
│   └── scanner/            # 扫描数据 & 自动检测
│       ├── views.py        #   - 节点/VM/容器/存储/网络/Ceph/HA/SDN 列表+详情
│       ├── urls.py         #   - /api/scanner/ 路由（13 个端点）
│       └── models.py       #   - ClusterNode/VM/LXC/VMConfig/LXCConfig/Storage/NetworkInterface/CephStatus/ScanHistory/DetectionRule/DetectionResult/SDNZone/SDNVNet/SDNSubnet
├── frontend/               # Vue 3 + Vite 前端
│   ├── src/
│   │   ├── views/
│   │   │   ├── Home.vue                       # Landing Page
│   │   │   ├── auth/
│   │   │   │   ├── Login.vue                  # 登录（用户名/邮箱通用）
│   │   │   │   ├── Register.vue               # 注册（邮箱必填）
│   │   │   │   └── ForgotPassword.vue         # 找回密码（两步流程）
│   │   │   ├── dashboard/
│   │   │   │   ├── index.vue                  # 控制台主布局
│   │   │   │   ├── StatCards.vue              # 统计卡片（集群/节点/VM&容器/告警）
│   │   │   │   ├── AlertList.vue              # 最近告警列表（固定高度+滚动）
│   │   │   │   ├── TrendChart.vue             # 资源趋势 ECharts 折线图
│   │   │   │   ├── NodeTable.vue              # 节点详情表格
│   │   │   │   └── ChangePassword.vue         # 修改密码
│   │   │   ├── clusters/index.vue             # 集群管理（表格+CRUD+详情弹窗）
│   │   │   ├── nodes/index.vue                # 节点管理（表格+详情弹窗）
│   │   │   ├── vms/index.vue                  # 虚拟机（表格+详情弹窗）
│   │   │   ├── containers/index.vue           # 容器（表格+详情弹窗）
│   │   │   ├── sdn/index.vue                  # SDN 虚拟网络（Tab 切换：区域/VNet/子网）
│   │   │   ├── alerts/index.vue               # 告警中心（告警记录表格）
│   │   │   ├── services/index.vue             # 运维服务（HA 资源状态表格）
│   │   │   ├── settings/index.vue             # 用户信息（完整编辑界面）
│   │   │   ├── user-logs/index.vue            # 操作日志（分页表格+筛选）
│   │   │   └── user-notifications/index.vue   # 通知设置（空状态）
│   │   ├── components/
│   │   │   ├── AppSidebar.vue        # 侧边栏导航（含折叠图标居中 + 集群选择器）
│   │   │   └── AppHeader.vue         # 顶栏（含主题切换 + 退出登录）
│   │   ├── layouts/MainLayout.vue    # 后台主布局（侧边栏+顶栏+内容区）
│   │   │── router/index.ts           # 路由 + 守卫（所有后台页面在 /dashboard/ 下）
│   │   ├── stores/
│   │   │   ├── app.ts                # 全局状态（侧边栏折叠）
│   │   │   ├── auth.ts               # JWT 认证
│   │   │   ├── theme.ts              # 亮暗主题（默认暗色）
│   │   │   └── cluster.ts            # 集群列表 + 当前选中集群 (全局集群选择器)
│   │   ├── api/
│   │   │   ├── request.ts            # Axios 实例 + 拦截器
│   │   │   ├── auth.ts               # 登录/注册/密码重置/用户信息/操作日志 API
│   │   │   ├── clusters.ts           # 集群 CRUD + Agent 查询 API
│   │   │   ├── dashboard.ts          # Dashboard 统计/告警/趋势/节点 API
│   │   │   ├── nodes.ts              # 节点/详情 API
│   │   │   ├── ceph.ts               # Ceph 状态 API
│   │   │   ├── ha.ts                 # HA 资源 API
│   │   │   └── sdn.ts                # SDN 区域/VNet/子网 API
│   │   └── style.css                 # CSS 变量 / 亮暗色值
│   ├── package.json
│   └── vite.config.ts
├── agent/                    # Agent（单文件，零依赖）
│   └── agent.py               # PVE 数据采集 + 上报（纯 Python stdlib）
├── data-structure/           # PVE 数据结构分析文档
│   ├── README.md             # 分析说明与 License 声明
│   ├── database-models.md    # 数据库模型与 PVE 字段映射
│   ├── api-interfaces.md     # PVE API 接口清单
│   ├── field-mapping.md      # 字段对照表

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Vrolist/PCS](https://github.com/Vrolist/PCS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-03 -->
