---
trigger: always_on
description: ts 虽然有类型，但类型要求不严格。而 uts 为了编译为原生语言，是完全的强类型。
---


## uts 和 ts 的差异

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
}; //对象字面量默认推导为UTSJSONObject
console.log(person["name"] as string); //UTSJSONObject类型不能直接用.运算符，并且下标访问后的每个值都是any类型，想正确使用时需要as为正确的类型
console.log(person["age"] as number); //UTS提供了有限的隐式转换能力，UTSJSONObject第一层对象如果可以被编译器识别推导类型，也可以使用.运算符。但第二层起无法使用.运算符，需要使用下标

// 在UTS中推荐使用type替代UTSJSONObject
type Person = {
  name: string;
  age: number;
};
// 声明时直接指定类型
const person2: Person = {
  name: "John",
  age: 30,
};
console.log(person2.name);
```

在 JSON.parse 的场景中，如果不通过泛型指定 type，那么返回值也是 UTSJSONObject。

TS 开发者一般都熟悉使用 interface 来声明类型，UTS 中改为 type 即可。但不熟悉 TS 的开发者，务必需要详细了解[UTSJSONObject](https://doc.dcloud.net.cn/uni-app-x/uts/data-type.html#UTSJSONObject)和[type](https://doc.dcloud.net.cn/uni-app-x/uts/data-type.html#type%E8%87%AA%E5%AE%9A%E4%B9%89%E7%B1%BB%E5%9E%8B)


#### 对象字面量仅支持构造 type 定义的对象类型，不支持 interface @uts110111163

级别: 错误

错误码: UTS110111163

ts 中可以通过 interface 或 type 声明对象字面量的类型。 <br/> 在 UTS 中，interface 有其他使用场景，所以对象字面量赋值只能给 type 关键词定义的对象类型。 <br/> 注意：不能赋值给 interface 定义的类型

| 描述 | UTS-TS | UTS-ArkTS | UTS-Kotlin | UTS-Swift |
| :- | :- | :- | :- | :- |
| 适用版本 | 4.75 | 4.75 | 4.75 | 4.75 |
| 约束状态 | x | x | √ | √ |

TypeScript写法:

```ts
interface Person {
  name: string;
  age: number;
}
// 对象字面量可以赋值 interface 类型
const person: Person = {
  name: "John",
  age: 30,
};
```

UTS正确写法：

```ts
// 只有 type 定义的对象类型，才可以用对象字面量赋值
type Person = {
  name: string;
  age: number;
};

// 声明时直接指定类型
const person1: Person = {
  name: "John",
  age: 30,
};
// 使用对象字面量 as 语法
const person2 = {
  name: "John",
  age: 30,
} as Person;
```

#### 不支持变量和函数的声明提升 (hoisting) @uts110111150

级别: 错误

错误码: UTS110111150

js/ts 在某些情况写可以忽视代码顺序，实现变量和函数的声明提升 (hoisting)。但 uts 编译为强类型语言时不行，所有变量和函数必须先声明后使用，严格注意顺序，不能访问未声明的变量或函数(包括函数自身)。

| 描述 | UTS-TS | UTS-ArkTS | UTS-Kotlin | UTS-Swift |
| :- | :- | :- | :- | :- |
| 适用版本 | 4.75 | 4.75 | 4.75 | 4.75 |
| 约束状态 | x | √ | √ | √ |

TypeScript写法:

```ts
// 变量提升
console.log(x); // undefined (不会报错)
var x = 5;

// 函数提升
foo(); // "hello" (可以在声明前调用)
function foo() {
  console.log("hello");
}

// 可以访问自身
const factorial = (n: number): number => {
  if (n <= 1) return 1;
  return n * factorial(n - 1); // 可以在函数内部调用自身
};
```

UTS正确写法：

```ts
// 变量必须先声明后使用
let x = 5;
console.log(x);
function foo() {
  console.log("hello");
}
// 函数必须先声明后调用
foo();

// 需要先声明函数变量
let factorial: ((n: number) => number) | null = null;
factorial = (n: number): number => {
  if (n <= 1) return 1;
  return n * factorial!(n - 1);
};
```

#### 使用 let 而非 var @uts110111121

级别: 警告


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dcloudio/uni-app-x-ai-rules](https://github.com/dcloudio/uni-app-x-ai-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
