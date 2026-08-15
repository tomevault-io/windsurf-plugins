---
trigger: always_on
description: - 本仓库只维护统一入口、固定 submodule gitlinks、资产 manifest、环境说明、bootstrap、quickstarts 和发布文档。
---

# ApexOracle super-repo 维护约束

- 本仓库只维护统一入口、固定 submodule gitlinks、资产 manifest、环境说明、bootstrap、quickstarts 和发布文档。
- 不复制 Core、DLM-Pretraining、MDLM、Evo-2 或 Generation 的实现代码。
- `.gitmodules` 只能加入已经存在、可公开 clone 且通过 module-level 验收的 repository；禁止加入浮动或失效 URL。
- 每个 active gitlink 必须在 `manifests/modules.lock.yaml` 记录完整 40-character commit，并由
  `python scripts/check_module_locks.py` 验证。
- checkpoint、embedding、dataset、raw output、cache 和 private assay data 不进入 Git；只在 asset manifest
  登记 URI、revision、SHA-256、许可和发布状态。
- 现有 ApexOracle legacy tree 已由 branch `legacy-monorepo` 与 annotated tag
  `legacy-monorepo-snapshot-2026-08-10` 保存。不得删除或移动这两个远程恢复点。
- 最终 Core 直接复用当前 `DragonDescentZerotsu/Synergy` repository，并在完整 history audit 后重命名为
  `DragonDescentZerotsu/ApexOracle-Core`；不得建立第二份 Core repository。
- 当前发布阶段和剩余 gate 记录在 `docs/RELEASE_STATUS.md`；每次新增 module gitlink 或资产时必须同步更新。
- `docs/RELEASE_PROVENANCE.md` 必须区分 release gitlinks、科学实现验收 commits、恢复 refs 和外部资产
  revisions；文档-only module commit 不得被误写为重新验证过的科学实现。
- 论文 Code availability 已固定 Zenodo embedding dataset DOI `10.5281/zenodo.15612048`；README、
  `CITATION.cff`、`manifests/data_assets.yaml` 与 release provenance 必须保持一致，不得再写成没有 Zenodo record。
- `v0.2.3` 只收口 downstream reporting/candidate scorer 的误导命名：canonical profile 为
  `fixed_epsilon_non_pad`，本地资产固定 `t=1e-3`，不是精确 clean `t=0`。Core/MDLM/Generation gitlinks
  分别固定到 `1973d2d3cc6b27202a3960c363c207dd030f74e7`、
  `931e3dc09bfc2501809c03dbd016741406950f5f`、`67b593e1a623af3af80c64e263bde527d73d89ef`；checkpoint
  bytes/SHA 与 Generation sampler 权重均未改变。
- **2026-08-10 post-`v0.2.3` reviewer reproducibility batch：** Core `main` 已新增 compact paper strain
  mapping，并在后续 paper genome list batch 推进到 commit
  `1ab309c3275f21e3f4e7346e8d0340894a7507cc`；root `manifests/data_assets.yaml`
  必须登记其 730,151-byte file、SHA-256 `51db55fe...d8f4` 和 1,766 labels/1,769 routes/92,322 routed rows
  scope。Reviewer code/data 回复的 canonical working draft 为 `docs/REVIEWER_CODE_RESPONSE_DRAFT.md`；其中
  `DONE` 与 `OPEN` 必须按 public immutable asset 分开，禁止把 prediction capsule、exact runtime/RAM/VRAM
  或未发布 model-ready tables 提前写成完成。全 checkpoint 上传不是 release gate；固定政策见
  `docs/REPRODUCIBILITY_SCOPE.md`。
- README 只允许使用从 legacy history 恢复并由 SHA-256 固定的 `assets/ApexOracle_1.png` 与
  `assets/upenn.png` 两个视觉资产；其他 root binary/data 文件仍由 `python scripts/check_release_tree.py`
  拒绝，不能借 README 美化放宽发布边界。
- 发布前运行 `python scripts/check_release_tree.py`、`python scripts/check_module_locks.py` 和
  `python scripts/check_repository_bloat.py`、`python -m pytest -q`；四个入口均通过后才允许更新默认分支。
