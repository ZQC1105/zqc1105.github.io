# `import` vs `import type` 终极总结

## 📌 核心区别

| 特性           | `import`            | `import type`    |
| -------------- | ------------------- | ---------------- |
| **导入内容**   | 值和类型            | 只导入类型       |
| **运行时存在** | ✅ 生成 JS 导入语句 | ❌ 完全被移除    |
| **打包体积**   | 可能包含未用代码    | 零运行时开销     |
| **主要用途**   | 需要运行时功能时    | 只需要类型信息时 |

## 🎯 使用场景对照表

### 使用 `import`（需要运行时值）

```typescript
// 导入函数
import { fetchData, saveData } from "./api";

// 导入类（需要实例化）
import { UserModel } from "./models";

// 导入常量、配置
import { API_URL, MAX_SIZE } from "./config";

// 导入组件、模块
import React from "react";
import Router from "next/router";

// 导入工具函数
import { formatDate, debounce } from "./utils";
```

### 使用 `import type`（纯类型）

```typescript
// 导入接口/类型别名
import type { User, ApiResponse } from "./types";

// 导入 React 类型
import type { FC, PropsWithChildren } from "react";

// 避免循环依赖
import type { Post } from "./post"; // A 文件
import type { User } from "./user"; // B 文件

// 三方库类型
import type { Request, Response } from "express";
import type { AppProps } from "next/app";
```

## 🔧 混合导入模式

### TypeScript 4.5+ 推荐写法

```typescript
// 一行内混合（最清晰）
import { useState, useEffect, type FC, type ReactNode } from "react";

// 默认导入 + 类型
import MyComponent, { type Props } from "./MyComponent";

// 带重命名
import { saveUser, type User as UserType } from "./api";
```

### 分开导入（兼容性好）

```typescript
// 值导入
import { getUser, updateUser } from "./services";
import { logger } from "./utils";

// 类型导入
import type { User, UserForm } from "./types";
import type { ApiResponse } from "./api/types";
```

## ⚠️ 常见误区

### 错误用法

```typescript
// ❌ 类型应该用 import type
import { User } from "./types"; // 不清晰，虽然能工作

// ❌ 值不能用 import type
import type { fetchData } from "./api"; // 错误！需要运行时

// ❌ 类需要实例化时
import type { Database } from "./db";
const db = new Database(); // 错误！不能实例化
```

### 正确用法

```typescript
// ✅ 清晰区分
import type { User } from "./types"; // 类型
import { UserService } from "./services"; // 类（需要实例化）

// ✅ 需要时混合使用
import {
  database, // 值
  type DatabaseConfig, // 类型
  type QueryResult, // 类型
} from "./db";
```

## 📊 决策流程图

```
需要导入什么？
    ├── 只需要类型信息（接口、类型别名、泛型参数）
    │       └── 使用 `import type`
    │
    ├── 需要运行时功能
    │       ├── 函数、常量、类实例化
    │       ├── React/Vue 组件
    │       ├── 工具函数、配置值
    │       └── 使用普通 `import`
    │
    └── 混合需求
            ├── TypeScript 4.5+ → 混合语法
            └── 旧版本 → 分开导入
```

## 🚀 最佳实践

### 1. **优先使用 `import type`**

```typescript
// 好 ✅
import type { User } from "./types";
import { getUser } from "./api";

// 不好 ❌（不清晰）
import { User } from "./types"; // 虽然是类型
```

### 2. **按需导入，避免通配符**

```typescript
// 好 ✅
import { Button, Input } from "./ui";
import type { FormProps } from "./types";

// 不好 ❌
import * as UI from "./ui"; // 可能导入未用代码
```

### 3. **文件组织建议**

```typescript
// types/ 目录 - 纯类型定义
// 全部使用 import type
import type { User, Product, Order } from "../types";

// services/ 目录 - 业务逻辑
// 混合使用
import { apiClient } from "./client";
import type { ApiConfig } from "../types";
```

### 4. **配置建议**

```json
// tsconfig.json
{
  "compilerOptions": {
    // TypeScript 4.5+ 推荐
    "verbatimModuleSyntax": true,

    // 或者
    "isolatedModules": true,
    "importsNotUsedAsValues": "error"
  }
}
```

## 💡 记忆口诀

**"类型用 type，值用普通，混合写一行，分开也兼容"**

- **类型用 type**：接口、类型别名、泛型 → `import type`
- **值用普通**：函数、类、常量、组件 → 普通 `import`
- **混合写一行**：TypeScript 4.5+ 支持混合语法
- **分开也兼容**：分开导入兼容所有版本

## 🔍 检查清单

下次导入时问自己：

1. ✅ 我只需要类型信息吗？ → `import type`
2. ✅ 我需要运行时功能吗？ → 普通 `import`
3. ✅ 两者都需要？ → TypeScript 4.5+ 混合语法
4. ✅ 是否避免循环依赖？ → 优先 `import type`
5. ✅ 打包体积重要吗？ → `import type` 零开销

## 🎖️ 黄金法则

> **"当不确定时，先尝试 `import type`，如果 TypeScript 报错说需要值，再改为普通 `import`。"**

这个法则保证：

- 最大化类型安全
- 最小化运行时开销
- 代码意图最清晰
