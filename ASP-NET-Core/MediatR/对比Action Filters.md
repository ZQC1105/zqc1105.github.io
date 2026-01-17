你记得没错！**ASP.NET Core 确实有另一个非常类似的管道机制** - 那就是 **Action Filters（动作过滤器）**。它和 MediatR 管道行为在概念上非常相似。

## **Action Filters（动作过滤器）**

这是 ASP.NET Core MVC 中的一个核心特性，提供了与 MediatR 管道行为类似的 AOP 能力，但作用于 **Controller/Action 级别**。

### 1. **基本概念对比**

| 特性         | MediatR 管道行为                  | ASP.NET Core Action Filters                        |
| ------------ | --------------------------------- | -------------------------------------------------- |
| **作用对象** | `IRequest<T>` / `INotification`   | `ActionExecutingContext` / `ActionExecutedContext` |
| **作用范围** | 业务命令/查询                     | MVC Action 方法                                    |
| **执行时机** | 业务逻辑执行前后                  | Action 方法执行前后                                |
| **AOP 能力** | 完整的 AOP（Before/After/Around） | 完整的 AOP（Before/After/Around）                  |
| **异常处理** | 可以捕获处理                      | 可以捕获处理                                       |
| **返回结果** | 可以修改返回值                    | 可以修改 ActionResult                              |

### 2. **Action Filters 的类型**

```csharp
// 1. 同步 Action Filter
public class CustomActionFilter : IActionFilter
{
    // Action执行前
    public void OnActionExecuting(ActionExecutingContext context)
    {
        // 相当于 Before Advice
        Log.Information($"Action {context.ActionDescriptor.DisplayName} 开始执行");

        // 可以修改参数
        if (context.ActionArguments.ContainsKey("id"))
        {
            context.ActionArguments["id"] = ModifyId(context.ActionArguments["id"]);
        }

        // 可以短路执行（不执行Action）
        if (!IsValid(context))
        {
            context.Result = new BadRequestObjectResult("参数无效");
        }
    }

    // Action执行后
    public void OnActionExecuted(ActionExecutedContext context)
    {
        // 相当于 After Advice
        Log.Information($"Action {context.ActionDescriptor.DisplayName} 执行完成");

        // 可以修改结果
        if (context.Result is ObjectResult objectResult)
        {
            objectResult.Value = WrapResponse(objectResult.Value);
        }

        // 可以处理异常
        if (context.Exception != null && !context.ExceptionHandled)
        {
            Log.Error(context.Exception, "Action执行异常");
            context.Result = new ObjectResult("内部错误")
            {
                StatusCode = 500
            };
            context.ExceptionHandled = true;
        }
    }
}

// 2. 异步 Action Filter（推荐）
public class AsyncActionFilter : IAsyncActionFilter
{
    public async Task OnActionExecutionAsync(
        ActionExecutingContext context,
        ActionExecutionDelegate next)
    {
        // Action执行前 - Before Advice
        Log.Information($"开始执行 Action: {context.ActionDescriptor.DisplayName}");
        var stopwatch = Stopwatch.StartNew();

        try
        {
            // 执行Action - Around Advice
            var resultContext = await next();
            stopwatch.Stop();

            // Action执行后 - After Returning Advice
            Log.Information($"Action执行成功，耗时: {stopwatch.ElapsedMilliseconds}ms");

            // 可以修改结果
            if (resultContext.Result is ObjectResult objectResult)
            {
                objectResult.Value = new ApiResponse
                {
                    Success = true,
                    Data = objectResult.Value,
                    ExecutionTime = stopwatch.ElapsedMilliseconds
                };
            }
        }
        catch (Exception ex)
        {
            // 异常处理 - After Throwing Advice
            stopwatch.Stop();
            Log.Error(ex, $"Action执行失败，耗时: {stopwatch.ElapsedMilliseconds}ms");

            context.Result = new ObjectResult(new ApiResponse
            {
                Success = false,
                Error = ex.Message,
                ExecutionTime = stopwatch.ElapsedMilliseconds
            })
            {
                StatusCode = 500
            };
        }
        finally
        {
            // 最终处理 - After Finally Advice
            // 资源清理等
        }
    }
}

// 3. 其他类型的Filter
public class AuthorizationFilter : IAuthorizationFilter
{
    public void OnAuthorization(AuthorizationFilterContext context)
    {
        // 授权检查 - 在Action执行前最先执行
        if (!IsAuthorized(context.HttpContext.User))
        {
            context.Result = new ForbidResult();
        }
    }
}

public class ResourceFilter : IResourceFilter
{
    public void OnResourceExecuting(ResourceExecutingContext context)
    {
        // 资源处理 - 在Authorization之后，Model Binding之前
    }

    public void OnResourceExecuted(ResourceExecutedContext context)
    {
        // 资源处理后
    }
}

public class ResultFilter : IResultFilter
{
    public void OnResultExecuting(ResultExecutingContext context)
    {
        // 结果执行前 - 在Action返回结果后，但结果未写入响应前
    }

    public void OnResultExecuted(ResultExecutedContext context)
    {
        // 结果执行后 - 结果已写入响应
    }
}

public class ExceptionFilter : IExceptionFilter
{
    public void OnException(ExceptionContext context)
    {
        // 专门处理异常
        if (context.Exception is ValidationException validationEx)
        {
            context.Result = new BadRequestObjectResult(validationEx.Errors);
            context.ExceptionHandled = true;
        }
    }
}
```