- Repository anti-bloat policy 固定在 `manifests/repository_size_policy.json`，解释与当前基线见
  `docs/REPOSITORY_HYGIENE.md`。任何新 tracked file 默认不得超过 1 MiB；只有精确路径、窄 size cap 和
  明确科学理由的 allowlist 才允许例外。六棵 active trees 中任意 >=20 KiB exact duplicate、checkpoint/cache
  suffix、generated cache/build path、repo file-count/total-byte 超限都会使 CI 失败。Paper model-ready tables、
  sample predictions、embeddings 和 checkpoints 必须外置到 Zenodo/Hugging Face，Git 只保留 compact manifest、
  exporter、split IDs、hash 和 recomputation code。
- **2026-08-11 六仓库文件系统复核：** anti-bloat schema v2 额外拒绝同一仓库内 >=1 KiB exact duplicate 和
  未登记的顶层目录，并报告每个顶层路径的 file/byte 分布、>=500-line source review candidates 与 80% soft
  limit alerts。当前六棵发布树均无 nonignored untracked file、无仓库内重复；root/Core/MDLM file-count 与 Evo2
  bytes 进入软警戒但未越界，处理原则固定在 `docs/REPOSITORY_HYGIENE.md`。DLM-Pretraining 与 downstream
  MDLM 之间少量相同 upstream runtime 文件只作信息报告，因两者必须独立 clone/install，不得为表面去重建立
  cross-submodule import。Core 的 16 个 tracked experiment directories 均有 README；本机旧 producer 中的
  unfinished reviewer files 不得混入 clean public gitlink。该复核同时发现 MDLM fixed-epsilon scorer 改名
  manifest 未进入 generated lineage ledger；MDLM `26e414b` 已重建四份 ledger outputs，176/176 tracked
  code/config assets 覆盖且 118 tests 通过，不涉及模型或接口变更。
- **2026-08-11 发布后维护复核：** Generation `706e06f` 只修正 README 中两个仍指向旧
  `guidance_eval/` 位置的 shell-script 链接，实际脚本始终位于 `scripts/`；15 tests 与 module release checker
  通过，未改 sampler/config/API。Root 恢复 `manifests/model_ready_capsule_sources.json` 为 Zenodo v5 发布时的
  精确构建输入，tokenizer 前 121,265、超长排除 310 的解释只保留在审计计划 manifest，避免当前 builder
  生成与公开 archive hash 不同的 payload。Super-repo current Generation lock 同步为 `706e06f`。
- **2026-08-11 Providencia screening 维护：** Core `bbaaedf` 发布 ATCC 29914 exact-asset、screening 与
  generation capsule，并复用 canonical Evo-2/strain-text producers；MDLM `548f65c` 将 peptide inventory
  prepare/reporting 收敛为 supplier/strain-neutral API/CLI，修复 `.pt` sidecar、blank peptide、token limit 与
  condition provenance，同时把 genome scale `1e14` 变为显式 manifest contract。Core 221 passed / 4 skipped、
  MDLM 127 tests、module ledger 179 assets 与 quickstart 单测通过。Root 只推进两个 gitlink/locks；MIC 与
  Generation quickstart 脚本、HF revisions、checkpoint/tensor bytes 和协议均未改变。远端 recursive fresh
  clone 已展开五个精确 commits，module-lock/release-tree/anti-bloat gates、17 root tests 与 2 个 Core MIC
  quickstart tests 全部通过。
- **2026-08-11 Core 本机维护边界：** 日常 Core 开发只在原 `Synergy` 工作区进行；它对应唯一的公开
  `DragonDescentZerotsu/ApexOracle-Core` repository。Super-repo 只长期保存 `modules/core` 的 gitlink、
  `.gitmodules` URL 和 lock commit，不保留第二份长期 Core checkout，也不得用复制目录或文件系统 symlink
  替代 gitlink。`git submodule deinit modules/core` 后 `git submodule status` 的前导 `-` 只表示本机未展开

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DragonDescentZerotsu/ApexOracle](https://github.com/DragonDescentZerotsu/ApexOracle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-15 -->
