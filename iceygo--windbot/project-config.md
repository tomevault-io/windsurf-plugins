---
trigger: always_on
description: - WindBot 是面向 YGOPro/YGOSharp/SRVPro 协议的 C# 决斗机器人，本质上是一个自动操作的 YGOPro 客户端。
---

# AGENTS.md

## 项目定位

- WindBot 是面向 YGOPro/YGOSharp/SRVPro 协议的 C# 决斗机器人，本质上是一个自动操作的 YGOPro 客户端。
- 主程序基于 .NET Framework 4.8，解决方案支持 Any CPU 和 x86，当前默认平台与 CI 主验证平台为 Any CPU；项目是旧式非 SDK `.csproj`，不要默认使用仅适用于现代 .NET/SDK 项目的工具和 API。
- 机器人只能使用服务器发给当前客户端的信息。对方手牌、牌组、盖卡等未公开信息通常只有数量或 `Id == 0` 的占位对象，不能读取、推断或硬编码其真实内容。
- 服务器在要求客户端响应时已经给出了合法操作候选。牌组 AI 的职责主要是判断“现在是否值得这样做”以及“选哪个目标”，而不是复述发动条件。

## 解决方案与运行链路

解决方案包含两个项目：

- `WindBot.csproj`：控制台主程序，包含网络协议、客户端状态和全部决斗 AI；输出 `WindBot.exe`。
- `BotWrapper/BotWrapper.csproj`：供 YGOPro 人机模式调用的轻量启动器，整理启动参数并启动 `WindBot.exe`；输出 `Bot.exe`。

主程序的调用链如下：

1. `Program.cs` 读取命令行或配置文件，初始化牌组注册表和 `cards.cdb`，然后以单实例模式或 HTTP server 模式启动机器人。
2. `Game/GameClient.cs` 建立连接、进入房间并把收到的数据包交给 `GameBehavior`。
3. `Game/GameBehavior.cs` 按 `StocMessage`/`GameMessage` 解包，更新 `Duel`、`ClientField`、`ClientCard` 等客户端可见状态，并在需要响应时调用 `GameAI`。
4. `Game/GameAI.cs` 整理服务端给出的合法候选，按优先级查询当前牌组的 `Executor` 并返回决策；`GameBehavior` 负责验证、编码并把响应发回服务器。
5. `Game/AI/DecksManager.cs` 通过反射发现带 `[Deck]` 的执行器，并为每局实例化对应牌组 AI。
6. `Game/AI/Executor.cs` 定义公共回调和有序的 `CardExecutor` 列表；各牌组通常继承 `DefaultExecutor`。

server 模式会为每个 HTTP 请求创建独立线程和独立的 `GameClient`/`Duel`/`Executor`。不要把单局可变状态放进静态字段，也不要让不同机器人实例共享未同步的牌组状态。

## 目录职责

- `Game/`：决斗状态、消息处理、主阶段/战斗阶段动作模型。
- `Game/AI/`：AI 调度、选择队列、通用决策、卡片扩展方法和领域常量。
- `Game/AI/Decks/`：各牌组的专用执行器，是新增或调整牌组逻辑的首选位置。
- `Game/AI/Enums/`：跨牌组共享的已知卡分类，例如压制卡、危险怪兽、不可取对象等。
- `YGOSharp.Network/`：TCP 数据帧和 CTOS/STOC 协议层。
- `YGOSharp.OCGWrapper/`、`YGOSharp.OCGWrapper.Enums/`：卡片数据库模型和协议/规则枚举。
- `Decks/`：运行时 `.ydk` 牌组文件。
- `Dialogs/`：运行时 JSON 对话资源。
- `BotWrapper/`：外部 bot 启动包装器及其配置。

层次边界应保持清晰：

- 协议解析和客户端状态同步放在 `GameBehavior`/网络层。
- 通用、确实能被多个牌组复用的决策放在 `DefaultExecutor`、`AIUtil`、`CardExtension` 或共享枚举中。
- 单一牌组的展开路线、卡片优先级和临时标志留在对应牌组执行器中。
- 不要为了一个牌组的特例污染协议层或全局默认逻辑。

## AI 决策模型

### `AddExecutor` 顺序就是优先级

`GameAI` 会按注册顺序遍历 `Executor.Executors`，再遍历服务器给出的候选卡；第一个匹配且返回 `true` 的执行器立即胜出。因此：

- 构造函数中的 `AddExecutor` 顺序具有决定性影响，调整顺序属于行为修改，不是格式整理。
- 更具体、更紧急的响应放在前面，通用处理和兜底放在后面。
- 同一张卡的不同效果或不同局面可以注册多次；需要用 `Card`、`ActivateDescription`、`CurrentTiming` 和当前连锁状态区分。
- `AddExecutor(type, cardId)` 表示匹配后无条件接受；只应在所有合法出现时都适合执行的场景使用。
- `GoToBattlePhase`、`GoToEndPhase`、`Surrender` 等没有卡片上下文，其条件函数不能依赖 `Card`。

服务器给出的候选已经满足“能否发动/召唤”的基本规则，所以执行器条件应回答“是否适合发动/召唤”。只有在策略确实需要时，才检查会被无效、资源不足、后续路线冲突、每回合次数等因素。
特别地，支付生命值发动的效果在规则上允许自杀，此类效果的执行器条件应该检查当前生命值以避免自杀。

执行器条件可能在一次响应过程中被多次查询。避免在返回 `false` 的路径上消耗资源、写入已发动标志或排入残留选择；只有确定接受动作时才提交与该动作绑定的选择和状态。

### 已知架构局限

