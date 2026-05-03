---
trigger: always_on
description: LCEDA Pro 原理图 API 规范使用规则
---


# LCEDA Pro 原理图 API 规范使用规则

> **适用范围**：使用Cursor AI进行LCEDA Pro原理图自动化设计的所有场景  
> **规则基础**：基于LCEDA Pro官方API文档和源码规范  
> **执行优先级**：在涉及LCEDA Pro API调用时，必须严格遵循本规则

## API基础规范

### API对象访问规范

- **标准访问方式**：必须使用 `const api = window.top.eda1;` 访问API对象
- **上下文检查**：在调用API前，必须确保在LCEDA Pro编辑器环境中
- **禁止行为**：禁止在非LCEDA环境中调用API，会导致运行时错误

### API模块结构理解

- **模块化调用**：必须使用模块化的API调用方式，禁止直接访问内部实现
- **职责边界**：理解各模块的职责边界，禁止跨模块的非法调用
- **接口限制**：只使用公开的接口，禁止假设API的内部实现细节

## 元件库操作规范

### 元件搜索 - lib_Device.search()

#### 参数使用规范

- **keyword参数**：必填，支持中文和英文关键词（如 "ESP32"、"电阻"、"GPS"）
- **libraryUuid参数**：⚠️ **关键规则**：如果传分页参数（itemsOfPage/page），则该参数必选；如果只搜索关键词则可选
- **分页参数规则**：
    - ✅ **正确用法1**：只搜索关键词，不传分页参数
        ```javascript
        const results = await api.lib_Device.search('ESP32');
        ```
    - ✅ **正确用法2**：带分页参数时，必须提供libraryUuid
        ```javascript
        const results = await api.lib_Device.search(
        	'ESP32',
        	api.lib_LibrariesList.extensionUuid, // 必须提供
        	null,
        	null,
        	20,
        	1,
        );
        ```
    - ❌ **错误用法**：带分页参数但未提供libraryUuid（这样查不到任何元件）
        ```javascript
        const results = await api.lib_Device.search('ESP32', null, null, null, 20, 1);
        ```

#### 返回值处理规范

- **必须检查**：调用search()后必须检查返回结果是否为空
- **错误处理**：如果results.length === 0，必须输出错误信息并返回
- **UUID使用**：从返回结果中获取的`uuid`是设备UUID，用于后续的create()操作

### 通过LCSC编号获取元件 - lib_Device.getByLcscIds()

- **批量操作**：优先使用批量获取方式，提高效率
- **参数规范**：lcscIds必须是数组格式，libraryUuid可选

## 原理图操作规范

### 放置原理图元件 - sch_PrimitiveComponent.create()

#### UUID使用规范（⚠️ 最高优先级）

- **必须使用设备UUID**：create()方法必须使用`component.uuid`（设备UUID），**禁止**使用`component.symbolUuid`
- **错误示例**：使用symbolUuid会导致API调用失败
    ```javascript
    // ❌ 错误：使用symbolUuid
    const component = await api.sch_PrimitiveComponent.create(
        { uuid: device.symbolUuid, libraryUuid: device.libraryUuid },
        ...
    );
    ```
- **正确示例**：
    ```javascript
    // ✅ 正确：使用设备UUID
    const component = await api.sch_PrimitiveComponent.create(
        { uuid: device.uuid, libraryUuid: device.libraryUuid },
        ...
    );
    ```

#### subPartName参数规范（⚠️ 必须遵守）

- **必须提供**：subPartName参数必须提供，即使是空字符串`""`
- **参数错位风险**：不提供subPartName会导致后续参数错位，造成严重错误
- **正确用法**：
    ```javascript
    const component = await api.sch_PrimitiveComponent.create(
    	{ uuid: device.uuid, libraryUuid: device.libraryUuid },
    	1000,
    	2000,
    	'',
    	0,
    	false,
    	true,
    	true, // subPartName: ""（必须提供）
    );
    ```

#### 坐标范围检查规范

- **画布大小获取**：在放置元件前，必须获取并解析画布大小
- **坐标验证**：确保所有坐标在画布范围内，严禁超出画布范围
- **实现方式**：
    ```javascript
    const footprintSources = await api.sys_FileManager.getDocumentFootprintSources();
    const { width, height } = parseCanvasSize(footprintSources[0].documentSource);
    const x = Math.max(0, Math.min(width, targetX));
    const y = Math.max(0, Math.min(height, targetY));
    ```

