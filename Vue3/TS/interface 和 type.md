在 TypeScript 中，`interface` 和 `type` 都用于定义类型，但它们有一些关键区别：

## 主要区别

### 1. **扩展性**

```typescript
// interface - 使用 extends
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}

// type - 使用 & 交叉类型
type AnimalType = {
  name: string;
};

type DogType = AnimalType & {
  breed: string;
};
```

### 2. **合并声明（Declaration Merging）**

```typescript
// interface 支持合并
interface User {
  name: string;
}

interface User {
  age: number;
}

// 最终 User 为：{ name: string; age: number; }

// type 不支持合并，会报错
type UserType = {
  name: string;
};

type UserType = {
  // ❌ 错误：重复标识符
  age: number;
};
```

### 3. **灵活性**

```typescript
// type 更灵活，可以定义联合类型、元组等
type Status = "active" | "inactive"; // 联合类型
type Coordinates = [number, number]; // 元组
type Callback = () => void; // 函数类型

// interface 主要用于对象类型
interface Point {
  x: number;
  y: number;
}
```

## 选择建议

### **使用 interface 的情况：**

1. **需要声明合并时**（如扩展第三方库类型）
2. **面向对象编程**（类实现接口）
3. **API 定义**（更清晰的继承结构）
4. **团队约定使用 interface**

```typescript
// 面向对象示例
interface Printable {
  print(): void;
}

class Document implements Printable {
  print() {
    console.log("Printing...");
  }
}
```

### **使用 type 的情况：**

1. **需要联合类型、交叉类型时**
2. **需要定义元组类型时**
3. **需要使用条件类型或映射类型时**
4. **简单的类型别名**

```typescript
// 复杂类型示例
type Nullable<T> = T | null;
type Record<K extends keyof any, T> = {
  [P in K]: T;
};

// 函数重载
type MathFunc = {
  (x: number, y: number): number;
  (x: string, y: string): string;
};
```

## 现代实践推荐

### **通用原则：**

1. **默认使用 interface** - 对于对象类型
2. **需要特殊类型时用 type** - 联合类型、元组等
3. **保持一致性** - 项目中统一风格

### **具体场景：**

```typescript
// 场景1：对象形状 - interface
interface User {
  id: string;
  name: string;
  email: string;
}

// 场景2：联合类型 - type
type UserRole = "admin" | "user" | "guest";

// 场景3：API响应 - interface（便于扩展）
interface ApiResponse<T> {
  data: T;
  status: number;
  message?: string;
}

// 场景4：工具类型 - type
type PartialUser = Partial<User>;
type PickUser = Pick<User, "id" | "name">;
```

## 性能考虑

- 早期版本中 `interface` 性能稍好
- 现代 TypeScript 中差异可以忽略
- 选择主要基于语义需求而非性能

## 总结表格

| 特性     | interface | type                            |
| -------- | --------- | ------------------------------- |
| 扩展     | `extends` | `&` 交叉类型                    |
| 合并声明 | ✅ 支持   | ❌ 不支持                       |
| 实现类   | ✅ 支持   | ❌ 不支持（对象类型可间接实现） |
| 联合类型 | ❌ 不支持 | ✅ 支持                         |
| 元组类型 | ❌ 不支持 | ✅ 支持                         |
| 映射类型 | ❌ 不支持 | ✅ 支持                         |

**最佳实践**：优先使用 `interface` 定义对象类型，使用 `type` 定义联合类型、元组等复杂类型。保持项目内一致。
