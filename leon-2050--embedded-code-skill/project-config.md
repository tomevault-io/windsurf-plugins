---
trigger: always_on
description: 嵌入式 C 的 REWRITE/REVIEW/GUIDE：三层整理、低层审查、RTOS/构建咨询。硬件参数须有出处。
---


# Embedded C 代码助手 Skill

---

## 1. 定位与使用原则

### 1.1 定位

帮助处理嵌入式 C 代码：旧代码整理（**REWRITE**）、低层固件审查（**REVIEW**）、RTOS/构建/调试指导。

本 skill 提供不绑定 IDE/agent 的保守编码规范。寄存器偏移、位定义、IRQ、屏障、cache/DMA、时序等须来自手册、厂商头文件或仓库——**不编造硬件事实**。

### 1.2 使用原则

1. 先判 `REWRITE`/`REVIEW`/`GUIDE`（§1.4）→ 读仓库头文件、命名、SDK、已有驱动样例
2. 遵守 §4 三层架构（reg → drv → app；ISR 回调通知 app）
3. 项目规范优先；CMSIS/厂商结构体直接复用，不为 fallback 再包一层
4. 硬件信息缺口先列出；待定值标 placeholder
5. 输出优先 patch；风格让位于 correctness/并发/硬件风险

### 1.3 前置信息

| 信息 | 要求 | 示例 |
|------|------|------|
| 外设/模块名 | REWRITE 必需 | `uart`, `spi`, `gpio`, `dma` |
| 硬件来源 | 强烈建议 | 参考手册章节、厂商头文件、现有驱动 |
| 芯片或架构 | 强烈建议 | `STM32F4`, `Cortex-M4`, `ESP32` |
| 基地址/位定义 | 生产代码必需 | `UART_BASE_ADDR = 0x4000C000U` |
| 项目约定 | 重写前读取 | status type、命名、SDK、build macros |
| RTOS（如适用） | 驱动层需确认 | FreeRTOS、Zephyr、RT-Thread、裸机 |

缺口先列出；待定值标 `USER_PROVIDED` / `REPO_DERIVED` / `PLACEHOLDER`。

### 1.4 工作模式

**REWRITE**：保留 public API、ABI、寄存器写入顺序与时序序列；按 §4 整理类型/命名/分层。输出：简述 → 缺口 → patch（必要时文件布局）。workaround 标 `/* 有意保留：原因 */`。

**REVIEW**：不产出代码。按寄存器抽象 → 分层/ISR/同步 → volatile/barrier/cache/DMA → 错误处理/内存 顺序查。输出表：`| P0/P1/P2 | 位置 | 问题 | 建议 |`（P0 行为/安全，P1 并发/可移植，P2 风格）。

**GUIDE**：无代码整理/审查需求（RTOS 任务设计、CMake/链接脚本、调试策略等）。按 §5–§9 给建议或示例片段，不走 REVIEW 表。

### 1.5 RED LINES

禁伪造硬件参数；禁低层 `malloc`/VLA；禁公共接口用 `int`/`char`/`long`；禁业务代码裸寄存器地址；禁不可编译输出；**禁违反三层解耦（reg/drv/app 分层，ISR 回调通知 app）**。

---

## 2. Fallback 编码规范

仓库无更强约定时适用；符合则沿用，不符合则不改逻辑地统一。

### 2.1 命名

| 元素 | 规范 | 示例 |
|------|------|------|
| 变量 | `snake_case` | `rx_count` |
| 全局变量 | `g_snake_case` | `g_system_ticks` |
| 函数 | `camelCase` | `uartInit()` |
| 结构体/枚举类型 | `snake_case_t` | `uart_handle_t` |
| 枚举值 | `PREFIXED_SNAKE` | `UART_STATE_IDLE` |
| 常量/宏 | `SCREAMING_SNAKE` | `UART_SR_RX_READY_MASK` |
| 指针 | 项目无约定时用清晰语义名 | `rx_buffer`, `handle`；或 `p_rx_buffer` |

### 2.2 类型与错误处理

- 公共接口优先使用 `<stdint.h>`、`<stdbool.h>`、`<stddef.h>`；默认 `uint8_t` / `uint16_t` / `uint32_t`、`int32_t`、`bool`
- 不把 `int`、`char`、`long` 作为默认跨平台接口类型

项目没有既有 status 类型时，公共函数默认返回 `embedded_code_status_t`：

```c
typedef enum {
    EmbedCode_Ok           =  0,
    EmbedCode_ErrNullPtr   = -1,
    EmbedCode_ErrInvalidArg = -2,
    EmbedCode_ErrTimeout   = -3,
    EmbedCode_ErrBusy      = -4,
    EmbedCode_ErrNotInit   = -5,
} embedded_code_status_t;

#define VALIDATE_NOT_NULL(ptr) \
    do { if ((ptr) == NULL) return EmbedCode_ErrNullPtr; } while (0)

#define VALIDATE_INIT(handle) \
    do { if (!(handle) || !(handle)->initialized) return EmbedCode_ErrNotInit; } while (0)
```

### 2.3 数据结构

