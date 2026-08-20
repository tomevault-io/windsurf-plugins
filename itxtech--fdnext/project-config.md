---
trigger: always_on
description: 本文件是给后续编码代理的仓库工作指南。进入本仓库后，请先阅读本文件，再阅读相关源码和文档。
---

# AGENTS.md

本文件是给后续编码代理的仓库工作指南。进入本仓库后，请先阅读本文件，再阅读相关源码和文档。

## 项目概览

`fdnext` 是面向存储器芯片的一站式解析方案，使用 `pnpm` 和严格 TypeScript monorepo 组织。核心能力包括 PN / typed identifier 解码、iTXTech fdnext DecodePack JSON 规则编译、资源包、HTTP server、CLI、result contract 检查和 FDB / MDB 维护。

主要目录：

- `packages/core`: 解码引擎、公共 SDK、DecodePack JSON 规则 / 编译器、内置 `fdb` / `mdb` / 多语言资源、平台无关 runtime 和输出转换。
- `packages/fdbgen`: 从本地数据集生成 FDB 的工具。
- `packages/server`: HTTP 服务。
- `docs`: iTXTech fdnext DecodePack、集成、FDBGen 和 PN 编码资料。

常用命令：

```bash
pnpm build
pnpm test
pnpm typecheck
pnpm check
pnpm -C packages/core test
pnpm -C packages/core typecheck
pnpm contract:check
```

Engine 生命周期约束：常规 Node、浏览器、Worker isolate 和服务进程都应创建并长期复用一个 `FdnextEngine`，不要为每次 decode/search 创建新 engine。`PreparedCatalog` 仅用于确实需要多个不同配置 engine 时共享不可变资源准备结果，不能把它描述成推荐的逐请求多实例模式。

## 工作习惯

- 开始前先执行 `git status --short`，确认已有未提交修改。不要回退用户或其他代理的改动。
- 搜索文件和文本优先用 `rg` / `rg --files`。
- 小范围手工改文件用 `apply_patch`。
- 文档、源码、测试和提交信息中不得写入本机绝对路径；引用本地资料时最多写文件名，仓库内文件用相对路径。
- 新增或调整规则后，优先补对应产品线测试；测试位置按芯片类型选择，例如 DRAM 用 `packages/core/test/decodepack/dram/<vendor-or-module>.test.ts`，PN / part decode 用 `packages/core/test/decodepack/part-number/<vendor-or-module>.test.ts`。
- DRAM 搜索建议测试默认不跑；只有新增 / 调整 DRAM PN 资源、FBGA marking 或搜索建议相关行为时，额外运行 `pnpm -C packages/core test:dram:search`。如果改动影响 contract SDK 的 part search 输出，也额外运行 `pnpm -C packages/contract-test test:part-search:dram`。
- DRAM 默认拓扑以“厂商规则已识别封装 / topology token”为边界：已确认公开 `package` 默认可继续补单 die，plain DDR 同时补单 CS；如果厂商的 die/CS token 与公开 package 来源不同，规则必须用内部 `meta.dramTopologyTokenRecognized` 显式声明 `true` / `false`。已知 token 但没有可公开的封装尺寸时可设 `true`；未知 token 即使仍能输出其他来源的 `package` 也必须设 `false`，且不得补 `dram_die_count` / `cs_count`。不要用“公开 package 缺失”替代 token 识别判断。
- Micron PN 搜索资源以 `packages/core/resources/mdb.json` 为优先来源。有效 MDB mapping 已包含同一 PN，或在该 PN 后通过 `-`、`:`、空格等 suffix 边界给出更详细的 speed / temperature / status / revision 时，不得再把较短或等价 PN 加入 `dram-pn.json` / `managed-nand-pn.json`。带 `DO NOT USE` 的 MDB 值不算有效覆盖。修改这些资源时必须保持 DRAM 与 managed NAND 的 MDB 去重审计测试通过。
- 新增 SSD 整盘、DIMM / SODIMM / RDIMM、LPCAMM 等模组 decoder 前必须获得用户明确批准，不能从“所有品类”或一般补全任务推断授权。Micron `MTFC` 等芯片级 BGA SSD / managed NAND 可按既有范围维护；不要据此扩展到其他厂商的盘级 SSD 或内存模组。
- 不新增厂商规则、资源或文档，除非用户明确同意该厂商；默认只完善仓库已有厂商和产品线。
- DecodePack / PN 资料完善优先级：第一优先 SK hynix、Samsung、Micron；第二优先 YMTC、CXMT；其他现有厂商只在前两级没有更高价值缺口或属于顺手修复时处理。
- 新增或重命名 canonical field key 时，同步检查 `packages/core/src/field-registry.ts`、`packages/core/resources/lang/eng.json` 和 `packages/core/resources/lang/chs.json`。
- 对 iTXTech fdnext DecodePack JSON 文件保持可读的表驱动结构。不要为了过测试引入一次性特判。

