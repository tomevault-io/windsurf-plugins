---
trigger: always_on
description: 你熟悉 uni-app x框架，擅长编写跨平台且高性能的代码。
---

# Memory Bank

你熟悉 uni-app x框架，擅长编写跨平台且高性能的代码。
uni-app x项目使用UTS语言编写script。 UTS是一种跨平台的强类型语言，类似TS语言但类型要求更加严格。

## Code Style and Structure
    - 简洁易懂，复杂的代码配上中文注释。
    - 严格类型匹配，不使用隐式转换。
    - 不使用变量和函数的声明提升，严格的在清晰的范围内使用变量和函数。
    - 当生成某个平台专用代码时，应使用条件编译进行平台约束，避免干扰其他平台。

## project
    - 遵循uni-app x的项目结构，在正确的目录中放置生成的文件。

## page
    - 使用uvue作为页面后缀名，uvue与vue基本类似，但有少量细节差异。
    - 生成的uvue页面放置在项目的pages目录下，生成的页面需要在pages.json中注册。
    - 可滚动内容必须在scroll-view、list-view、waterflow等滚动容器中。如果页面需要滚动，则在页面template的一级子节点放置滚动容器，例如` <scroll-view style="flex:1">`。此时应在 App 上使用条件编译，例如: `<!-- #ifdef APP --><scroll-view class="container"><!-- #endif -->`
    - 生成uvue页面时，页面内容需符合uts.mdc、uvue.mdc、ucss.mdc、api.mdc约定的规范。

# API
    - 可以使用uts的api，但注意版本和平台的兼容性。
    - 可以使用uni-app x的api，但注意版本和平台的兼容性。
    - 可以使用vue3的api，但注意版本和平台的兼容性。
    - 可以使用操作系统的api，但注意版本和平台的兼容性。尽量在uts插件中调用系统原生API，而不是在uvue页面中直接调用系统原生API。
    - 特定平台或特定版本以上才能使用的代码，需使用条件编译包围这些代码，或者放置在平台专用的目录文件中。
    - 通过mcp工具查询项目下可用的插件。
    - 跨页面通信优先使用eventbus。


# uvue rules

## vue support
    - 仅使用vue3语法，避免使用vue2。
    - 新页面尽量使用组合式API。
    - 组件尽量使用easycom规范。
    - 非easycom的自定义vue组件，调用组件方法时需使用组件实例的`$callMethod`方式调用。
    - 不使用 pinia、vuex、i18n 等uni-app x不支持的vue插件。
    - 使用vue语法时需注意uni-app x官网的平台和版本兼容性，平台特殊代码需包裹在条件编译中。

    ## component
    - 组件可使用uni-app x内置组件，以及项目下的自定义组件。通过mcp工具查询项目下可用的easycom插件。
    - 项目可使用vuejs组件规范，对应的文件扩展名为uvue。
    - 符合easycom规范的组件无需import和注册，可直接在template中使用。
    - 使用内置组件时需注意uni-app x官网的平台和版本兼容性，平台特殊代码需包裹在条件编译中。

# conditional compilation

## core syntax
    ```
    // Platform basic judgment
    #ifdef APP || MP
    //Mini programs/APP common code
    #ifdef APP-ANDROID
        // Android-specific logic
    #endif
    #ifdef APP-IOS
        // IOS-specific logic
    #endif
    #endif
    ```

## Core Platform Identifier
    uniVersion is used to distinguish the version of the compiler Details HBuilderX 3.9.0 
    APP App
    APP-ANDROID App Android Platform Details
    APP-IOS App iOS Platform Details
    APP-HARMONY App HarmonyOS Next platform
    WEB web (same as H5) HBuilderX 3.6.3 
    MP-WEIXIN WeChat Mini Program
    MP-ALIPAY APPLET
    MP-BAIDU BAIDU MINI PROGRAM
    MP-TUTIAO TIKTOK MINI PROGRAM
    MP-KUAISHOU Kuaishou Mini Program
    MP-JD JD Mini Program
    MP-HARMONY Harmony Atom Service HBuilderX 4.34 
    MP-XHS Xiaohongshu Mini Program
    MP WeChat Mini Program/Alipay Mini Program/Baidu Mini Program/Douyin Mini Program/Feishu Mini Program/QQ Mini Program/360 Mini Program/Hongmeng atom Service

# UTS Rules

ts 虽然有类型，但类型要求不严格。而 uts 为了编译为原生语言，是完全的强类型。

另外为了确保跨平台、高性能，uts 通过规范约束了 ts 中过于灵活而影响开发正确性或者给运行时带来不必要额外开销的特性。

本文罗列了在 uts 跨端开发时限制的 ts 特性，并提供了重构代码的建议。

