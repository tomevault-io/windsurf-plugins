---
trigger: always_on
description: > 行为准则源自 Andrej Karpathy 对 LLM 编码痛点的观察。四条铁律在最前面，技术规范紧随其后。
---

# 串口调试助手 — 项目规则

> 行为准则源自 Andrej Karpathy 对 LLM 编码痛点的观察。四条铁律在最前面，技术规范紧随其后。

## 行为准则（高于一切）

### 1. 动手前先想清楚
**禁止假设。禁止隐藏困惑。暴露所有权衡。**

- 不确定时**显式声明假设**，宁可问清楚也不要猜
- 存在歧义时**列出所有可能的解读**，不要默默选一个
- 有更简单的方案时**直接说出来**，哪怕和当前方向不同
- 发现自己困惑时**停下来**，指出哪里不清楚，请求澄清

### 2. 简洁优先
**刚好解决问题的代码量。不写任何推测性代码。**

- 没有被要求的 features 不写
- 只为一次调用写的抽象不建
- 没被要求的"灵活性"或"可配置性"不加
- 不可能发生的场景不加错误处理
- 200 行能解决的问题不写 1000 行
- **自我审查：** 写完问自己——一个 senior 工程师会不会觉得这里复杂过头了？会就改。

### 3. 精准修改
**只改必须改的。只清理自己的垃圾。**

改已有代码时：
- 不改相邻代码的风格、注释、格式
- 不重构没坏的东西
- 匹配已有风格，哪怕不是你喜欢的
- 发现无关的死代码，**提一嘴**——但不动手删

清理时：
- 只删除**你的改动**导致不再使用的 import/变量/函数
- 不动原有代码中的死代码，除非被要求

**测试标准：** 每一行改动都能直接追溯到用户的需求。

### 4. 目标驱动
**定义成功标准。循环直到验证通过。**

| 用户说…… | 转化为…… |
|----------|---------|
| "加个校验" | "先写非法输入的测试，跑通了再实现" |
| "修这个 bug" | "先写一个能复现的测试，跑挂了再修" |
| "重构 X" | "确保重构前后测试全部通过" |

多步任务的计划格式：
```
1. [步骤] → 验证: [检查点]
2. [步骤] → 验证: [检查点]
3. [步骤] → 验证: [检查点]
```

强成功标准让 Agent 能自主循环。弱标准（"把它弄好"）只会让你反复澄清。

---

## 技术栈

| 项 | 选型 |
|---|------|
| 语言 | C++17 |
| 框架 | Qt 6.11.0 |
| 编译器 | MSVC 2022 64-bit (主力) / MinGW 13.1.0 64-bit (备选) |
| 构建 | CMake 3.28+ |
| 串口 | QSerialPort (Qt 模块) |
| 图表 | QChart (实时波形) — QPainter 手动绘制作为高性能备选 |
| 脚本 | QJSEngine (JS 引擎，用于宏/脚本) |
| 配置存储 | JSON (QJsonDocument + QSaveFile，跟随 exe 目录) |
| 主题 | QSS 暗色主题，扁平化现代风格 |
| 测试 | GoogleTest |
| 打包 | windeployqt → NSIS/Inno Setup |

## 目录结构

```
project/
├── CMakeLists.txt
├── src/
│   ├── main.cpp
│   ├── app/
│   │   ├── Application.h/cpp       # QApplication 子类，全局初始化
│   │   └── ThemeManager.h/cpp      # 暗色主题 QSS 管理
│   ├── core/
│   │   ├── SerialPort.h/cpp        # QSerialPort 封装，串口线程
│   │   ├── ProtocolParser.h/cpp    # 协议解析器基类 + 注册工厂
│   │   ├── MacroEngine.h/cpp       # QJSEngine 封装，脚本执行
│   │   └── DataLogger.h/cpp        # 数据日志读写
│   ├── ui/
│   │   ├── MainWindow.h/cpp        # 主窗口
│   │   ├── TabManager.h/cpp        # 多串口 Tab 管理 (QTabWidget)
│   │   ├── SessionView.h/cpp       # 单个串口会话视图
│   │   ├── ReceiveView.h/cpp       # 接收区 (QPlainTextEdit)
│   │   ├── SendView.h/cpp          # 发送区 + 快捷按钮列表
│   │   ├── WaveformView.h/cpp      # 波形绘制窗口
│   │   ├── ProtocolView.h/cpp      # 协议解析面板
│   │   ├── MacroEditView.h/cpp     # 脚本编辑器
│   │   └── StatusBar.h/cpp         # 状态栏统计
│   └── util/
│       ├── HexUtils.h/cpp          # HEX/ASCII 互转工具
│       └── Constants.h             # 全局常量
├── res/
│   ├── theme/
│   │   ├── dark.qss
│   │   └── light.qss
│   └── icons/
├── scripts/                          # 用户 JS 脚本目录
│   └── example.js                    # 示例脚本
├── tests/
│   ├── CMakeLists.txt
│   ├── test_hex_utils.cpp
│   ├── test_protocol_parser.cpp
│   └── test_serial_port.cpp
└── docs/
    └── architecture.md
```

