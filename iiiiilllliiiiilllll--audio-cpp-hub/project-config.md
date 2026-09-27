---
trigger: always_on
description: audio.cpp-hub 是 [audio.cpp](https://github.com/0xShug0/audio.cpp) 的 Web 管理面板：一个用 Go 写的轻量 HTTP 服务（原生单二进制，无需任何运行环境），负责拉起 / 停止 / 监控多个 `audiocpp_server` 模型实例子进程，并提供中文为主的 Web UI 进行 TTS / ASR / 音乐分离等音频任务。仓库地址：https://github.com/IIIIIllllIIIIIlllll/audio.cpp-hub
---

# audio.cpp-hub

## 项目概述

audio.cpp-hub 是 [audio.cpp](https://github.com/0xShug0/audio.cpp) 的 Web 管理面板：一个用 Go 写的轻量 HTTP 服务（原生单二进制，无需任何运行环境），负责拉起 / 停止 / 监控多个 `audiocpp_server` 模型实例子进程，并提供中文为主的 Web UI 进行 TTS / ASR / 音乐分离等音频任务。仓库地址：https://github.com/IIIIIllllIIIIIlllll/audio.cpp-hub

- 入口：仓库根目录 `main.go`（`func main`），Go module 为 `github.com/IIIIIllllIIIIIlllll/audio.cpp-hub`
- hub 本身默认监听 `httpPort`（`hub.config.json`，本仓库开发副本为 18080，代码内默认 8080）；各模型实例从 `instancePortBase`（本副本 18090）起自动分配端口，绑定 127.0.0.1
- 模型实例不在 hub 进程内运行：hub 为每个实例写 `run/<id>/server.json`，再用 `os/exec` 拉起外部 `audiocpp_server --config server.json`，stdout/stderr 重定向到 `run/<id>/server.log`，并后台轮询实例的 `/health`（最多 120s，见 `instance.go`）
- 推理任务走**异步队列**（前端主链路）：`POST /api/tasks`（body `{"instanceId","request":{...}}`）创建任务立即返回，`TaskManager`（`task.go`）为每个实例起一个单线程 executor **同实例串行排队执行**（与引擎 busy 锁语义一致），**无执行时长上限**；`GET /api/tasks[?active=1&modelId=]`（列表，活跃在前）、`GET /api/tasks/<id>`（详情含队列位置 position）、`GET /api/tasks/<id>/result`（非 TTS 结果 `data/tasks/<id>.result.json` 流式回写）、`DELETE /api/tasks/<id>`（QUEUED 直接取消 / RUNNING 中断 hub 侧等待——引擎会跑完，属已知限制 / 已结束则删除记录）。任务状态落盘 `data/tasks/<id>.task.json`（每次状态变迁原子写），hub 重启回放重建（上次中断时进行中的任务标记为 CANCELLED），已完成任务内存保留最近 100 条。TTS 任务复用历史链路：taskId 即历史记录 id，响应落盘后 `history.go` 中的音频提取器流式扫描提取 `"audio"` 写成 `data/history/<modelId>/<taskId>.wav`，前端结果音频直接用 `/api/history/.../audio` URL（不碰 base64）；非 TTS 结果统一 `forwardToFile` 落盘。前端 2s 轮询，**任务并入右侧操作历史侧栏**（任务创建即一条记录：进行中的在前，已结束的其次；TTS 终态与历史按 taskId 去重由历史行代表，非 TTS 完成任务行带「载入」可重新渲染结果、「详情」行内展开完整文本结果；进行中行内可取消，侧栏对所有类别开放），允许连续提交排队；页面加载与模型切换时经 `?modelId=` 重挂全部任务（进行中的恢复轮询），**刷新页面不再丢任务**。旧 `POST /api/run/<instanceId>` 同步接口保留兼容，TTS 流式链路：api.go → 实例 `http://127.0.0.1:<port>/v1/tasks/run`，TTS 响应落盘 → 提取进历史 → 临时文件分块回写
- 操作历史（TTS，`history.go`）：按 modelId 隔离到 `data/history/<modelId>/`（`index.jsonl` 一行一条记录只追加 + `<taskId>.wav` 结果音频 + `<taskId>.ref|emo|spkN.wav` 参考音频快照——记录时把请求里的 voice_ref/audio/voice_samples 源文件复制进历史目录，历史自包含，快照随记录一并删除），内存索引启动时回放重建。**无数量/容量淘汰**：历史是用户资产，只由用户手动删除（原 Java 版有 50 条/500MB 上限，Go 版按用户要求移除）。记录含 `refs`（快照名→原始文件名）、`refBytes`、`groupId` 等可选字段，旧记录向后兼容。API：`GET /api/history/<modelId>`（简要列表，新→旧，text 截断 100 字并带 `textTruncated` 标记）、`GET /api/history/<modelId>/<taskId>`（完整记录）、`GET /api/history/<modelId>/<taskId>/audio`（流式回 wav）、`GET /api/history/<modelId>/<taskId>/audio/<name>`（参考音频快照，name 为 ref|emo|spkN）、`DELETE /api/history/<modelId>[/<taskId>]`（清空 / 单删）。手动分组存 `groups.json`：`GET|POST /api/history/<modelId>/groups`、`PUT|DELETE .../groups/<gid>`、`PUT .../<taskId>/group`（移入/移出组；删组记录回未分组，清空历史保留分组）。前端为页头 🕘 按钮弹出的全屏面板（替换原右侧边栏），音频懒加载（点击播放才拉取 wav），历史行「详情」行内展开四要素（参考音频/参考文本/音色提示词/生成内容）、「移动」弹菜单换组，分组可折叠；界面记住上次选中的模型（localStorage `hub-model`），刷新后历史视图不丢
- 参考音频（音色库，`voices.go`）：全局资源，存 `data/voices/`（`<vid>.wav` + `index.json`），条目 = vid + 名称（全局唯一，重名拒绝 `VOICE_NAME_EXISTS`）+ 音频文本内容 `text`（部分模型要求参考音频配套文本）+ 音频文件。API：`GET /api/voices`（列表含 text）、`POST /api/voices`（`{name, text?, uploadId?|path?}`，上传件或服务器路径二选一）、`PUT /api/voices/<vid>`（改名称/文本，排除自身重名）、`GET /api/voices/<vid>/audio`（流式回 wav）、`DELETE /api/voices/<vid>`。前端：页头 🎙 按钮弹出全屏管理面板（voices-panel.js：列表/试听/行内编辑/删除/添加——添加走完整 AudioPicker 上传/录制/裁剪）；TTS 表单里所有参考音频入口（主 voice_ref、VibeVoice 多说话人、其它模型 voice_ref、index_tts2 情感参考）统一用 `VoiceSelect` 下拉组件（voice-select.js），选中即生效并把音色路径填入请求的 voice_ref，自动回填参考文本；ASR/分离等仍用 `AudioPicker`（已移除其「保存到音色库」入口，库管理只在大面板）
- OpenAI 兼容代理（`proxy.go`）：`GET /v1/models` 聚合全部 READY 实例的服务名；`POST|PUT /v1/*`（如 `/v1/audio/speech`）——请求体流式落盘到 `run/proxy-cache/`（上限 `hub.config.json` 的 `proxyMaxBodyBytes`，默认 1GB），逐字节扫描提取顶层 `"model"`（大 base64 字段不落内存）后按服务名路由（READY 才转发，启动中 409，不存在 404），落盘文件作为 body 转发到实例同名接口，响应状态码/Content-Type 透传、逐块 Flush（SSE 兼容）；上游无整体超时，客户端断开即取消。错误体为 OpenAI 风格 `{"error":{"message","type"}}`。已知限制：multipart/form-data 无法提取 model（extractor 只认 JSON），会 400
- 实例服务名（instanceName）：启动时可显式指定（默认 modelId），是 `/v1/*` 的路由键，也写进实例 server.json 的 model id（实例自校验一致）；全局唯一，重名拒绝启动
- 设备探测：`GET /api/executables/<id>/devices` 用该可执行文件运行 `--list-devices`（注入条目 env，60s 超时），解析输出为 `{devices:[{backend,index,name,type}],raw}`；前端启动弹窗打开/切换程序时自动探测，「设备」为下拉选单（选项显示设备名称，选中即联动后端，提交设备号）
- 高级参数（sessionOptions）：启动模型弹窗底部「高级参数」区按每行 key=value 填写，API 为启动/配置 body 的 `sessionOptions` 对象（值统一转字符串），经 `optStringMap` 校验后写入 server.json 模型条目的 `session_options`；随启动配置（Profile）持久化

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IIIIIllllIIIIIlllll/audio.cpp-hub](https://github.com/IIIIIllllIIIIIlllll/audio.cpp-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
