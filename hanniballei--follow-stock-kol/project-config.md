---
trigger: always_on
description: - 设计：[docs/DESIGN.md](docs/DESIGN.md)
---

# AGENTS.md · 给后续协作开发的注意事项

最后更新：2026-05-30
项目：美股 KOL 推特监控
配套文档：
- 设计：[docs/DESIGN.md](docs/DESIGN.md)
- 实施：[docs/IMPLEMENTATION.md](docs/IMPLEMENTATION.md)
- 测试：[docs/TESTING.md](docs/TESTING.md)
- 运维：[docs/OPERATIONS.md](docs/OPERATIONS.md)

> 这份文档面向接手或扩展本项目的 AI 助手 / 工程师，记录"光看代码看不出来"的决策和坑。请在重大改动前先读完，并在踩坑后回来更新"已知问题与教训"区。

---

## 0. 已经预先创建的文件

不要重新创建以下文件，直接复用 / 增量编辑：

- `config/kols.yaml` — 56 位 KOL，按字母序，handle 大小写如 X 实际显示
- `config/settings.yaml` — 全部可调参数（调度、抓取、媒体、AI、发布、保留、日志）
- `.env.example` — 环境变量模板，复制为 `.env` 填值；本机覆盖可放 `.env.local`
- `.gitignore` — 已正确忽略 `.env / .env.local / *.db / *.log / media/` 等

---

## 1. 凭据和配置

### 1.1 必备环境变量（`.env`）

模板见 [`.env.example`](.env.example)。核心变量：

```bash
OPENTWITTER_TOKEN=          # 必填，6551.io 注册申请
OPENTWITTER_BASE_URL=https://ai.6551.io   # 一般不改
ANTHROPIC_API_KEY=          # 必填
ANTHROPIC_BASE_URL=         # 走代理时填，走官方留空
# ANTHROPIC_MODEL=          # 可选覆盖；默认从 settings.yaml.ai.model 读
ANTHROPIC_FALLBACK_API_KEY= # 可选备用 Claude key
ANTHROPIC_FALLBACK_BASE_URL= # 可选备用 Claude base URL；为空则复用 ANTHROPIC_BASE_URL
ANTHROPIC_THIRD_API_KEY=    # 可选第三层 Claude 兼容后端 key
ANTHROPIC_THIRD_BASE_URL=   # 可选第三层 Claude 兼容后端 base URL
ANTHROPIC_THIRD_MODEL=      # 可选第三层模型名；默认同 settings.yaml.ai.model
# KOL_MONITOR_DB=           # 可选覆盖 SQLite 路径
# KOL_MONITOR_MEDIA_DIR=    # 可选覆盖媒体目录
KOL_MONITOR_ALLOW_PUSH=false # 可选；默认不执行远端 git push
```

### 1.2 凭据安全

- `.env` 和 `.env.local` 必须在 `.gitignore` 里。如果不小心 commit 了，**立即** rotate token。
- `kol_monitor.log` 也要在 .gitignore，因为 rich 日志可能在 stack trace 里 echo 请求 header。
- 任何对外发的报告都不能包含 token；anthropic SDK 出错时它会在 traceback 里打印 base_url，base_url 里如果带了鉴权也敏感。

---

## 2. 6551.io API 关键事项

### 2.1 速率与计费

