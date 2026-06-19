---
trigger: always_on
description: >
---


# Boss直聘每日简历Brief工作流

配置文件路径：与本 skill.md 同目录的 `config.json`（参考 `config.example.json`）。

---

## 初始化（`boss init` — 首次使用）

当用户说"boss init"、"初始化"、"第一次配置"时，运行以下四个阶段。每个阶段完成后明确告知用户，再进入下一阶段。

---

### Phase 1：工具检查

依次检查：

```bash
boss --version 2>&1
```
```bash
lark-cli --version 2>&1
```
```bash
node --version 2>&1
```

- ✅ 三个工具均存在 → 进入 Phase 2
- ❌ `boss` 未找到 → 提示：`npm install -g @joohw/boss-cli`
- ❌ `lark-cli` 未找到 → 提示：`npm install -g @larksuite/cli`
- ❌ `node` 未找到 → 提示安装 Node.js（https://nodejs.org）

工具全部就绪后继续。

---

### Phase 2：登录授权

#### 2.1 boss CLI

```bash
boss list 2>&1 | head -3
```

- ✅ 返回候选人列表 → 已登录
- ❌ `未检测到 .menu-list` → 提示用户运行 `boss login`，在浏览器完成登录后告知我继续

#### 2.2 lark-cli

```bash
lark-cli auth status 2>&1
```

- `valid` → 已授权
- 其他状态 → 运行授权命令：

```bash
lark-cli auth login --scope "drive:drive:readonly drive:file:create docx:document:create docx:document:write_only docx:document:readonly bitable:app:create bitable:app:update bitable:table:create bitable:table:update bitable:field:create bitable:record:create bitable:record:read im:message im:message.send_as_user contact:user.base:readonly" --as user 2>&1
```

提示用户打开返回的授权链接，完成后确认继续。

---

### Phase 3：飞书资源配置

#### 3.1 文档文件夹

列出云盘根目录，找或创建存放每日 brief 的文件夹：

```bash
lark-cli api GET /open-apis/drive/v1/files --as user 2>&1
```

询问用户：**文件夹名称是什么？**（默认：`boss直聘每日简历brief`）

- 在返回列表中找到对应 `folder` 类型的条目 → 记录 `folder_token`
- 未找到 → 创建：

```bash
lark-cli api POST /open-apis/drive/v1/files/create_folder \
  --body '{"name":"boss直聘每日简历brief","folder_token":""}' \
  --as user 2>&1
```

记录 `folder_token`。

#### 3.2 多维表格

询问用户：**是否已有候选人管理表格？**

**选项 A：已有表格** → 请输入 `base_token` 和 `table_id`，验证后记录。

**选项 B：自动创建**

**Step 1** 创建多维表格：

```bash
lark-cli api POST /open-apis/bitable/v1/apps \
  --body '{"name":"Boss直聘候选人招聘管理","folder_token":"<folder_token>"}' \
  --as user 2>&1
```

从返回值取 `app_token`（即 `base_token`）。

**Step 2** 获取默认 table_id：

```bash
lark-cli api GET /open-apis/bitable/v1/apps/<base_token>/tables --as user 2>&1
```

取第一个 `table_id`，重命名默认表：

```bash
lark-cli api PATCH /open-apis/bitable/v1/apps/<base_token>/tables/<table_id> \
  --body '{"name":"候选人信息"}' \
  --as user 2>&1
```

**Step 3** 询问用户：**你们目前在招哪些职位？**（多选，输入序号，可追加自定义）

给出参考选项让用户勾选：
```
1. 安卓开发工程师      7. 产品营销
2. iOS开发工程师       8. 技术美术（TA）
3. 前端开发工程师      9. UI/UX设计师
4. 后端开发工程师     10. 供应链/采购
5. APP软件产品经理    11. 运营
6. 智能硬件产品经理   12. 自定义输入...
```

收集用户选择的职位列表，后续作为"应聘职位"字段的预设选项。

**Step 4** 创建字段（逐个执行，失败不中断，记录错误在最后统一提示）：

