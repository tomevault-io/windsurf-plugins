---
trigger: always_on
description: > 本文件为 AI 编码代理（及新加入的开发者）提供在 hikyuu 仓库中工作所需的核心上下文：
---

# AGENTS.md — Hikyuu 项目 AI 开发指南

> 本文件为 AI 编码代理（及新加入的开发者）提供在 hikyuu 仓库中工作所需的核心上下文：
> 项目结构、构建/测试命令、代码规范与常见注意事项。**先读本文件，再动手改代码。**

## 1. 项目概览

- **Hikyuu** 是一个基于 **C++/Python** 的开源超高速量化交易研究框架，聚焦策略分析、回测与实盘能力扩展（深度适配国内 A 股市场）。
- 核心能力：交易模型研发、极速计算引擎、高效回测体系、实盘交易拓展。
- 项目组成：**高性能 C++ 核心库**（`hikyuu_cpp`）+ **pybind11 绑定层**（`hikyuu_pywrap`）+ **Python 接口层**（`hikyuu` 包）+ **交互式探索工具**（`hikyuu.interactive`）。
- 开源协议：Apache License 2.0；当前版本 2.8.x；默认分支 `master`，另有 `release`、`bugfix`、`feature/*` 分支。
- 项目文档：[https://hikyuu.readthedocs.io/zh-cn/latest/index.html](https://hikyuu.readthedocs.io/zh-cn/latest/index.html)（Sphinx，中文为主）。

## 2. 仓库结构

```
hikyuu/
├── xmake.lua                 # 顶层构建脚本（xmake 工程，定义全局 options/依赖）
├── copy_dependents.lua       # 拷贝第三方依赖头文件/库的任务
├── requirements.txt          # Python 侧依赖
├── setup.py / sub_setup.py   # Python 包安装脚本
├── hikyuu_cpp/               # C++ 核心引擎库
│   ├── hikyuu/               #   ├─ 核心代码：Stock/KData/Indicator/StockManager...
│   │   ├── data_driver/      #   ├─ 数据驱动引擎（HDF5/MySQL/SQLite/TDX）
│   │   ├── trade_sys/        #   ├─ 交易系统组件（SG/MM/ST/CN/EV/PG/SP/PF/SE/AF/MF）
│   │   ├── trade_manage/     #   ├─ 交易管理（TradeManager/OrderBroker）
│   │   ├── indicator/        #   ├─ 指标库（含 indicator_talib）
│   │   ├── strategy/         #   ├─ 策略上下文
│   │   ├── factor/           #   ├─ 多因子
│   │   ├── utilities/        #   ├─ 基础工具（日志/配置/网络等）
│   │   └── xmake.lua         #   └─ 核心 target("hikyuu") 定义
│   ├── unit_test/            # C++ 单元测试（doctest），targets: unit-test / small-test / real-test
│   └── demo/                 # C++ 示例
├── hikyuu_pywrap/            # pybind11 绑定（target "core" → core.so / core.pyd）
│   ├── main.cpp              # 绑定注册入口
│   ├── indicator/ trade_sys/ trade_manage/ data_driver/ ...   # 各模块绑定
│   └── xmake.lua
├── hikyuu/                   # Python 接口包
│   ├── __init__.py           # 包入口：加载编译产物 core.so 及依赖库
│   ├── core.py / extend.py   # 核心对象导入与扩展
│   ├── indicator/            # Python 侧指标扩展（.pyi 存根与实现）
│   ├── trade_sys/ trade_manage/ analysis/ data/ draw/ util/
│   ├── fetcher/ gui/ strategy/ interactive.py / hub.py
│   ├── plugin/               # 运行时插件（数据导入、行情等）
│   ├── cpp/                  # 编译产物目录：core310~core313.so、lib*.dylib 等（gitignore）
│   ├── test/                 # Python 测试（test.py 为入口）
│   └── examples/             # 示例与 notebook 教程
├── docs/                     # Sphinx 文档（docs/source，中文；docs/make.sh 构建）
├── test_data/                # C++ 测试数据（运行测试时自动拷贝）
├── i18n/                     # 国际化/语言文件
├── docker/                   # 容器化配置
└── .github/workflows/        # CI：ubuntu.yml / windows.yml / macosx.yml
```

## 3. 构建系统（xmake）

- 构建工具：**xmake**（顶层 `set_xmakever("3.0.0")`，CI 用 3.0.8）。C++ 标准 **C++20**；Windows 用 clang-cl。
- 第三方依赖全部通过 xmake 包管理（`add_requires`）拉取：boost、hdf5、mysql、fmt、spdlog、sqlite3、flatbuffers、nng、nlohmann_json、eigen、xxhash、utf8proc、ta-lib、mimalloc、pybind11、doctest 等；外部仓库 `hikyuu-extern-libs`（github/gitee）。
- 关键配置项（`xmake f` 选项）：`mysql`、`hdf5`、`sqlite`、`tdx`、`ta_lib`、`low_precision`、`omp`、`serialize`、`leak_check`、`stacktrace`、`log_level`、`async_log`、`feedback`、`spend_time` 等。
- 产物输出到 `build/{mode}/{plat}/{arch}/lib`；Python 包运行所需的 `core.so` 与依赖库需拷贝到 `hikyuu/cpp/`（见下方工作流）。

### 常用命令

```bash
# 配置（首次或变更依赖/选项后）
xmake f -k shared -y -vD

# 编译 C++ 核心库
xmake -b core

# 编译并运行 C++ 单元测试（doctest，small-test 不依赖真实数据）
xmake r small-test

# 执行完整单测（含 indicator/trade_sys 等大部分模块）
xmake r unit-test

# 真实数据测试（需 HKU_USE_REAL_DATA_TEST 与真实行情数据，通常只在 CI/本地有数据时跑）
xmake r real-test

# 调试/覆盖率模式
xmake f -m debug -y          # debug
xmake f -m coverage -y       # coverage（生成 lcov/genhtml 报告）
```

> 注意：运行 `xmake r` 系列测试时，构建系统会自动把 `test_data`、`hikyuu/plugin`、`i18n` 拷贝到可执行文件所在目录（见 `hikyuu_cpp/unit_test/xmake.lua` 的 `prepare_run`）。

## 4. 测试

### Python 测试（hikyuu/test/）

```bash
export PYTHONPATH=.
python3 hikyuu/test/test.py     # CI 使用的入口
```

- 各模块独立测试文件：`Indicator.py`、`KData.py`、`Signal.py`、`MoneyManager.py`、`Stoploss.py`、`AllocateFunds.py`、`Datetime.py`、`Parameter.py` 等，可单独运行（如 `python3 hikyuu/test/Indicator.py`）。
- 新增 Python 功能应在 `hikyuu/test/` 下补充对应测试。

### C++ 测试（hikyuu_cpp/unit_test/）

基于 **doctest**，目录结构与核心库模块一一对应。测试工程组织须遵循以下原则：

#### 组织原则

1. **物理隔离，结构并行**：测试工程与源代码工程物理隔离，使用完全独立的并行目录（`hikyuu_cpp/unit_test/hikyuu/…` 对 `hikyuu_cpp/hikyuu/…`），内部目录结构保持一致。
2. **一模块一套件**：针对一个模块（通常为一个类），建立一个测试套件（test_suite），命名规则为 `test_模块名_suite`，例如 `test_iniparser_suite`，全部使用小写字母。测试套件在文件顶部用 `@defgroup` / `@ingroup` 声明，见 `test_iniparser.cpp`、`test_Stock.cpp` 等现有文件。
3. **一套件一文件**：每个测试套件使用一个独立测试文件，文件命名规则为 `test_模块名.cpp`，例如 `test_iniparser.cpp`、`test_Stock.cpp`。
4. **一函数/方法一用例**：针对每一个函数或类成员方法，建立一个独立的测试用例（`TEST_CASE`），命名规则为 `test_函数名` 或 `test_类名_方法名`。重名时可在其后加 `_case` 或其他标识进行区分。
5. **公开接口尽可能覆盖**：公开接口应尽可能添加测试；对于必须 mock 才能模拟的场景可不考虑。
6. **@arg 标注测试点**：在每个测试用例内，用 `/** @arg … */` 注释明确标注每一个测试点，便于代码审查与快速定位。示例：

```cpp
TEST_CASE("test_IniParser_hasSection") {
    IniParser ini_parser;
    // …准备数据…
    /** @arg 存在指定的 section */
    CHECK_UNARY(ini_parser.hasSection("test1"));
    /** @arg 不存在指定 section */
    CHECK_UNARY(!ini_parser.hasSection("test2"));
}
```

7. **边界条件必须覆盖**：每个函数/方法的测试中，边界条件必须覆盖，尤其注意：

   - **循环边界**：0 次、1 次、恰好 N 次、N-1、N+1 次迭代的情况（例如空容器、单元素、多元素）。
   - **极值边界**：最小值/最大值、空字符串、空范围、`Null<T>()`、越界索引、零值、负值（若允许）。
   - **分支边界**：`if/else`、`switch` 的每个分支，三元表达式两侧，提前 `return` / `break` / `continue` 的路径。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fasiondog/hikyuu](https://github.com/fasiondog/hikyuu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