### 3. **与 MediatR 管道行为的对比**

#### **相同点**：

```csharp
// 1. 都实现了AOP思想
public class AopComparison
{
    // MediatR管道行为
    public class MediatRPipeline : IPipelineBehavior<TRequest, TResponse>
    {
        public async Task<TResponse> Handle(TRequest request, RequestHandlerDelegate<TResponse> next, CancellationToken ct)
        {
            // Before
            var response = await next(); // Around
            // After
            return response;
        }
    }

    // Action Filter
    public class ActionFilter : IAsyncActionFilter
    {
        public async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
        {
            // Before
            var result = await next(); // Around
            // After
        }
    }
}

// 2. 都支持横切关注点的分离
public class CrossCuttingConcerns
{
    // 都可以处理：
    // - 日志记录
    // - 验证
    // - 授权
    // - 性能监控
    // - 缓存
    // - 异常处理
    // - 事务管理
}

// 3. 都基于管道/中间件模式
public class PipelinePattern
{
    // 都是责任链模式的变体
    // 都支持"洋葱模型"执行
}
```

#### **不同点**：

```csharp
public class Differences
{
    // 1. 作用域不同
    public void ScopeDifference()
    {
        // MediatR管道行为: 作用于业务命令/查询级别
        // Action Filters: 作用于HTTP请求/Action方法级别
    }

    // 2. 集成方式不同
    public void IntegrationDifference()
    {
        // MediatR: 通过依赖注入自动注册
        services.AddTransient(typeof(IPipelineBehavior<,>), typeof(LoggingBehavior<,>));

        // Action Filters: 可以通过多种方式注册
        // - 全局注册
        services.AddControllers(options =>
        {
            options.Filters.Add<GlobalActionFilter>();
        });

        // - 控制器级别
        [ServiceFilter(typeof(CustomActionFilter))]
        public class OrdersController : ControllerBase

        // - Action级别
        [TypeFilter(typeof(ValidationFilter))]
        public IActionResult CreateOrder([FromBody] OrderRequest request)
    }

    // 3. 执行顺序不同
    public void ExecutionOrder()
    {
        /*
        ASP.NET Core Filters执行顺序：
        1. 授权过滤器 (Authorization Filters)
        2. 资源过滤器 (Resource Filters) - OnResourceExecuting
        3. 模型绑定 (Model Binding)
        4. Action过滤器 (Action Filters) - OnActionExecuting
        5. Action方法执行
        6. Action过滤器 (Action Filters) - OnActionExecuted
        7. 异常过滤器 (Exception Filters) - 如果有异常
        8. 结果过滤器 (Result Filters) - OnResultExecuting
        9. 结果执行 (Result Execution)
        10. 结果过滤器 (Result Filters) - OnResultExecuted
        11. 资源过滤器 (Resource Filters) - OnResourceExecuted

        MediatR管道行为执行顺序：
        按注册顺序执行
        */
    }

    // 4. 上下文信息不同
    public void ContextDifference()
    {
        // Action Filters有完整的HTTP上下文
        var httpContext = context.HttpContext;
        var request = context.HttpContext.Request;
        var response = context.HttpContext.Response;

        // MediatR管道行为只有业务上下文
        var requestData = request; // IRequest对象
        // 没有直接的HTTP上下文访问
    }
}
```