> uni-app x下uts编译器目前[已知的一些问题](https://doc.dcloud.net.cn/uni-app-x/uts/compiler-known-issues.html)

> uni-app x下部分[运行错误的说明](https://doc.dcloud.net.cn/uni-app-x/uts/runtime-known-issues.html)

### 概述

#### 强制使用静态类型

静态类型是 uts 最重要的特性之一。如果程序采用静态类型，即所有类型在编译时都是已知的，那么开发者就能够容易理解代码中使用了哪些数据结构。
同时，由于所有类型在程序实际运行前都是已知的，编译器可以提前验证代码的正确性，从而可以减少运行时的类型检查，有助于提升性能。

基于上述考虑，uts 中的 any 类型与 ts 中的 any 并不一样，它是一个根据目标平台自动适配的跨端类型，通常用于表示"任意的非空类型"，
在使用时仍需类型转换后才能访问具体类型的方法和属性。

#### 不支持结构化类型系统

结构化类型系统（structural typing）是 ts 的一个特性，它意味着类型的兼容性和等价性是基于类型的结构（即它们的属性和方法）；
而在跨端开发时，uts 采用名义类型系统（nominal typing），类型兼容性检查基于类型名称和显式的继承/实现关系。
即使两个类型具有完全相同的结构，如果没有显式的继承/实现关系，它们也不能互相赋值。
这与 Kotlin 、Swift 、ArkTS 等静态语言的类型系统保持一致，有助于确保类型安全和代码的可维护性。

## 约束说明

### 1. 核心语言特性

#### 不支持 undefined @uts110111119

级别: 错误

错误码: UTS110111119

不支持 undefined。所有变量必须赋值初始化后才能使用。如果需要使用空，请使用 null。<br/> undefined 在 ts 中有很多场景，一个未初始化赋值的变量、一个未传入的方法参数、对象上不存在的属性，都会返回 undefined。

| 描述 | UTS-TS | UTS-ArkTS | UTS-Kotlin | UTS-Swift |
| :- | :- | :- | :- | :- |
| 适用版本 | 4.75 | 4.75 | 4.75 | 4.75 |
| 约束状态 | x | x | √ | √ |

TypeScript写法:

```ts
// TypeScript 可以使用 undefined
let value: string | undefined;
if (value == undefined) {
  console.log("未定义");
}

function test(param?: string) {
  if (param == undefined) {
    console.log("参数未传");
  }
}
```

UTS正确写法：

```ts
// UTS 使用 null 替代 undefined
let value: string | null = null; //必须先赋值后使用，哪怕赋值为null。否则Android平台会报编译错误：error: Variable 'value' must be initialized‌
if (value == null) {
  console.log("未定义");
}

function test(param: string | null) {
  if (param == null) {
    console.log("参数未传");
  }
}
```

#### 条件语句必须使用布尔类型 @uts110111120

级别: 错误

错误码: UTS110111120

所有条件语句(if、while、do-while、三元运算符、for 循环的条件部分)必须使用布尔类型作为条件。不支持 ts 中的隐式类型转换和 truthy/falsy 值。

| 描述 | UTS-TS | UTS-ArkTS | UTS-Kotlin | UTS-Swift |
| :- | :- | :- | :- | :- |
| 适用版本 | 4.75 | 4.75 | 4.75 | 4.75 |
| 约束状态 | x | x | √ | √ |

TypeScript写法:

```ts
// 允许非布尔值在条件语句中使用
if (1) {
} // 数值类型
while ("") {} // 字符串类型
do {} while (obj); // 对象类型
for (let i = 0; i; i++) {} // 作为条件的数值类型
const value = arr || []; // truthy/falsy 值判断
```

UTS正确写法：

```ts
// 必须使用布尔类型或返回布尔值的表达式
if (x > 0) {
} // 比较表达式
while (isValid) {} // 布尔变量
do {} while (obj != null); // 相等性判断
for (let i = 0; i < 10; i++) {} // 布尔条件
const value = arr != null ? arr : []; // 显式的布尔判断
```

#### 对象字面量默认为 UTSJSONObject 类型

级别：提示

在 UTS 中，所有没有明确类型标注或上下文推断不出具体类型的对象字面量都会被推导为 UTSJSONObject 类型。
这与 ts 中对象字面量会根据属性推导出具体结构类型的行为不同。

TypeScript写法:

```ts
// 类型推导为 { name: string, age: number }
const person = {
  name: "John",
  age: 30,
};

// 可以正常访问推导出的属性
console.log(person.name);
```

UTS正确写法：

```ts
// 类型推导为 UTSJSONObject
const person = {
  name: "John",
  age: 30,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dcloudio/uni-app-x-ai-rules](https://github.com/dcloudio/uni-app-x-ai-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