#### done()方法调用规范（⚠️ 必须遵守）

- **必须调用**：创建元件后必须调用`done()`方法，否则刷新后无法看到元件
- **调用时机**：在create()方法返回后立即调用
- **正确用法**：
    ```javascript
    const component = await api.sch_PrimitiveComponent.create(...);
    await component.done();  // ⚠️ 必须调用
    ```

### 获取元件引脚 - sch_PrimitiveComponent.getAllPinsByPrimitiveId()

#### Y轴坐标反转规范（⚠️ 必须遵守）

- **坐标反转**：通过`getAllPinsByPrimitiveId()`获取的坐标，y轴是反的
- **必须处理**：使用引脚坐标时必须对y轴坐标取反
- **正确用法**：
    ```javascript
    const pins = await api.sch_PrimitiveComponent.getAllPinsByPrimitiveId(primitiveId);
    const pinX = pins[0].x;
    const pinY = -pins[0].y; // ⚠️ y轴必须取反
    ```
- **错误用法**：直接使用会导致坐标错误
    ```javascript
    const pinY = pins[0].y; // ❌ 错误！
    ```

#### 引脚查找规范

- **名称匹配**：通过引脚名称查找时，必须使用大小写不敏感的匹配
- **常见引脚名称**：
    - VCC引脚：'VCC'、'VDD'、'POWER'
    - GND引脚：'GND'、'VSS'、'GROUND'

### 创建原理图连线 - sch_PrimitiveWire.create()

#### 坐标数组连续性规范（⚠️ 必须遵守）

- **连续性要求**：坐标数组必须是连续的，多段线彼此无任何连接则创建将会失败
- **正确用法**：

    ```javascript
    // ✅ 正确：连续的两点
    const wire = await api.sch_PrimitiveWire.create([1000, 2000, 3000, 2000], 'VCC');

    // ✅ 正确：连续的折线（L型连接）
    const lWire = await api.sch_PrimitiveWire.create([1000, 2000, 2000, 2000, 2000, 3000, 3000, 3000], 'GND');
    ```

- **错误用法**：
    ```javascript
    // ❌ 错误：不连续的坐标
    const wire = await api.sch_PrimitiveWire.create(
    	[1000, 2000, 3000, 2000, 5000, 4000],
    	'VCC', // 不连续
    );
    ```

#### 网络名称规范

- **标准命名**：使用标准的网络名称（如 "VCC"、"GND"、"VDD_3V3"等）
- **命名一致性**：同一网络的名称必须保持一致

## 文档源码解析规范

### 文档源码获取规范

- **必须使用**：无论何时都不应该截图查看布局，而是应该通过获取文档源码来检索布局
- **获取方法**：
    ```javascript
    const footprintSources = await api.sys_FileManager.getDocumentFootprintSources();
    ```

### 画布大小解析规范

- **格式理解**：文档源码格式为每行`{JSON对象1}||{JSON对象2}`，用`||`分隔
- **画布信息位置**：画布大小信息在`ATTR`类型中，`key`为`"Width"`和`"Height"`
- **解析要求**：
    - 必须从文档源码中解析画布大小
    - 禁止使用硬编码的画布大小值
    - 必须处理解析错误，使用默认值作为后备

### 画布范围检查规范（⚠️ 最高优先级）

- **严禁超出**：所有元件和连线的坐标必须在画布范围内
- **检查时机**：在放置任何元件或创建连线前，必须检查坐标是否在画布范围内
- **实现方式**：
    ```javascript
    const { width, height } = parseCanvasSize(documentSource);
    const x = Math.max(0, Math.min(width, targetX));
    const y = Math.max(0, Math.min(height, targetY));
    ```

## 开发流程规范

### 标准开发流程（必须遵循）

#### 步骤0：画布大小检查（必须）

- **执行时机**：在任何元件操作前
- **执行内容**：
    1. 获取文档源码
    2. 解析画布大小
    3. 规划元件位置（留出边距，建议10%）
    4. 验证坐标在画布范围内

#### 步骤1：元件搜索（必须）

- **执行内容**：
    1. 使用正确的搜索参数
    2. 检查返回结果是否为空
    3. 注意分页参数规则

#### 步骤2：元件放置（必须）

- **执行内容**：

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [np1139059565/pro-schematic-ai-drafty](https://github.com/np1139059565/pro-schematic-ai-drafty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