### 4. **实际应用场景对比**

#### **适合使用 Action Filters 的场景**：

```csharp
// 场景1：API响应统一包装
public class ApiResponseFilter : IAsyncActionFilter
{
    public async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
    {
        var result = await next();

        if (result.Result is ObjectResult objectResult)
        {
            objectResult.Value = new ApiResponse
            {
                Success = true,
                Data = objectResult.Value,
                Timestamp = DateTime.UtcNow
            };
        }
    }
}

// 场景2：请求限流
public class RateLimitFilter : IAsyncActionFilter
{
    public async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
    {
        var clientIp = context.HttpContext.Connection.RemoteIpAddress.ToString();

        if (!_rateLimiter.AllowRequest(clientIp))
        {
            context.Result = new ObjectResult("请求过于频繁")
            {
                StatusCode = 429 // Too Many Requests
            };
            return;
        }

        await next();
    }
}

// 场景3：模型验证（替代ModelState.IsValid）
public class AutoValidateModelFilter : IAsyncActionFilter
{
    public async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
    {
        if (!context.ModelState.IsValid)
        {
            var errors = context.ModelState
                .Where(e => e.Value.Errors.Count > 0)
                .ToDictionary(
                    kvp => kvp.Key,
                    kvp => kvp.Value.Errors.Select(e => e.ErrorMessage).ToArray()
                );

            context.Result = new BadRequestObjectResult(new
            {
                Message = "验证失败",
                Errors = errors
            });
            return;
        }

        await next();
    }
}
```

#### **适合使用 MediatR 管道行为的场景**：

```csharp
// 场景1：业务事务管理
public class TransactionBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
{
    public async Task<TResponse> Handle(TRequest request, RequestHandlerDelegate<TResponse> next, CancellationToken ct)
    {
        // 业务级别的事务，Action Filter无法处理
        await using var transaction = await _dbContext.Database.BeginTransactionAsync(ct);

        try
        {
            var response = await next();
            await _dbContext.SaveChangesAsync(ct);
            await transaction.CommitAsync(ct);
            return response;
        }
        catch
        {
            await transaction.RollbackAsync(ct);
            throw;
        }
    }
}

// 场景2：业务缓存（基于业务逻辑）
public class BusinessCachingBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
    where TRequest : ICacheableRequest
{
    public async Task<TResponse> Handle(TRequest request, RequestHandlerDelegate<TResponse> next, CancellationToken ct)
    {
        // 基于业务逻辑的缓存策略
        var cacheKey = $"{typeof(TRequest).Name}:{request.GetCacheKey()}";

        var cached = await _cache.GetAsync<TResponse>(cacheKey, ct);
        if (cached != null) return cached;

        var response = await next();
        await _cache.SetAsync(cacheKey, response, TimeSpan.FromMinutes(5), ct);

        return response;
    }
}

// 场景3：业务验证（复杂业务规则）
public class BusinessValidationBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
    where TRequest : IRequest<TResponse>
{
    public async Task<TResponse> Handle(TRequest request, RequestHandlerDelegate<TResponse> next, CancellationToken ct)
    {
        // 业务规则验证，不仅仅是数据格式
        if (request is CreateOrderCommand orderCommand)
        {
            var inventory = await _inventoryService.GetStockAsync(orderCommand.ProductId);
            if (inventory < orderCommand.Quantity)
                throw new BusinessException("库存不足");

            var credit = await _creditService.GetUserCreditAsync(orderCommand.UserId);
            if (credit < orderCommand.TotalAmount)
                throw new BusinessException("信用额度不足");
        }

        return await next();
    }
}
```

### 5. **如何选择？**

#### **选择 Action Filters 当**：

```csharp
// 1. 需要访问HTTP上下文
if (needHttpContext)
{
    // Action Filters可以访问：
    // - Request/Response对象
    // - Headers/Cookies/Session
    // - User身份信息
    // - Route数据
}

// 2. 处理MVC/Action特定逻辑
if (mvcSpecific)
{
    // - 模型绑定验证
    // - View渲染处理
    // - ActionResult转换
}

// 3. 全局API策略
if (globalApiPolicy)
{
    // - 统一响应格式
    // - 全局异常处理
    // - API版本控制
    // - 跨域处理
}
```

#### **选择 MediatR 管道行为当**：

