# TypeScript 与 C# 语法对比详解

## 1. 语言定位与生态

| 特性         | TypeScript                               | C#                                    |
| ------------ | ---------------------------------------- | ------------------------------------- |
| **类型系统** | JavaScript 超集，静态类型检查            | 强类型，编译时类型检查                |
| **运行环境** | 编译为 JavaScript，在浏览器/Node.js 运行 | .NET 运行时（CLR）                    |
| **编译方式** | 转译（Transpilation）                    | 编译（Compilation）                   |
| **主要应用** | Web 前端，Node.js 后端                   | 桌面应用，Web 后端，游戏开发（Unity） |

## 2. 基础语法对比

### 变量声明

```typescript
// TypeScript
let name: string = "Alice";      // 块级作用域
const age: number = 25;         // 常量
var oldWay: boolean = true;     // 函数作用域（避免使用）

// C#
string name = "Alice";
const int age = 25;
readonly DateTime birthDate;    // 只读字段
```

### 类型推断

```typescript
// TypeScript - 类型推断
let count = 10;            // 推断为 number
let items = [1, 2, 3];     // 推断为 number[]

// C# - var 关键字
var count = 10;           // 推断为 int
var items = new[] {1, 2, 3}; // 推断为 int[]
```

## 3. 类型系统

### 基础类型

```typescript
// TypeScript
let isDone: boolean = false;
let decimal: number = 6;
let color: string = "blue";
let list: number[] = [1, 2, 3];
let tuple: [string, number] = ["hello", 10];
let notSure: any = 4;      // 任意类型
let u: undefined = undefined;
let n: null = null;

// C#
bool isDone = false;
double decimal = 6.0;
string color = "blue";
int[] list = {1, 2, 3};
Tuple<string, int> tuple = Tuple.Create("hello", 10);
dynamic notSure = 4;      // 动态类型
object obj = 4;           // 对象类型
```

### 枚举

```typescript
// TypeScript
enum Color {
    Red = 1,
    Green = 2,
    Blue = 4
}
let c: Color = Color.Green;

// 字符串枚举
enum Direction {
    Up = "UP",
    Down = "DOWN"
}

// C#
public enum Color
{
    Red = 1,
    Green = 2,
    Blue = 4
}
Color c = Color.Green;

// C# 不支持字符串枚举，可用静态类替代
public static class Direction
{
    public const string Up = "UP";
    public const string Down = "DOWN";
}
```

## 4. 面向对象编程

### 类定义

```typescript
// TypeScript
class Animal {
    private name: string;
    protected age: number;

    constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
    }

    public move(distance: number = 0): void {
        console.log(`${this.name} moved ${distance}m`);
    }

    // 抽象方法（需要在抽象类中）
    abstract makeSound(): void;
}

// C#
public class Animal
{
    private string name;
    protected int age;

    public Animal(string name, int age)
    {
        this.name = name;
        this.age = age;
    }

    public virtual void Move(int distance = 0)
    {
        Console.WriteLine($"{name} moved {distance}m");
    }

    // 抽象方法
    public abstract void MakeSound();
}
```

### 接口

```typescript
// TypeScript - 更灵活
interface IPerson {
    name: string;
    age: number;
    readonly id: number;     // 只读属性
    optionalProp?: string;   // 可选属性

    // 方法
    greet(): string;

    // 索引签名
    [key: string]: any;
}

// 接口可继承多个
interface Employee extends IPerson, IWorker {
    employeeId: string;
}

// C# - 更严格
public interface IPerson
{
    string Name { get; }     // 只读属性
    int Age { get; set; }

    string Greet();

    // C# 8.0+ 支持默认实现
    default string DefaultGreet() => "Hello";
}

// C# 不支持多继承，但可实现多个接口
public class Employee : IPerson, IWorker
{
    public string EmployeeId { get; set; }
}
```

### 继承

```typescript
// TypeScript
class Dog extends Animal {
    private breed: string;

    constructor(name: string, age: number, breed: string) {
        super(name, age);
        this.breed = breed;
    }

    override move(distance: number = 5): void {
        super.move(distance);
        console.log("Dog is running");
    }
}

// C#
public class Dog : Animal
{
    private string breed;

    public Dog(string name, int age, string breed) : base(name, age)
    {
        this.breed = breed;
    }

    public override void Move(int distance = 5)
    {
        base.Move(distance);
        Console.WriteLine("Dog is running");
    }
}
```

## 5. 泛型

```typescript
// TypeScript
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGeneric = new GenericNumber<number>();
myGeneric.zeroValue = 0;

// 泛型约束
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);
    return arg;
}

// C#
public class GenericNumber<T>
{
    public T ZeroValue { get; set; }
    public Func<T, T, T> Add { get; set; }
}

var myGeneric = new GenericNumber<int>();
myGeneric.ZeroValue = 0;

// 泛型约束
public interface ILengthwise
{
    int Length { get; }
}

public T LoggingIdentity<T>(T arg) where T : ILengthwise
{
    Console.WriteLine(arg.Length);
    return arg;
}
```

## 6. 异步编程

```typescript
// TypeScript - async/await (基于Promise)
async function fetchData(): Promise<string> {
    try {
        const response = await fetch('api/data');
        const data = await response.json();
        return data;
    } catch (error) {
        console.error(error);
        throw error;
    }
}

// C# - async/await (基于Task)
public async Task<string> FetchDataAsync()
{
    try
    {
        using var client = new HttpClient();
        var response = await client.GetAsync("api/data");
        response.EnsureSuccessStatusCode();
        var data = await response.Content.ReadAsStringAsync();
        return data;
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
        throw;
    }
}
```