- 机器人无法得知自己或对方发动某个效果后，游戏引擎和卡片脚本接下来会如何处理。它只能依据服务器当前已发送的状态，以及编写者对卡片效果和规则的知识做基本判断；这是本项目最主要的局限之一。不要把策略写成能够预演脚本、完整结算效果或读取未来状态。
- 卡片逻辑中的选卡是预先选择：执行器接受动作时通过 `AI.Select*` 排入选择，等处理效果的选择消息到达后再消费。目前预选项与具体效果、连锁环节之间没有稳定的一一对应关系，同一连锁包含多次发动或选择流程与预期不一致时可能消费错位；这是另一项主要局限。
- 预先选择的顺序必须与卡片脚本实际发出选择请求的顺序一致。卡片脚本可参考 `https://raw.githubusercontent.com/Fluorohydride/ygopro-scripts/refs/heads/master/c{id}.lua`，其中 `{id}` 替换为卡片密码；较复杂的效果不能仅凭效果文本猜测选目标、选cost、选素材等请求的先后顺序。

### 上下文、状态与选择

- `Executor.SetCard` 会在查询执行器条件前设置当前 `Type`、`Card`、`ActivateDescription`、`CurrentTiming`。这些字段只表示“当前正在查询的执行器候选”，不是当前决斗动作或当前连锁的全局上下文；仅可在执行器条件及其同步调用的辅助方法中使用。
- `OnSelectCard`、`OnSelectPlace`、`OnSelectPosition` 等函数和生命周期回调不得依赖上述字段。应根据回调参数、`Duel.GetCurrentChainCard()`、`Duel.GetCurrentSolvingChainInfo()` 或明确维护的牌组状态识别上下文；在回调中假设评估某张候选卡时，应把候选卡、效果描述和时点显式传给辅助方法。
- `Bot` 和 `Enemy` 分别是本机视角的 `Duel.Fields[0]` 与 `Duel.Fields[1]`；协议玩家编号应通过现有本地化逻辑转换，不要自行假定座位编号。
- 优先使用 `ClientField`、`ClientCard`、`AIUtil`、`CardExtension` 的现有查询方法，避免重复遍历和散落的区域位掩码。
- 未知卡的 `Id` 可能为 `0`，`Data`/`Name` 可能为 `null`。对隐藏区域只能依赖客户端实际知道的数量、位置和已公开历史；己方牌组的卡号计数应通过专用 API 查询。
- 脚本的 `aux.Stringid(code, index)` 与 WindBot 的 `Util.GetStringId(id, option)` 使用相同编码：`cardId * 16 + offset`。其中 `offset` 是从 `0` 开始的字符串偏移量，不是 Lua 表下标；它对应 YGOPro 的 `cards.cdb` 的 `texts.str{offset + 1}`，例如偏移量 `0` 对应 `str1`，偏移量 `3` 对应 `str4`。
- `StringId` 的偏移量不一定等同于卡片效果编号。判断某个描述值的实际语义时，应同时核对卡片脚本中该值传给了哪个 API。难以确定时可以查询 `cards.cdb` 中对应的 `texts.str*` 内容。
- 避免无意义或错误地检查 `ActivateDescription`。只有同一候选上下文确实可能对应多个效果、且协议会提供真实效果描述时才用它区分；单独出现的可选诱发效果可能以通用提示到达并被转换为 `-1`，此时 `-1` 只表示描述未知，不能默认映射到某个效果。优先用卡片位置、阶段、事件状态等可靠上下文区分；脚本在单个效果内部通过 `Duel.SelectOption` 等 API 提供的选项也不能用 `ActivateDescription` 判断。
- `Bot.Deck` 只表示客户端可见的牌堆槽位，不可直接按其中对象的卡号查询剩余牌组：决斗开始时其中的卡通常为 `Id == 0`，洗牌后也会被重置为 `Id == 0`。因此禁止用 `Bot.Deck.Any(card => card.IsCode(...))` 或等价写法判断某卡是否仍在牌组。
- 需要判断己方牌组中是否还有某卡时，使用 `Bot.HasInDeck(...)` 或 `Bot.GetCardCountInDeck(...)`；这些方法从本地 `.ydk` 初始化计数并随服务器消息维护。Tag Duel 中己方非活动队友的牌组不能查询。
- 牌组执行器的回合、阶段、连锁、召唤尝试和使用次数标志应在 `OnNewTurn`、`OnNewPhase`、`OnChainEnd`、`OnSpSummoning`、`OnMove` 等正确生命周期回调中维护和重置。
- `Duel.CurrentChain`、`CurrentChainInfo`、`ChainTargets`、`LastSummonedCards` 等状态由消息流维护；使用前注意它表示当前客户端已收到的时点，而不是完整规则模拟。

动作和后续选卡通常分两步发生：

- 在接受动作前用 `AI.SelectCard`、`AI.SelectNextCard`、`AI.SelectThirdCard`、`AI.SelectMaterials`、`AI.SelectPlace`、`AI.SelectPosition`、`AI.SelectOption` 等预设后续响应。
- `SelectCard` 是第一个选择；必须先调用它，再调用 `SelectNextCard`/`SelectThirdCard`。
- 选择器会在后续服务端选择消息到达时消费，并可能跨越多个连续选择。不要排入与实际效果流程不一致的额外选择。
- 复杂的、依赖 `hint`/`min`/`max`/候选集合的选择应覆盖 `OnSelectCard` 或素材选择回调；无法处理时返回 `base`/`null`，让通用逻辑继续。
- 返回的卡片数量必须满足 `min`/`max`，并且对象必须来自服务器传入的候选集合；唯一的数量例外是 `cancelable == true` 时可以返回空列表以取消选择。

必发效果的发动和选卡需要特别处理：


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IceYGO/windbot](https://github.com/IceYGO/windbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