```csharp
// 1. 纯业务逻辑处理
if (pureBusinessLogic)
{
    // - 业务事务管理
    // - 业务规则验证
    // - 领域事件发布
}

// 2. 需要业务上下文
if (needBusinessContext)
{
    // - 业务实体状态
    // - 业务操作日志
    // - 业务指标收集
}

// 3. 与MediatR生态系统集成
if (integrateWithMediatr)
{
    // - CQRS模式
    // - 领域驱动设计
    // - 事件溯源
}
```

### 6. **实际项目中如何结合使用**

```csharp
// 最佳实践：两者结合，各司其职
public class CombinedArchitecture
{
    /*
    请求处理流程：

    HTTP Request
        ↓
    ASP.NET Core Middleware (HTTP层面)
        ↓
    ASP.NET Core Action Filters (Action层面)
        ├── 1. 全局异常处理Filter
        ├── 2. 模型验证Filter
        ├── 3. 授权Filter
        ├── 4. 日志Filter
        └── 5. 响应包装Filter
        ↓
    Controller Action Method
        ↓
    MediatR.Send(command)  → 进入业务层
        ↓
    MediatR Pipeline Behaviors (业务层面)
        ├── 1. 业务日志Behavior
        ├── 2. 业务验证Behavior
        ├── 3. 业务事务Behavior
        ├── 4. 缓存Behavior
        └── 5. 业务处理器
        ↓
    返回业务结果
        ↓
    Controller返回ActionResult
        ↓
    经过Action Filters (结果处理)
        ↓
    返回HTTP响应
    */
}

// 具体代码示例
[ApiController]
[Route("api/orders")]
[ServiceFilter(typeof(LoggingActionFilter))]  // Action级别Filter
[ServiceFilter(typeof(ValidationActionFilter))]
public class OrdersController : ControllerBase
{
    private readonly IMediator _mediator;

    [HttpPost]
    [Authorize(Policy = "CanCreateOrder")]  // 授权Filter
    public async Task<IActionResult> CreateOrder([FromBody] CreateOrderRequest request)
    {
        // Action Filters已经处理了：
        // 1. 日志记录
        // 2. 模型验证
        // 3. 用户授权

        // 转换为业务命令
        var command = new CreateOrderCommand(
            request.ProductId,
            request.Quantity,
            User.Identity.Name
        );

        // 交给MediatR处理业务逻辑
        var result = await _mediator.Send(command);

        // 返回结果（会被ResponseFilter包装）
        return Ok(result);
    }
}

// Action Filter处理HTTP层面
public class LoggingActionFilter : IAsyncActionFilter
{
    public async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
    {
        var actionName = context.ActionDescriptor.DisplayName;
        var httpMethod = context.HttpContext.Request.Method;
        var path = context.HttpContext.Request.Path;

        Log.Information($"HTTP请求: {httpMethod} {path} -> {actionName}");

        await next();

        var statusCode = context.HttpContext.Response.StatusCode;
        Log.Information($"HTTP响应: {statusCode} <- {actionName}");
    }
}

// MediatR管道行为处理业务层面
public class BusinessTransactionBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
{
    public async Task<TResponse> Handle(TRequest request, RequestHandlerDelegate<TResponse> next, CancellationToken ct)
    {
        // 业务事务管理
        await using var transaction = await _dbContext.Database.BeginTransactionAsync(ct);

        try
        {
            var response = await next();
            await _dbContext.SaveChangesAsync(ct);
            await transaction.CommitAsync(ct);
            return response;
        }
        catch
        {
            await transaction.RollbackAsync(ct);
            throw;
        }
    }
}
```

## **总结**

**ASP.NET Core Action Filters** 和 **MediatR 管道行为** 确实非常相似，都提供了强大的 AOP 能力，但它们的**关注点不同**：

- **Action Filters** 关注 **HTTP/Action 层面的横切关注点**
- **MediatR 管道行为** 关注 **业务逻辑层面的横切关注点**

**简单记忆**：

- **Action Filters**：处理 **HTTP 请求怎么进入 Controller，结果怎么返回给客户端**
- **MediatR 管道行为**：处理 **业务命令怎么被处理，业务逻辑怎么执行**

在实际项目中，通常会**两者结合使用**：

- Action Filters 处理 **HTTP 协议相关** 的横切关注点
- MediatR 管道行为处理 **业务逻辑相关** 的横切关注点

这样既能保持关注点分离，又能充分利用两种机制的优点，构建出清晰、可维护的应用程序架构。
