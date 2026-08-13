---
trigger: always_on
description: axum 0.8 WebSocket 终端服务器。把一个 PTY 会话同时当成「浏览器前端」(WebSocket `/ws`)和「终端截图生成器」用。edition 2024,ratatui/crossterm TUI,vt100 终端模拟,portable-pty。
---

# vibetty — 项目备忘

axum 0.8 WebSocket 终端服务器。把一个 PTY 会话同时当成「浏览器前端」(WebSocket `/ws`)和「终端截图生成器」用。edition 2024,ratatui/crossterm TUI,vt100 终端模拟,portable-pty。

## 工作约定

- **改完 Rust 代码、提交/推送前先 `cargo fmt`**。CI(`.github/workflows/`)跑 `cargo fmt --check` + clippy + build(ubuntu+windows),没格式化直接挂 CI。
  - `cargo fmt --manifest-path` 的值要指向 **`Cargo.toml` 文件**,传目录会报错。
- CI 对纯 markdown/docs 改动会跳过(`bd6508f` 之后),但代码改动照常全跑。

---

## skill 子命令(`vibetty skill install/uninstall`)

把内置的 `run-vibetty` SKILL.md 装进 / 移出 agent 的**用户级** skills 目录——方便别人 `cargo install vibetty` 后一条命令装好,不用手动复制 skill 文件夹。skill 内容是教用户「后台 tmux 起 vibetty 会话、经 MQTT 把终端画面分享给 ESP32」。

```
vibetty skill install --claude          # → ~/.claude/skills/run-vibetty/
vibetty skill install --codex           # → ~/.agents/skills/run-vibetty/(Codex USER scope)
vibetty skill install --claude --codex  # 两个都装
vibetty skill uninstall --claude        # 移除(目录随后为空才删目录)
```

- `--claude` / `--codex` 是 bool flag,可同时给;都不给 → `anyhow::bail!` 报错退出。
- 两边 SKILL.md 格式一致(name + description frontmatter + 渐进披露),仓库只内嵌**一份** `resources/skills/run-vibetty/SKILL.md`,用 `include_str!` 编译进二进制(`src/skill.rs`),按 flag 写到对应目录。
- **版本感知**:install 前比 `env!("CARGO_PKG_VERSION")` 与目标目录下伴生文件 `.vibetty-version`(**不污染 SKILL.md frontmatter**)。同版本 → 跳过不重写;版本不同 / 无记录 → 覆盖升级。版本号唯一真相源是 `Cargo.toml`,发版自动跟随,不用手改 SKILL.md。
- **uninstall 安全**:删 `SKILL.md` + `.vibetty-version`,只在目录随后变空时才 `remove_dir`(**绝不**用 `remove_dir_all`,避免误删 `~/.claude/skills/` 或 `~/.agents/skills/`)。
- Codex 路径是 `~/.agents/skills/`(不是 `~/.codex/`):见 developers.openai.com/codex/skills 的 USER scope;旧 `~/.codex/prompts/` 已废弃。
- 代码:`src/config.rs` 的 `Commands::Skill { action: SkillAction }`(嵌套子命令 `Install` / `Uninstall`,各自带 `claude` / `codex` bool)、`src/skill.rs`(`run_skill` + `Agent` + `install_one` / `uninstall_one`)、`main.rs` 的 dispatch arm 镜像 `Setup`。`run_skill` 不接 `cli.config`(skill 与 MQTT 配置无关)。

---

## 终端截图:调色板 PNG(已上线 main,`dd4331b`)

`ws.rs:968` 的 PNG 分支已从「image crate 默认编码」换成 `png_encode::encode_paletted_png`,终端截图体积 ~82.5K → ~22K(PSNR ~49dB)。JPEG 质量从 100 调到 85(`ws.rs:982`)。

**为什么不用 image crate 默认 PNG**:默认 `PngEncoder` 压缩很弱,同样的调色板图 image crate 出 ~77.5K,而 png crate + `Compression::Best` 能压到 ~22K。

**`src/png_encode.rs` 的非显而易见点**:
- NeuQuant 必须用 **RGBA(4 字节/像素)** 训练,用 RGB 会让索引错位、颜色全乱(PSNR 掉到 26dB)。训练完 `color_map_rgb()` 拿调色板。
- `index_of` 入参也是 RGBA 4 字节,不能传 RGB。
- png crate 写 8-bit indexed 时 `ColorType::Indexed` + 一次性 `write_image`,配合 `Compression::Best`。

---

## 可选 MQTT 传输(feat/mqtt-transport 分支)

