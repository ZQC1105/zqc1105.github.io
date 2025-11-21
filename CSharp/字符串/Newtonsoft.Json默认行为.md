## [JsonSerializerDefaults Enum](https://learn.microsoft.com/zh-cn/dotnet/api/system.text.json.jsonserializerdefaults?view=net-9.0#system-text-json-jsonserializerdefaults-web)
PostAsJsonAsync<TValue>(HttpClient, String, TValue, CancellationToken)
此方法使用 JsonSerializerDefaults.Web 选项进行序列化，而 JsonSerializer 序列化方法默认不使用。
System.Text.Json 预置枚举对比表（General vs Web）

| 特性 / 选项  | General (0)    | Web (1)                                      |
| ------------ | -------------- | -------------------------------------------- |
| 大小写敏感   | ✅ 敏感         | ❌ 不敏感（PropertyNameCaseInsensitive=true） |
| 属性命名策略 | 无（保持原样） | CamelCase（JsonNamingPolicy.CamelCase）      |
| 允许注释     | ❌              | ✅（ReadCommentHandling=Skip）                |
| 允许尾随逗号 | ❌              | ✅（AllowTrailingCommas=true）                |
| 允许引号数字 | ❌              | ✅（NumberHandling=AllowReadingFromString）   |
| 忽略 null 值 | ❌ 写出 null    | ❌ 写出 null（**未**设 WhenWritingNull）      |
| 缩进         | ❌ 无           | ❌ 无（需手动 WriteIndented=true）            |

> 除上表列出的差异外，其余 JsonSerializerOptions 属性均保持框架级默认值。