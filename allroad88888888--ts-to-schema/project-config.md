---
trigger: always_on
description: Transforms to:
---

# TS to JSON Schema Conversion Rules

These rules define how to transform TypeScript types into JSON Schema dynamically.
They will be used by cursor to generate JSON Schema from provided TypeScript code.

---

## Rule 1: Interface Declarations

- **When:** The node represents an `InterfaceDeclaration`.
- **Action:** 
  - Generate an object schema.
  - Use the interface name as the schema title.
  - For each property:
    - Recursively transform its type.
    - Include a description if available.
    - If not optional, add the property name to the `required` array.
- **Example:**  
  For an interface like:
  ```tsx
  export interface VDom extends HTMLElement {
    custom: string
  }
  ```
  The transformation should produce an object schema that extends the `HTMLElement` definition and adds a `custom` property.

---

## Rule 2: Union Types

- **When:** The node represents a union type (using the `|` operator).
- **Action:** 
  - Create an `anyOf` array.
  - Each member of the union is recursively transformed.
- **Example:**  
  For:
  ```tsx
  export type SimpleChartMark = SimpleIntervalMark | SimplePointMark;
  ```
  The resulting schema should have an `anyOf` with two schemas: one for `SimpleIntervalMark` and one for `SimplePointMark`.

---

## Rule 3: Literal String Types

- **When:** The node is a literal type with a string value (e.g. `'interval'` or `'point'`).
- **Action:** 
  - Generate a schema with `type: "string"` and use `const` to lock the value.
  - Include any available description.
- **Example:**  
  ```tsx
  type: 'interval';
  ```
  Transforms to:
  ```json
  {
    "type": "string",
    "const": "interval",
    "description": "标记类型标识符"
  }
  ```

---

## Rule 4: Primitive Types

- **When:** Encountering TypeScript primitive types.
- **Action:**  
  - `number` → `{ "type": "number" }`
  - `string` → `{ "type": "string" }`

---

## Rule 5: Date Type

- **When:** The type is a reference to `Date`.
- **Action:**  
  - Transform to a JSON string with ISO date-time format:
    ```json
    {
      "type": "string",
      "format": "date-time",
      "description": "Date 对象，在 JSON 中序列化为 ISO 日期字符串"
    }
    ```

---

## Rule 6: Type References

- **When:** A node is a type reference (除了 Date 外)。
- **Action:**  
  - Generate a `$ref` pointing to the corresponding definition in the `#/definitions` section.
- **Example:**  
  ```tsx
  vDom: VDom;
  ```
  Transforms to:
  ```json
  { "$ref": "#/definitions/VDom" }
  ```

---

## Rule 7: Object Literals / Inline Objects

- **When:** The node represents an inline object type.
- **Action:**  
  - Recursively parse its properties, similar to interface declarations.
  - Build an object schema，包含 `properties` 和 `required` 字段。

---

## Definitions Handling

- 所有通过 TypeReference 引用的类型（如 `HTMLElement`、`CSSProperties` 等）都应在最终 schema 的 `definitions` 中定义。
- 定义的内容可以进一步扩展，支持更多属性。
- 示例中：
  - `HTMLElement` 包含 `id`、`className`、`tagName`、`innerHTML` 和 `style` 属性。
  - `CSSProperties` 定义常见的 CSS 属性及允许的额外属性。

---

## Example Transformation

假设有如下 TypeScript 定义：

```tsx
export interface VDom extends HTMLElement{
  custom:string
}

export type SimpleIntervalMark = {
  type: 'interval';
  x: number;
  y: number;
  color: string;
  vDom: VDom;
};

export type SimplePointMark = {
  type: 'point';
  x: number;
  y: number;
  size: number;
  date: Date;
  style: CssProperties;
};

export type SimpleChartMark = SimpleIntervalMark | SimplePointMark;
```

根据以上规则，cursor 生成的 JSON Schema 应该类似于：

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "SimpleChartMark",
  "description": "图表标记的联合类型，支持区间和点两种标记类型",
  "anyOf": [
    {
      "title": "SimpleIntervalMark",
      "type": "object",
      "properties": {
        "type": {
          "type": "string",
          "const": "interval",
          "description": "标记类型标识符"
        },
        "x": { "type": "number", "description": "X轴坐标" },
        "y": { "type": "number", "description": "Y轴坐标" },
        "color": { "type": "string", "description": "标记颜色" },
        "vDom": {
          "type": "object",
          "description": "虚拟DOM元素，继承自HTMLElement",
          "allOf": [
            { "$ref": "#/definitions/HTMLElement" },
            {
              "properties": {
                "custom": {
                  "type": "string",
                  "description": "自定义属性"
                }
              },
              "required": ["custom"]
            }
          ]
        }
      },
      "required": ["type", "x", "y", "color", "vDom"]
    },
    {
      "title": "SimplePointMark",
      "type": "object",
      "properties": {
        "type": {
          "type": "string",
          "const": "point",
          "description": "标记类型标识符"
        },
        "x": { "type": "number", "description": "X轴坐标" },
        "y": { "type": "number", "description": "Y轴坐标" },
        "size": { "type": "number", "description": "点的大小" },
        "date": {
          "type": "string",
          "format": "date-time",
          "description": "Date 对象，在 JSON 中序列化为 ISO 日期字符串"
        },
        "style": {
          "type": "object",
          "$ref": "#/definitions/CSSProperties",
          "description": "CSS样式属性集合，类型为 CssProperties"
        }
      },
      "required": ["type", "x", "y", "size", "date", "style"]
    }
  ],
  "definitions": {
    "HTMLElement": {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/allroad88888888) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