给 ESP32/MCU 这类不方便跑 WS 的设备加第二条传输通道。**配置驱动、可选**:只在 `~/.vibetty/config.toml` 有 `[mqtt]` 段时才启用;否则完全不碰 MQTT,WebSocket/HTTP 原样保留。两条通道并存,复用同一个 PTY 会话、`cli_tx` / broadcast `tx`、PTY 逻辑。

### 配置(`MqttConfig`,config.rs)

```toml
[mqtt]
enable = true           # transport client 的 auto-start 开关
broker = ""             # mqtt(s)://[user:pass@]host:port;空 + builtin_broker 时默认本地
builtin_broker = true   # 内置 broker(rumqttd)的 auto-start 开关
builtin_port = 1883     # 内置 broker TCP 端口
builtin_ws_port = 9001  # 内置 broker WS 端口
qos = 1                 # ⚠️ 当前未生效:inbound QoS 在 mqtt.rs 写死(pty_in=0, control=1);此字段只在 setup TUI 可编辑
keep_alive_secs = 30
```

`enable` / `builtin_broker` 现在纯粹是 **auto-start 标志**:boot 时 `run_command` 见 `enable`→起 transport client、`builtin_broker`→起内置 broker。两者还能在 TUI 弹窗里手动起/停。

**URL 解析(`MqttConfig::for_client`,config.rs)**:client 连的 broker URL 一律以 config 里 `broker` 为准;**只有** `broker` 空 **且** `builtin_broker=true` 时才默认填本地 `mqtt://127.0.0.1:{builtin_port}`。即:即便内置 broker 开着,只要 `broker` 填了,client 就连填的地址。boot 自动起、面板(重)spawn、URL 预填/比对都走这一处(唯一真相源)。

### TUI 控制(顶部按钮行)

顶部按钮行(屏幕第 1 行,原 footer)是 `HTTP | MQTT | Quit`;`[mqtt]` 配置存在即显示 `MQTT` 按钮(不再绑 `builtin_broker`),点击弹 **MqttPanel**(上下两块):
- **Broker 块**:`TCP:` / `WS :` 端口可编辑(Enter 存回 config)+ `Start broker`(起内置 rumqttd;**只能起不能停**——rumqttd 无 shutdown API,起了之后变只读 `● broker running :{port}`)。
- **Client 块**:`URL:` broker 地址可编辑(Enter 存回 `[mqtt] broker`)+ `Start client`/`Stop client`。
- `Tab`/`↑↓` 在 `TCP → WS → BrokerStart → URL → ClientToggle` 循环;Enter 行为随聚焦项变(端口=存盘、BrokerStart=起 broker、URL=存 URL、ClientToggle=切 client);底部提示也随聚焦项动态变化。
- MQTT 按钮文字反映组合状态:`MQTT off` / `MQTT brkr` / `MQTT conn` / `MQTT on`(brkr=broker 在跑 client 没跑,以此类推)。

### 顶部按钮行渲染与悬停高亮(ui/mod.rs + ws.rs)

按钮原在 footer,现已移到屏幕**第 1 行**(header 标题块已删,动态 title 改挂终端 pane 上边框左上角)。外观与鼠标交互(按钮**功能**见上节):
- 布局 `Layout::Vertical` 两段:`[Length(1) 按钮行, Min(0) 终端 pane]`。`TUI_ROWS_PADDING=2` = 按钮 1 + 终端 pane 上边框 1(无下边框);`TUI_COLS_PADDING=0`(终端 pane 只有上边框、无左右下)。
- 按钮**无边框**:`render_button` 用 `Paragraph`+`Style.bg` 整块填色(默认 `DarkGray`,悬停 `LightBlue`+黑字),不是 `Block::borders`(`borders` 会让按钮高 3 行)。
- 按钮**左右各内缩 1 格**留白(与终端上边框 title 左沿视觉对齐):`http_button_rect`=`button_row.x+1`、`quit_button_rect`=`button_row.x+button_row.width-1-width`;MQTT 按钮从 HTTP 右侧推导、自动跟随。
- **命中 rect 三处共用**:`render_frame` 渲染、`handle_click` 点击、`button_row_at` 悬停命中都走 `*_button_rect`/同一 `Layout`——改按钮布局要同步这几处。
- **悬停高亮**靠 crossterm `?1003h`(any-event,`Moved` 无需按键上报),两层节流避免刷屏:
  1. producer 侧(`event_loop_thread`):按钮在第 1 行(`row==0`,不随窗口高度变),`Moved` 只在 `row==0`(进入/在按钮行内移动)或刚离开按钮行时转发;终端区滑动直接丢、不进 channel。
  2. consumer 侧(`run_command` Hover 分支):`button_row_at` 算当前悬停按钮,只在变化时重绘。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [second-state/vibetty](https://github.com/second-state/vibetty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