重写或修改时：状态/错误/标志 → `enum`；配置/上下文 → `struct`；>3 标量参数 → 结构体指针；位域用 `MASK` 宏，禁裸魔数。

### 2.3.1 结构体模式

默认拆成配置、运行时句柄和状态：

```c
typedef struct {
    uint32_t base_address;
    uint32_t baud_rate;
} uart_config_t;

typedef struct {
    bool            initialized;
    volatile bool   tx_busy;
    uint8_t        *rx_buffer;
    uint16_t        rx_head;
    uint16_t        rx_tail;
    uart_config_t   config;
} uart_handle_t;

typedef enum {
    UART_STATE_IDLE = 0,
    UART_STATE_TX_BUSY,
    UART_STATE_RX_ACTIVE,
    UART_STATE_ERROR,
} uart_state_t;
```

### 2.4 注释

#### 2.4.1 注释原则

- 注释语言遵循项目约定；项目无约定时可用中文或用户指定语言。
- 注释解释硬件原因、约束、时序和意图；不要逐行复述代码。
- 注释应回答"为什么"而非"是什么"——代码本身说明"是什么"，注释补充"为什么这样写"。
- 保持注释与代码同步更新；过时注释比无注释更有害。
- **寄存器层注释要详细**：每个寄存器必须标注偏移地址 `/* 0xNN */`、位域含义 `bit[n]=功能`、读写属性。底层代码是硬件交互的唯一依据，注释不够详细会导致后续维护者误操作。
- **应用层注释侧重意图**：说明业务目的、调用约束、错误处理策略，不需要重复底层已有的位域信息。

#### 2.4.2 必须注释的场景

| 场景 | 说明 | 示例 |
|------|------|------|
| **操作步骤** | 驱动层和应用层函数体内的关键步骤必须用编号注释标注操作顺序 | 见下方示例 |
| 硬件约束 | 寄存器写入顺序、时序要求、errata workaround | `/* 必须先写 CTRL 再写 DATA，否则 FIFO 错位（Errata 3.2） */` |
| 非显而易见的逻辑 | 算法选择原因、特殊边界处理 | `/* 使用查表法而非计算，节省 12 个 CPU 周期 */` |
| 临时方案 | workaround、TODO、待确认项 | `/* FIXME: 临时绕过芯片 Rev.A 的 DMA 冻结问题 */` |
| 安全关键路径 | 看门狗刷新点、冗余检查、故障注入点 | `/* 喂狗必须在 SPI 传输完成后，否则超时复位 */` |
| 并发/中断相关 | critical section、屏障、volatile 使用原因 | `/* 关中断保护 tx_tail，ISR 中会修改 */` |
| 魔数来源 | 非自明的常量值来源 | `/* 1200 = 72MHz / (16 * 3750)，参考手册 §23.4.2 */` |

**操作步骤注释要求（驱动层 + 应用层均适用）：**

驱动层函数和应用层函数体内，每个关键操作步骤必须用编号注释标注，让读者不用看手册也能理解操作流程：

```c
embedded_code_status_t uartDrvInit(uart_drv_handle_t *p_handle, const uart_config_t *p_config)
{
    /* Step 1: 参数校验 */
    VALIDATE_NOT_NULL(p_handle);
    VALIDATE_NOT_NULL(p_config);

    /* Step 2: 禁用 UART，避免配置过程中产生意外中断 */
    UART_REG->ctrl.CTRL &= ~UART_CTRL_ENABLE_MASK;

    /* Step 3: 配置波特率（值来自参考手册 §23.4.2 公式） */
    UART_REG->ctrl.BAUD = calcBaudDiv(p_config->baud_rate);

    /* Step 4: 清除挂起状态和 FIFO 残留数据 */
    UART_REG->status.STATUS = 0U;

    /* Step 5: 使能外设，标记初始化完成 */
    UART_REG->ctrl.CTRL |= UART_CTRL_ENABLE_MASK;
    p_handle->initialized = true;
    return EmbedCode_Ok;
}
```

应用层同理：`Step 1: 参数校验` → `Step 2: 填充驱动配置` → `Step 3: 调用驱动层` → `Step 4: 初始化缓冲区和状态`。

#### 2.4.3 注释格式

**函数注释**（公共 API 必须有，静态辅助函数酌情）：

```c
/**
 * @brief  初始化 UART 并配置波特率
 * @param  handle  句柄指针，调用前需填充 config
 * @param  baud    目标波特率
 * @return EmbedCode_Ok 成功；EmbedCode_ErrNullPtr / EmbedCode_ErrInvalidArg
 * @note   调用前需确保 GPIO 已配置为 AF 模式
 */
embedded_code_status_t uartInit(uart_handle_t *handle, uint32_t baud);
```

**内联注释**：关键行上方或同行，解释原因而非复述代码。

```c
uint32_t timeout = UART_TX_TIMEOUT_MS;
while (!(UART_REG->status.STATUS & UART_STATUS_TX_EMPTY) && --timeout) {
    /* 等待发送完成，超时防硬件死锁 */
}
```

**结构体/寄存器注释**：字段含义非自明时才加，寄存器字段标注偏移和位域。

```c
typedef struct {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leon-2050/embedded-code-skill](https://github.com/leon-2050/embedded-code-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