## PN iTXTech fdnext DecodePack 规则约束

PN 解析必须走结构化 token + 规则库，不允许写死完整 PN 白名单。

推荐做法：

- 按前缀、固定长度 token、最长前缀表、组合 key 表来解析。
- 对未知 token 保留已能确定的字段，不应让整条 PN 直接失效。
- `partSpecs.match` 用于识别厂商 / 产品线 / 已知头部结构；非定长或带可扩展尾缀的 PN，不要用完整已知后缀把未知后续 token 排除掉，头部结构符合时应继续命中对应类型并输出已确定字段。
- 官方 ordering 明确定长的 PN 可以在 `match` 里规定 token 长度 / 总长度，但必须是结构化长度和字符类别，不得退化成完整料号字面量或已知 PN 白名单。
- 后续 token 的未知情况应通过 `tokenDecoder` 的 `default`、`takeLongest`、`map`、剩余 `rest` 等结构化步骤自然降级，不能为了完整料号格式把规则写成完整料号特判。
- 规则文件尽量按厂商和芯片 / 产品类型拆分。一个 JSON pack 中最好只放一种芯片或产品线的解析规则，例如 `samsung-ufs-token.json`、`skhynix-emcp-token.json`。
- 新 pack 需要在 `packages/core/src/decodepack/rules/default-rules.ts` 导入并加入 `defaultPartDecodeSpecs`。
- `fields.density` / `fields.dram_density` 继续使用项目既有单位 Mbit，例如 8GB = `65536`。
- `tokenDecoder.assign` 只输出 native draft 路径：`device.*`、`fields.*`、`identifiers.*`、`controllers`、`components`、`meta.*`。用户可见字段使用 canonical snake_case key，例如 `component_density`、`generation_info`、`storage_interface`，不要直接写展示文本。
- `package_code`、`config_code`、`controller_code`、`die_code`、`feature_code` 以及其他 `*_code` token 只用于规则内部解析，不得进入 `fields.*` 或 public result；package / config / controller 等 token 命中后，应优先输出 `package`、`controller`、`controller_revision`、`die_revision`、`process_node`、`special_option` 等语义字段。
- `nand_component`、design ID、product generation code 等纯编码线索也默认只作内部 token；没有稳定可读语义时不要输出给用户。
- 用户可见字段不应重复表达同一语义。一个 token 同时能推导出 canonical 字段和原始/派生描述时，只保留用户最有价值的字段；例如 `dram_speed` 已经输出 `DDR3L-1333 (667MHz)` 时，不要再输出仅重复 `1333Mbps/pin` 的 `speed_grade`。
- 用户可见的数字代际统一使用紧凑 `GenN` 形式，例如 `Gen1`、`Gen2`、`Gen5 Xtacking 4.0`；不得输出 `1st Gen`、`1st generation`、`Gen 1`、`CXMT G3` 等变体。该约束适用于 `generation_info`、`product_generation`、`dram_generation`、把 maturity 表达成代际的 `prod_status`，以及其他公开代际字段值；内部 `generation_code` / token 变量名、`process_node` 中的厂商工艺别名和 `PCIe Gen4`、`USB 3.2 Gen 1` 等标准或专名不受影响。不要增加运行时归一或兼容转换，直接迁移源规则、共享表、测试和文档。
- `Engineering Sample(s)` / `Early Engineering Sample(s)` 这类样品状态只允许通过 `prod_status` 公开一次，不要重复塞进 `product_class`、`sku`、`special_option` 或其他字段。多个 token 同时推导出样品状态时，最终 public result 也只能有一个 Production Status 字段；公开文案应保留资料中的单复数，不要为了审计机械删 `s`。
- `speed_grade` 是例外但必须有额外用户价值：只在原始 speed / grade token 带有 binning、测试等级、CAS/RL/WL 时序、温度等级等 `dram_speed` 未表达的信息时保留，并可附带可读含义，例如 `046BT Fully Tested`、`PG Partial Good Mixed Bins`。如果只是同一速率的另一种单位或 token 回显，应省略。
- `voltage` / `dram_voltage` 只表达电压本身；不要把 DDR 代际、DRAM 类型、产品线等已在其他字段出现的信息重复塞进电压文本。
- `package` 只在官方资料、datasheet、catalog、拆解或可信分销页能确认封装类型、脚位、尺寸或特殊封装信息时输出；公开格式统一为 `TYPE[-PIN][, DIM][, SPECIAL]`，例如 `FBGA-153, 11.5x13x1.0`、`BGA, 11.0x13.0x0.8`、`WLGA`。缺 pin 时只输出已确认的 TYPE，不得补猜脚位；只有 DIM 被确认而 TYPE 未确认时只保留 DIM；不要输出 `mm`、`ball`、`pin` 等单位词或 `Unknown`。只有厂商 package token 时应省略公开 `package`，不要退回输出 package code。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iTXTech/fdnext](https://github.com/iTXTech/fdnext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