- 文档没有公开的速率限制数字，**实测前先小心**：56 个 KOL 一轮抓取期间 KOL 之间 sleep 2-5 秒，整个 batch 约 5-10 分钟。
- 计费按调用量，跑批前看一眼 [6551.io 控制台](https://6551.io/mcp) 余额。
- token 用尽时 API 一般返回 4xx，**千万不要重试 4xx**（tenacity 装饰器只对 ConnectError / ReadTimeout 重试，不要扩到 HTTPStatusError）。

### 2.2 没有 cursor，靠 maxResults 翻页

`/open/twitter_user_tweets` **不支持 cursor**，每次都从最新拿。要"翻深一点"靠扩大 `maxResults`（上限 100）。这就是 fetcher.py 里 `page_size = min(page_size*2, 100)` 的原因。如果某天某 KOL 发了超过 100 条还要全部拿到，必须用 `/open/twitter_search` 兜底（`fromUser` + `sinceDate`）。

### 2.3 媒体字段不全

6551 返回的 `media[]` 里只承诺 3 个字段：`type` / `url` / `thumbUrl`。视频的码率、时长、多版本 variants 都**不暴露**。我们的设计是视频只存 URL（不分析），所以够用；如果未来要喂 AI 视频，必须换底层（twscrape 或自抓）。

### 2.4 字段名注意

6551 用的是驼峰：`createdAt`, `userScreenName`, `retweetCount`，不是 X 原生 API 的 `created_at`。client.py 里要做一次字段名转换到内部 snake_case，避免散在各处。

---

## 3. 防漏拉边界条件（务必读）

### 3.1 tweet_id 比较用数值键不是字符串

X 的 tweet_id 是 64-bit snowflake，单调递增。比较时**一定要转成数值排序键**。SQLite 列是 TEXT 没关系（因为长度可能超 int8），但比较时务必：

```python
last_id_int = tweet_id_sort_value(last_id_str) if last_id_str else 0
new_tweets = [t for t in batch if tweet_id_sort_value(t["tweet_id"]) > last_id_int]
```

注意：`realDonaldTrump` 经 6551 返回的 ID 可能是 `truth_1780116388200996844` 这种带前缀形式。比较时取数字后缀做排序键，但数据库里的 `tweet_id` 和 `last_seen_tweet_id` 要保留原始完整字符串。

### 3.2 last_seen_tweet_id 更新原则

**只在本轮真有新推时更新**。如果某次拉取一条新都没有（或全部重复），保持原值不动。错误的实现会把 last_id 一路往前推到"最新无新推时的最大值"，下次拉漏掉的概率反而升高。

### 3.3 incomplete 标记的语义

`incomplete = True` 含义是"上次拉取**没在 MAX_ROUNDS 内找到与锚点的重叠**"，可能漏推。补救任务（每次主任务跑完后追加）要用 `/open/twitter_search` `fromUser=handle, sinceDate=last_fetched_at-1d` 强制扫一遍，扫完清标记。**别用 user_tweets 重试，因为没 cursor 翻不到更深**。

### 3.4 首次接入 vs 增量

```python
if last_id is None:           # 首次
    pull(20)                  # 不翻页，按用户要求只拉当天
else:                         # 增量
    while round <= 5: ...     # 翻页直到重叠
```

注意"首次"和"`last_id == 0`"是两回事，**不要把首次当成 last_id=0 用**（会无限翻页直到 MAX_ROUNDS）。

### 3.5 KOL handle 改名 / 注销

- 改名：6551 返回 `userIdStr` 不变，依赖 `screen_name` 做 join 会丢数据。**用 `twitter_user_id` 做长期标识更稳**，但 schema 已经用 `screen_name UNIQUE`，将来若要切换需迁移。当前先按 screen_name 处理，加监控（`fetch_runs.error_log` 里出现 user_info 失败时报警）。
- 注销 / 拉黑 / 私密：`twitter_user_info` 返回空或 4xx → `mark_kol_inactive(handle, reason=...)`，跑批不影响其他人。

---

## 4. Claude API 注入

### 4.1 base_url 优先级

```python
from anthropic import Anthropic

# 优先用 .env 里的 base_url；为空则用官方
client = Anthropic(
    api_key=os.environ["ANTHROPIC_API_KEY"],
    base_url=os.environ.get("ANTHROPIC_BASE_URL") or None,
)
```

不要写死 base_url。代理地址的协议头要带（`https://...`），SDK 不会自动补。

### 4.2 模型 ID

`claude-sonnet-4-6` 是 2026 当前的有效 ID。如果代理服务用别的 ID（比如 `claude-3-sonnet-..`），需要在 `settings.yaml.ai.model` 字段覆盖。

### 4.3 多模态消息构造

图片走 base64 inline：
```python
{
  "role": "user",
  "content": [
    {"type": "image", "source": {
      "type": "base64",
      "media_type": "image/jpeg",
      "data": base64_str
    }},
    {"type": "text", "text": "..."},
  ]
}
```

单条 message 总图数没有硬上限，但建议每个 KOL ≤ 8 张避免 token 爆炸。**JPEG 质量保留原图**，PNG 转 JPEG 时要处理透明通道（贴白底），不然 PIL 报错。

### 4.4 JSON 输出 fallback

Layer 2 要求 JSON 输出，模型偶尔会包一层 markdown ```json``` 代码块。Parse 顺序：

```python
def parse_layer2(text):
    # 1. 直接 json.loads
    try: return json.loads(text)
    except: pass
    # 2. 剥 markdown code fence
    m = re.search(r"```(?:json)?\s*(.+?)```", text, re.DOTALL)
    if m:
        try: return json.loads(m.group(1))
        except: pass
    # 3. 找第一个 { 到最后一个 }
    s, e = text.find("{"), text.rfind("}")
    if s != -1 and e > s:
        try: return json.loads(text[s:e+1])
        except: pass
    return None  # 这条 KOL 标 summary_failed，不影响其他
```

---

## 5. 媒体下载

### 5.1 文件名格式


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hanniballei/follow-stock-kol](https://github.com/hanniballei/follow-stock-kol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