```bash
# 应聘职位（单选，动态选项）
lark-cli api POST /open-apis/bitable/v1/apps/<base_token>/tables/<table_id>/fields \
  --body '{
    "field_name": "应聘职位",
    "type": 3,
    "property": {
      "options": [<用户选择的职位列表，格式: {"name":"职位名"}，逗号分隔>]
    }
  }' --as user 2>&1

# 性别（单选）
lark-cli api POST /open-apis/bitable/v1/apps/<base_token>/tables/<table_id>/fields \
  --body '{"field_name":"性别","type":3,"property":{"options":[{"name":"男"},{"name":"女"}]}}' \
  --as user 2>&1

# 年龄（数字）
lark-cli api POST /open-apis/bitable/v1/apps/<base_token>/tables/<table_id>/fields \
  --body '{"field_name":"年龄","type":2}' --as user 2>&1

# 学历（单选）
lark-cli api POST /open-apis/bitable/v1/apps/<base_token>/tables/<table_id>/fields \
  --body '{"field_name":"学历","type":3,"property":{"options":[{"name":"博士"},{"name":"硕士"},{"name":"本科"},{"name":"大专"}]}}' \
  --as user 2>&1

# 毕业院校（文本）
lark-cli api POST /open-apis/bitable/v1/apps/<base_token>/tables/<table_id>/fields \
  --body '{"field_name":"毕业院校","type":1}' --as user 2>&1

# 专业（文本）
lark-cli api POST /open-apis/bitable/v1/apps/<base_token>/tables/<table_id>/fields \
  --body '{"field_name":"专业","type":1}' --as user 2>&1

# 工作年限（数字）
lark-cli api POST /open-apis/bitable/v1/apps/<base_token>/tables/<table_id>/fields \
  --body '{"field_name":"工作年限","type":2}' --as user 2>&1

# 期望城市（文本）
lark-cli api POST /open-apis/bitable/v1/apps/<base_token>/tables/<table_id>/fields \
  --body '{"field_name":"期望城市","type":1}' --as user 2>&1

# 期望薪资（文本）
lark-cli api POST /open-apis/bitable/v1/apps/<base_token>/tables/<table_id>/fields \
  --body '{"field_name":"期望薪资","type":1}' --as user 2>&1

# 活跃状态（单选）
lark-cli api POST /open-apis/bitable/v1/apps/<base_token>/tables/<table_id>/fields \
  --body '{"field_name":"活跃状态","type":3,"property":{"options":[{"name":"刚刚活跃"},{"name":"今日活跃"},{"name":"昨天"},{"name":"3天内"},{"name":"本周内"}]}}' \
  --as user 2>&1

# 近期工作经历（文本）
lark-cli api POST /open-apis/bitable/v1/apps/<base_token>/tables/<table_id>/fields \
  --body '{"field_name":"近期工作经历","type":1}' --as user 2>&1

# 初筛评级（单选）
lark-cli api POST /open-apis/bitable/v1/apps/<base_token>/tables/<table_id>/fields \
  --body '{"field_name":"初筛评级","type":3,"property":{"options":[{"name":"⭐⭐⭐⭐⭐ 强推"},{"name":"⭐⭐⭐⭐ 推荐"},{"name":"⭐⭐⭐ 可约"},{"name":"⭐⭐ 观望"},{"name":"⭐ 不推荐"},{"name":"❌ 排除"}]}}' \
  --as user 2>&1

# 初筛理由（文本）
lark-cli api POST /open-apis/bitable/v1/apps/<base_token>/tables/<table_id>/fields \
  --body '{"field_name":"初筛理由","type":1}' --as user 2>&1

# 跟进状态（单选）
lark-cli api POST /open-apis/bitable/v1/apps/<base_token>/tables/<table_id>/fields \
  --body '{"field_name":"跟进状态","type":3,"property":{"options":[{"name":"待联系"},{"name":"已发邀约"},{"name":"面试中"},{"name":"已通过"},{"name":"已Offer"},{"name":"已录用"},{"name":"已淘汰"}]}}' \
  --as user 2>&1

# 排除原因（文本）
lark-cli api POST /open-apis/bitable/v1/apps/<base_token>/tables/<table_id>/fields \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Viy1204/boss-daily-brief](https://github.com/Viy1204/boss-daily-brief) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