## 文件约束

- 每文件 ≤ 800 行，超过则拆分
- 每函数 ≤ 50 行，超过则提取
- 嵌套 ≤ 4 层，超过则 early return
- 头文件用 `#pragma once`
- 禁止在头文件中 `using namespace`

## 编码风格

### 花括号与缩进

```cpp
// Allman 风格 — 花括号独立一行
void foo()
{
    if (condition)
    {
        doSomething();
    }
    else
    {
        doOther();
    }
}
```

- 缩进：4 空格，禁止 Tab
- 行尾不留空白
- 文件末尾保留一个空行

### include 顺序

```cpp
// 1. 自身头文件
#include "SerialPort.h"

// 2. Qt 头文件
#include <QSerialPort>
#include <QTimer>

// 3. 项目内部
#include "core/DataLogger.h"
#include "util/HexUtils.h"

// 4. 标准库
#include <memory>
#include <vector>
```

每组之间空一行。用 `""` 还是 `<>` 按 Qt 惯例。

### 类声明模板

```cpp
#pragma once

#include <QObject>
#include <QSerialPort>

class SerialPort : public QObject
{
    Q_OBJECT

public:
    explicit SerialPort(QObject *parent = nullptr);
    ~SerialPort() override;

    // 禁止拷贝
    SerialPort(const SerialPort &) = delete;
    SerialPort &operator=(const SerialPort &) = delete;

    bool openPort(const QString &name, qint32 baudRate, QSerialPort::DataBits dataBits,
                 QSerialPort::StopBits stopBits, QSerialPort::Parity parity,
                 QSerialPort::FlowControl flowControl);
    void closePort();
    bool isOpen() const;

    qint64 sendData(const QByteArray &data);

signals:
    void dataReceived(const ReceivedFrame &frame);
    void errorOccurred(const QString &message);
    void portOpened();
    void portClosed();

private slots:
    void onReadyRead();
    void onErrorOccurred(QSerialPort::SerialPortError error);

private:
    QSerialPort *m_serialPort = nullptr;
    QByteArray m_readBuffer;
};
```

要点：
- 成员变量 `m_` 前缀，指针成员在声明处初始化为 `nullptr`
- `explicit` 标记单参数构造函数
- 禁止拷贝（Qt 对象本来就不可拷贝，显式 delete 更清晰）
- signals/public/private 分区明确
- 析构函数 `override` 而非 `virtual`（C++17）

## 命名约定

| 类型 | 风格 | 示例 |
|------|------|------|
| 类/结构体 | PascalCase | `SerialPort`, `ReceiveView` |
| 函数/方法 | camelCase | `openPort()`, `sendData()` |
| 成员变量 | camelCase + `m_` 前缀 | `m_serialPort`, `m_receiveBuffer` |
| 静态/全局常量 | UPPER_SNAKE_CASE | `MAX_BUFFER_SIZE`, `DEFAULT_BAUD_RATE` |
| 局部变量 | camelCase | `portName`, `baudRate` |
| bool 变量 | `is`/`has`/`can`/`should` 前缀 | `isConnected`, `canSend`, `hasData` |
| 信号 | 动词过去式 | `dataReceived()`, `portOpened()` |
| 槽 | 动词原形 | `onSendClicked()`, `handleData()` |

## 信号槽规则

- 禁止用 `SIGNAL()`/`SLOT()` 宏（字符串匹配，编译期不检查）
- 一律用 `connect(ptr, &Class::signal, ptr, &Class::slot)` 新式语法
- 跨线程对象用 `connect` 时必须传 `Qt::QueuedConnection`
- 高频率信号（接收数据）用 `Qt::QueuedConnection` 避免阻塞串口线程
- lambda 连接中捕获 `this` 确保对象生命周期长于连接

## 串口线程模型

```
┌──────────────┐    信号槽    ┌──────────────┐
│  主线程 (UI)  │◄────────────│  串口工作线程   │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentthink/serial-helper](https://github.com/agentthink/serial-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