## 7. 特性/装饰器（重要区别）

```typescript
// TypeScript - 装饰器（实验性）
function sealed(constructor: Function) {
    Object.seal(constructor);
    Object.seal(constructor.prototype);
}

@sealed
class Greeter {
    greeting: string;

    constructor(message: string) {
        this.greeting = message;
    }

    greet() {
        return "Hello, " + this.greeting;
    }
}

// C# - 特性（内置特性）
[Serializable]
[Obsolete("Use NewMethod instead")]
public class Greeter
{
    [Required]
    [StringLength(50)]
    public string Greeting { get; set; }

    [MethodImpl(MethodImplOptions.AggressiveInlining)]
    public string Greet()
    {
        return "Hello, " + Greeting;
    }
}
```

## 8. LINQ vs 数组方法

```typescript
// TypeScript - 数组方法
const numbers = [1, 2, 3, 4, 5];

const evenNumbers = numbers.filter(n => n % 2 === 0);
const squared = numbers.map(n => n * n);
const sum = numbers.reduce((acc, n) => acc + n, 0);

// 链式调用
const result = numbers
    .filter(n => n > 2)
    .map(n => n * 2)
    .reduce((sum, n) => sum + n, 0);

// C# - LINQ
var numbers = new List<int> { 1, 2, 3, 4, 5 };

var evenNumbers = numbers.Where(n => n % 2 == 0);
var squared = numbers.Select(n => n * n);
var sum = numbers.Sum();

// 链式调用
var result = numbers
    .Where(n => n > 2)
    .Select(n => n * 2)
    .Sum();
```

## 9. 空值处理

```typescript
// TypeScript - 联合类型和可选链
interface User {
    name: string;
    address?: {
        street?: string;
        city: string;
    };
}

function getCity(user: User | null): string | undefined {
    return user?.address?.city;
}

// 非空断言（谨慎使用）
let value: string | undefined = "hello";
let length: number = value!.length;

// C# - 可空引用类型和空条件运算符
public class User
{
    public string Name { get; set; }
    public Address? Address { get; set; }
}

public string? GetCity(User? user)
{
    return user?.Address?.City;
}

// 空合并运算符
string name = user?.Name ?? "Unknown";
```

## 10. 模式匹配

```typescript
// TypeScript - 类型守卫和判别联合
interface Circle { kind: "circle"; radius: number }
interface Square { kind: "square"; sideLength: number }
type Shape = Circle | Square;

function getArea(shape: Shape): number {
    switch (shape.kind) {
        case "circle":
            return Math.PI * shape.radius ** 2;
        case "square":
            return shape.sideLength ** 2;
    }
}

// C# - 模式匹配（C# 8.0+）
public abstract record Shape;
public record Circle(double Radius) : Shape;
public record Square(double SideLength) : Shape;

public double GetArea(Shape shape)
{
    return shape switch
    {
        Circle c => Math.PI * c.Radius * c.Radius,
        Square s => s.SideLength * s.SideLength,
        _ => throw new ArgumentException("Unknown shape")
    };
}
```

## 11. 模块系统

```typescript
// TypeScript - ES6 模块
// math.ts
export function add(x: number, y: number): number {
    return x + y;
}

export const PI = 3.14159;

// main.ts
import { add, PI } from './math';
import * as math from './math';

// C# - 命名空间和 using
// Math.cs
namespace MyApp.Math
{
    public static class Calculator
    {
        public static int Add(int x, int y) => x + y;
        public const double PI = 3.14159;
    }
}

// Program.cs
using MyApp.Math;
using static MyApp.Math.Calculator; // 静态导入

var result = Calculator.Add(1, 2);
var result2 = Add(3, 4); // 使用静态导入
```

## 总结对比表

| 特性           | TypeScript                  | C#                              |
| -------------- | --------------------------- | ------------------------------- |
| **类型注解**   | `: type` 后缀               | 类型前置                        |
| **空安全**     | 严格空检查，可选链 `?.`     | 可空引用类型，空条件运算符 `?.` |
| **异步**       | `Promise<T>`, `async/await` | `Task<T>`, `async/await`        |
| **属性**       | 简写语法支持                | 完整 get/set 访问器             |
| **事件**       | 自定义实现                  | 内置 `event` 关键字             |
| **反射**       | 有限，通过装饰器            | 完整的 `System.Reflection`      |
| **元编程**     | 装饰器（实验性）            | 特性、表达式树                  |
| **函数式特性** | 一等函数，部分支持          | LINQ，部分函数式特性            |
| **编译时检查** | 类型检查，泛型擦除          | 完整编译检查，运行时泛型        |

## 最佳实践建议

1. **从 C#转 TypeScript**：

   - 注意 JavaScript 的异步模型差异
   - 适应更灵活的类型系统
   - 学习函数式编程模式

2. **从 TypeScript 转 C#**：

   - 利用更强的类型安全
   - 学习完整的 OOP 特性
   - 掌握 LINQ 和委托系统

3. **通用建议**：
   - 两者都支持现代编程范式
   - 关注生态差异而非语法
   - 根据项目需求选择合适的工具

两种语言虽有许多相似之处，但设计哲学和运行环境有本质区别。理解这些差异有助于在不同平台间高效切换和开发。
