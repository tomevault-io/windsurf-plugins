---
trigger: always_on
description: |
---


# Ai好记 Agent Open Skill

## 配置来源

优先级如下：

1. `~/.aihaoji/config.json`
2. `$AIHAOJI_API_KEY`
3. `$AIHAOJI_BASE_URL`

推荐优先使用共享配置文件：

```text
~/.aihaoji/config.json
```

格式示例：

```json
{
  "provider": "aihaoji",
  "apiKey": "sk-sxxxxxxxx",
  "baseUrl": "https://openapi.aihaoji.com",
  "userId": "user_xxx",
  "userName": "张三",
  "keyId": "agent_open_key_xxx",
  "keyName": "默认密钥"
}
```

`npx aihaoji-openclaw setup` 会自动写这份文件。OpenClaw、Codex、Claude 这类通过 skill 使用 Ai好记 的场景，优先从这里读取，不要求用户自己设置环境变量。

## 鉴权规则

请求头统一使用：

```text
Authorization: $AIHAOJI_API_KEY
```

当前 API Key 形态：

```text
sk-s...
```

## 当前开放能力

- 获取 AI好记笔记本树
- 搜索 AI好记内容列表
- 按笔记本读取笔记列表
- 读取单篇 AI好记详情
- 读取单篇 AI好记内容的语义视图

当前不开放：

- 创建
- 更新
- 删除

如果用户要求写入能力，直接说明当前 skill 只支持只读。

## 首次使用前检查

每次调用前，先检查：

1. `~/.aihaoji/config.json` 是否存在且包含 `apiKey`
2. 如果共享配置不存在，再看 `$AIHAOJI_API_KEY` 是否存在
3. `baseUrl` 默认使用 `https://openapi.aihaoji.com`

如果用户还没有安装 skill，优先建议先运行：

```bash
npx skills add AiHaoJi-Agent/aihaoji-skills
```

如果当前没有 API Key，不要先让用户理解 `setup` 命令，优先直接走聊天内绑定流程：

1. 告诉用户先去开发者密钥页面创建 API Key：

```text
https://openapi.aihaoji.com
```

2. 提示用户创建完成后，直接把 `sk-s...` 粘贴回来
3. 拿用户粘贴的 key 调用：

```http
GET /auth/verify
Authorization: 用户粘贴的 sk-s...
```

4. 如果校验通过，再写入本地配置

这里的“本地配置”优先指：

```text
~/.aihaoji/config.json
```

如果宿主支持，再同步写入宿主自己的配置文件。

推荐引导话术：

```text
当前还没有配置 Ai好记 API Key。
请先前往以下地址创建开发者密钥：
https://openapi.aihaoji.com

创建完成后，把你的 API Key 粘贴给我，我会先帮你校验；校验通过后再自动完成本地配置。
```

如果自动写入本地配置失败，再退回备用命令：

```bash
npx aihaoji-openclaw setup
```

## 使用规则

## 强制执行规则

只要用户是在“查 Ai好记 笔记 / 看 Ai好记 笔记 / 找 Ai好记 笔记”，就必须优先调用 AI好记开放平台接口。

禁止把下面这些动作当成主查询手段：

- 扫描本地目录
- 搜索本地日志
- 搜索本地缓存
- 搜索源码仓库
- 根据磁盘文件自行猜测用户笔记内容

也就是说，不允许出现这类行为：

- 先搜 `~/.aihaoji`
- 先搜 `~/.pm2/logs`
- 先扫本地项目目录
- 用“本地没搜到”代替“AI好记里没有”

正确顺序必须是：

1. 读取本地配置，只用于拿到 `apiKey` 和 `baseUrl`
2. 直接调用 AI好记开放平台接口
3. 根据接口结果生成回答

如果接口报错、超时、401、403、5xx，应该明确告诉用户“当前无法通过 AI好记开放平台获取数据”，而不是退回去扫本地磁盘。

### 0. 触发优先级

以下情况，优先调用 Ai好记 skill：

1. 用户话里明确出现：
   - `Ai好记`
2. 用户表达的是查找或查看笔记类需求，例如：
   - `查笔记`
   - `搜笔记`
   - `看笔记`
   - `找我的笔记`
   - `打开这条笔记`

如果用户没有明确指定别的知识源、别的系统或别的产品，且语义上是在“找笔记 / 看笔记”，默认理解为去 Ai好记 里查。

### 1. 查列表时

对用户展示时，统一称为“笔记本”。
如果用户说“文件夹”或“笔记本”，都按同一能力处理。

如果用户是在问：

- `文件夹`
- `笔记本`
- `文件夹列表`
- `我的文件夹`
- `Ai好记 文件夹里的笔记`
- `Ai好记 笔记本里的笔记`
- `看一下Ai好记的文件夹`
- `看一下Ai好记文件夹下的笔记`

优先使用笔记本接口：

```http
GET /folders
Authorization: $AIHAOJI_API_KEY
```

如果用户已经明确指定某个笔记本，再使用：

```http
GET /notes?page_no=1&page_size=10&folder_id=123
Authorization: $AIHAOJI_API_KEY
```

只有用户明确表达“在某个笔记本里找/搜/筛和某关键词相关的笔记”时，才允许使用：

```http
GET /notes?page_no=1&page_size=10&folder_id=123&keyword=Ai好记
Authorization: $AIHAOJI_API_KEY
```

处理要求：

