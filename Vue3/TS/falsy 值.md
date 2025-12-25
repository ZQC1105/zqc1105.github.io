在 JavaScript/TypeScript 中，**falsy 值**（假值）是指在布尔上下文中会被转换为 `false` 的值。

## 完整的 falsy 值列表

JavaScript 中有且仅有以下 8 个 falsy 值：

```typescript
false; // 布尔值 false
0 - // 数字 0
  0; // 负 0
0n; // BigInt 0
(""); // 空字符串
"" // 空字符串（单引号）
``; // 空字符串（模板字符串）
null; // 空值
undefined; // 未定义
NaN; // 非数字
```

## 示例演示

```typescript
// 使用 if 语句测试
if (false) {
} // 不会执行
if (0) {
} // 不会执行
if (-0) {
} // 不会执行
if (0n) {
} // 不会执行
if ("") {
} // 不会执行
if (null) {
} // 不会执行
if (undefined) {
} // 不会执行
if (NaN) {
} // 不会执行

// 以下都是 truthy 值
if (true) {
} // 执行
if (1) {
} // 执行
if (-1) {
} // 执行
if ("hello") {
} // 执行
if (" ") {
} // 执行（空格不是空字符串）
if ([]) {
} // 执行（空数组是 truthy！）
if ({}) {
} // 执行（空对象是 truthy！）
if (function () {}) {
} // 执行（函数是 truthy！）
```

## 实际应用场景

### 1. **使用 `||` 设置默认值**

```typescript
const name = "" || "默认名"; // "默认名"（因为 "" 是 falsy）
const count = 0 || 5; // 5（因为 0 是 falsy）
const active = false || true; // true（因为 false 是 falsy）
```

### 2. **使用 `??` 避免 0 被误判**

```typescript
const count1 = 0 || 10; // 10（可能不是我们想要的！）
const count2 = 0 ?? 10; // 0（保留 0 值）
```

### 3. **条件判断的陷阱**

```typescript
function printLength(str: string | null) {
  if (str) {
    // 这里 str 不会是 ""、null、undefined
    console.log(str.length);
  }
}

printLength(""); // 不会打印（"" 是 falsy）
printLength("hi"); // 打印 2
printLength(null); // 不会打印
```

## 记忆技巧

记住这个简单的规律：

- **数字**：只有 0（包括 0, -0, 0n）是 falsy，其他数字都是 truthy
- **字符串**：只有空字符串是 falsy，其他都是 truthy（包括空格字符串）
- **特殊类型**：false, null, undefined, NaN 都是 falsy
- **对象类型**：所有对象（包括数组、函数、空对象）都是 truthy

## 实际开发建议

```typescript
// 检查数组是否为空（错误方式）
const arr: number[] = [];
if (arr) {
  // 这里会执行！因为空数组是 truthy
}

// 检查数组是否为空（正确方式）
if (arr.length > 0) {
  // 只在数组有元素时执行
}

// 检查字符串是否有内容
const str = "";
if (str) {
  /* 不会执行 */
}
if (str !== "") {
  /* 明确检查 */
}

// 检查变量是否存在且有值
let value: string | undefined;
if (value) {
  /* 不会执行 */
}
if (value !== undefined) {
  /* 明确检查 */
}
```

理解 falsy 值对于正确使用逻辑运算符（`||`、`&&`）和条件判断非常重要！
