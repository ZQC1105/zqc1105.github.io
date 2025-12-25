在 TypeScript 中，**`type`** 是一个关键字，用于创建**类型别名**（Type Alias）。它让开发者能够为一个已有的类型定义一个新名称，或者创建复杂的联合类型、交叉类型等。

## 基本用法

```typescript
// 1. 基本类型别名
type ID = number | string;
type UserName = string;

// 2. 对象类型别名
type User = {
  id: ID;
  name: string;
  age: number;
  email?: string; // 可选属性
};

// 3. 使用类型别名
const user1: User = {
  id: 1,
  name: "Alice",
  age: 25,
};
```

## 主要特性

### 1. 联合类型（Union Types）

```typescript
type Status = "pending" | "success" | "error";
type UserRole = "admin" | "user" | "guest";
type NullableString = string | null | undefined;
```

### 2. 交叉类型（Intersection Types）

```typescript
type Person = {
  name: string;
  age: number;
};

type Employee = Person & {
  employeeId: string;
  department: string;
};
```

### 3. 映射类型（Mapped Types）

```typescript
// 将所有属性变为可选
type Partial<T> = {
  [P in keyof T]?: T[P];
};

// 将所有属性变为只读
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};
```

### 4. 条件类型（Conditional Types）

```typescript
type IsString<T> = T extends string ? true : false;
type Result = IsString<"hello">; // true
```

## type vs interface

虽然两者相似，但有重要区别：

| 特性          | type                 | interface               |
| ------------- | -------------------- | ----------------------- |
| 扩展方式      | 使用 `&`（交叉类型） | 使用 `extends`          |
| 合并声明      | ❌ 不能合并          | ✅ 可以合并（声明合并） |
| 实现类        | ❌ 不能              | ✅ 可以                 |
| 元组/联合类型 | ✅ 支持              | ❌ 不支持               |
| 映射类型      | ✅ 支持              | ⚠️ 有限支持             |

### 示例对比：

```typescript
// 使用 type
type Animal = {
  name: string;
};

type Bear = Animal & {
  honey: boolean;
};

// 使用 interface
interface Animal {
  name: string;
}

interface Bear extends Animal {
  honey: boolean;
}
```

## 实用类型别名示例

```typescript
// 1. 提取函数返回类型
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

// 2. 提取数组元素类型
type ArrayElement<T> = T extends Array<infer U> ? U : never;

// 3. 异步函数返回类型
type PromiseType<T> = T extends Promise<infer U> ? U : never;

// 4. 部分属性只读
type ReadonlyPartial<T> = {
  readonly [P in keyof T]?: T[P];
};
```

## 最佳实践

1. **何时使用 type：**

   - 创建联合类型、元组类型
   - 使用映射类型、条件类型
   - 简单的对象类型别名
   - 需要类型组合时（使用 `&`）

2. **何时使用 interface：**

   - 需要声明合并时
   - 定义对象类型且需要被类实现时
   - 库的类型定义（为了更好的扩展性）

3. **一致性：** 在项目中保持统一的选择，增强代码可读性。

```typescript
// 实际应用示例
type ApiResponse<T> = {
  data: T;
  status: number;
  message: string;
  timestamp: Date;
};

type PaginatedResponse<T> = ApiResponse<T[]> & {
  page: number;
  totalPages: number;
  totalItems: number;
};
```

`type` 是 TypeScript 类型系统中非常强大的工具，合理使用可以显著提高代码的类型安全性和可维护性。