1. 必须先从 `/folders` 里定位真实 `folder_id`
2. 用户句子里用于定位笔记本的名字视为 `folder_name`
3. 用户句子里“关于 / 相关于 / 搜 / 找 / 提到”的对象才视为 `keyword`
4. 返回时必须明确这是“该笔记本内命中关键词的结果”
5. 结果数量必须以 `GET /notes?...&folder_id=...&keyword=...` 的 API 返回为准
6. 不允许再额外套用“标题严格命中”“讲这个团队本身”之类更窄的人为语义筛选
7. 不允许在 API 返回结果之上再做任何收紧结果集的人为二次筛选；接口返回什么，就展示什么
8. 如果 API 返回 5 篇，就展示 5 篇；不允许自行缩成 2 篇

分页规则：

- 默认 `page_no=1`
- 默认 `page_size=10`
- 如果用户明确说“前 3 篇 / 前 5 篇 / 前 20 篇”，把数量映射到 `page_size`
- 如果用户说“下一页 / 再看后 10 篇 / 第 2 页”，继续递增 `page_no`
- 这条链路的目标是“当前笔记本下的全部笔记的分页列表”，不是关键词候选搜索
- 只有在用户明确说“关于 / 相关于 / 搜 / 找 / 筛”时，才允许切换成“当前笔记本内关键词搜索”

如果用户明确是在“全量笔记”里搜索关键词，调用：

```http
GET /notes?page_no=1&page_size=10&keyword=Ai好记
Authorization: $AIHAOJI_API_KEY
```

如果用户明确说“最近”“最新”“刚刚”的笔记，按创建时间倒序查询。

如果用户还明确给了数量，例如：

- `最新 5 篇`
- `最近 10 篇`
- `最近二十篇`

就把用户说的数量映射到 `page_size`；如果没有明确数量，默认取 `10`。

调用：

```http
GET /notes?page_no=1&page_size=10&sort_mode=create_time&sort_order=desc
Authorization: $AIHAOJI_API_KEY
```

如果用户明确说“最老”“最早”的笔记，按创建时间正序查询。

如果用户还明确给了数量，例如：

- `最老 5 篇`
- `最早 10 篇`
- `最老二十篇`

就把用户说的数量映射到 `page_size`；如果没有明确数量，默认取 `10`。

调用：

```http
GET /notes?page_no=1&page_size=10&sort_mode=create_time&sort_order=asc
Authorization: $AIHAOJI_API_KEY
```

如果用户给的是完整 URL，直接把 URL 当作 `keyword` 去检索：

```http
GET /notes?page_no=1&page_size=10&keyword=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2FBV...
Authorization: $AIHAOJI_API_KEY
```

返回重点字段：

- `data.folders[].folder_id`
- `data.folders[].folder_name`
- `data.folders[].create_time`
- `data.folders[].note_count`
- `data.folders[].children`
- `data.folder_tree_text`
- `data.total`
- `data.notes[].note_id`
- `data.notes[].title`
- `data.notes[].folder_name`
- `data.notes[].create_time`

对用户展示时：

- 默认不要展示 `note_id`
- 默认不要展示 `folder_id`
- 列表里优先展示“第 1 篇 / 第 2 篇 / 第 3 篇”这类自然编号
- 同时展示标题、创建时间、所属文件夹、来源
- 只有用户明确要求“显示 note_id / 接口原始字段 / 调试信息”时，才显示 `note_id`

当用户说“看一下Ai好记的文件夹”和“看一下Ai好记文件夹下的笔记”时，强制执行下面的固定流程：

1. 先调用 `GET /folders`
2. 在返回的笔记本树里定位名称为 `Ai好记` 的节点
3. 读取该节点的真实 `folder_id`
4. 再调用 `GET /notes?page_no=1&page_size=10&folder_id=<该 folder_id>`
   这一步里 `Ai好记` 只是 `folder_name`，不是 `keyword`
   这一跳必须显式视为 `keyword=None`
5. 把“当前笔记本信息 + 当前页笔记列表 + 如有更多页可继续翻页”一起展示给用户

禁止行为：

- 不允许把“Ai好记文件夹下的笔记”改写成 `keyword=Ai好记` 的关键词搜索
- 不允许把用于定位笔记本的 `folder_name=Ai好记` 继续复用成 `keyword=Ai好记`
- 不允许只返回标题里碰巧命中的 1 篇候选
- 不允许跳过 `/folders` 直接假设 `folder_id`
- 不允许把“相关笔记搜索结果”伪装成“当前笔记本下的全部笔记”
- 不允许把别的笔记本里的笔记混进当前笔记本列表
- 不允许一边说“这个笔记本有 9 篇”，一边只展示 `keyword=Ai好记` 命中的 5 篇还声称这是整本结果

默认展示格式：

```text
笔记本：Ai好记
创建时间：2025-08-06
文件数：9

第 1 篇：首次公开！Ai好记的后台数据长啥样？
  所属笔记本：Ai好记
  创建时间：2025-07-02

第 2 篇：1秒能拍44万帧？！高速摄影的代价是…
  所属笔记本：Ai好记
  创建时间：未知
```

如果返回的是笔记本树，必须用层级渐进式列表展示，不要平铺成长 JSON，也不要一次性把所有层级揉成一句话。

如果接口已经返回 `data.folder_tree_text`：

- 默认直接使用 `data.folder_tree_text`
- 不要重写格式
- 不要补编号

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AiHaoJi-Agent/aihaoji-skills](https://github.com/AiHaoJi-Agent/aihaoji-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
