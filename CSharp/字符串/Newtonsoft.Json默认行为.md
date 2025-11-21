下面把 ASP.NET Core 中 `System.Text.Json` 的“默认行为”逐条拆开说明，告诉你**为什么**、**怎么做**、**坑在哪**，并给出可运行的最小代码，看完就能彻底记住。

------------------------------------------------
1. 命名策略：PascalCase → camelCase  
------------------------------------------------
- **仅影响序列化后的字节**，反序列化时**不依赖**这个策略。  
- 实现方式：  
  `JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase`  
  这是 ASP.NET Core 在 `JsonOptions` 里**硬编码**的默认值，不是你项目里写了才生效。  

- 反序列化阶段：  
  框架会先把 JSON 字段名**统一成小写**，再跟 C# 属性名的小写形式比对，因此  
  `{"userName": "x"}`、`{"UserName": "x"}`、`{"USERNAME": "x"}` 都能绑定到  
  `public string UserName { get; set; }`。

------------------------------------------------
2. 大小写不敏感：源码级证明  
------------------------------------------------
核心代码在 `System.Text.Json.Serialization.Metadata.JsonPropertyInfo`  
```csharp
// 伪代码
bool NameMatches(string jsonPropertyName) =>
    PropertyName.AsSpan().Equals(jsonPropertyName, StringComparison.OrdinalIgnoreCase);
```
所以**与命名策略无关**，反序列化永远忽略大小写。

------------------------------------------------
3. null 值：默认不忽略  
------------------------------------------------
- 序列器会**显式写出 `"prop": null`**，前端收到的 JSON 里能看到。  
- 想省流量：  
```csharp
builder.Services.Configure<JsonOptions>(o =>
    o.SerializerOptions.DefaultIgnoreCondition =
        JsonIgnoreCondition.WhenWritingNull);
```

------------------------------------------------
4. 只认 public **属性**（property）  
------------------------------------------------
- 字段 (`public string Foo;`) 默认被**完全忽略**。  
- 想开启字段：  
```csharp
options.IncludeFields = true;   // 或给字段加 [JsonInclude]
```

------------------------------------------------
5. init / get-only / record 支持  
------------------------------------------------
- `.NET 5+` 增加了 **constructor-based deserialization**。  
- 框架会先找**最匹配的构造函数**，再按参数名（忽略大小写）匹配 JSON 字段。  
示例：
```csharp
public record User(string FullName, int Age); // 主构造
// JSON: {"fullName":"Alice","age":20}
```
反序列化时**不需要无参构造**，直接 `new User("Alice", 20)`。

------------------------------------------------
6. 循环引用：默认直接抛 `JsonException`  
------------------------------------------------
```csharp
public class Node { public Node Child { get; set; } }
var root = new Node(); root.Child = root;
JsonSerializer.Serialize(root);   // 抛异常
```
解决：
```csharp
services.Configure<JsonOptions>(o =>
    o.SerializerOptions.ReferenceHandler = ReferenceHandler.Preserve);
```
输出会带 `$id` / `$ref` 标记，类似 Newtonsoft 的 `ReferenceLoopHandling.Serialize`。

------------------------------------------------
7. 日期格式：ISO-8601 是**唯一**默认  
------------------------------------------------
- 格式：`2025-11-21T14:30:00.123Z`（UTC，带时区标记）。  
- 不支持 `"/Date(123456789)/"` 旧格式；需要自定义 `JsonConverter<DateTime>`。

------------------------------------------------
8. 与 Newtonsoft 的“肉眼可见”差异表  
------------------------------------------------
| 场景 | System.Text.Json 默认 | Newtonsoft.Json 默认 |
|------|------------------------|------------------------|
| 命名 | camelCase | 保持 PascalCase |
| 大小写匹配 | 不敏感 | 敏感 |
| 字段 | 忽略 | 可选（DefaultContractResolver） |
| 循环引用 | 抛异常 | 可选（ReferenceLoopHandling） |
| 性能 | 高（UTF8 直接写） | 低（先转字符串） |
| 特性丰富度 | 少 | 多（JsonProperty、JsonConstructor…） |

------------------------------------------------
9. 最佳实践速记口诀  
------------------------------------------------
1. **前端 camel，后端 Pascal**：默认就能对上，不用写任何特性。  
2. **不要无脑改命名策略**：一改就必须**连带**客户端、OpenAPI 文档、Postman 一起改。  
3. **record + init** 写 DTO：不可变、线程安全、支持反序列化。  
4. **有循环引用就加** `ReferenceHandler.Preserve`，否则一定炸。  
5. **老系统要 Pascal JSON**：  
```csharp
// 服务端
options.PropertyNamingPolicy = null;
// 客户端
HttpClient.PostAsJsonAsync(
    url, payload,
    new JsonSerializerOptions { PropertyNamingPolicy = null });
```
两边保持一致即可。

------------------------------------------------
10. 一段代码跑通所有默认行为  
------------------------------------------------
```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();   // 默认已配好 JsonOptions
var app = builder.Build();
app.MapControllers();
app.Run();

[ApiController]
[Route("[controller]")]
public class UsersController : ControllerBase
{
    public record User(string FullName, string? Email);

    [HttpPost]
    public User Create([FromBody] User user) => user with { FullName = user.FullName.ToUpper() };
}
```
请求：
```http
POST /users
Content-Type: application/json

{ "fullName": "Alice", "email": null }
```
响应：
```json
{"fullName":"ALICE","email":null}
```
全程**零配置**，大小写、null、init-only、record 都能跑通。

------------------------------------------------
总结一句话（加强版）：  
ASP.NET Core 的 `System.Text.Json` 默认用 **camelCase 输出**、**大小写不敏感输入**、**只认 public 属性**、**留 null**、**不支持循环引用**，**record/init 开箱即用**；记住这 6 点，以后再也不用翻文档。