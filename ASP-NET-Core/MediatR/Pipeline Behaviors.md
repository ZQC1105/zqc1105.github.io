# **管道行为（Pipeline Behaviors）实现 AOP 的详细原理**

管道行为是 MediatR 实现 **AOP（面向切面编程）** 的核心机制。它允许你在不修改业务代码的情况下，向请求处理管道中注入横切关注点。

## 什么是 AOP（面向切面编程）？

**AOP 的核心思想**：将横切关注点（Cross-Cutting Concerns）从业务逻辑中分离出来。

### 传统方式的代码

```csharp
public class OrderService
{
    public async Task<OrderResult> ProcessOrder(Order order)
    {
        // 横切关注点 1：日志
        Log.Information($"开始处理订单 {order.Id}");
        var stopwatch = Stopwatch.StartNew();

        try
        {
            // 横切关注点 2：验证
            if (!IsValid(order))
                throw new ValidationException("订单无效");

            // 横切关注点 3：授权
            if (!HasPermission(order))
                throw new UnauthorizedException("无权限");

            // 横切关注点 4：事务
            using var transaction = BeginTransaction();

            try
            {
                // ⭐ 核心业务逻辑（被各种横切关注点包围）
                var result = await ProcessOrderCoreLogic(order);

                transaction.Commit();

                // 横切关注点 5：日志
                stopwatch.Stop();
                Log.Information($"订单处理完成，耗时 {stopwatch.ElapsedMilliseconds}ms");

                return result;
            }
            catch
            {
                transaction.Rollback();
                throw;
            }
        }
        catch (Exception ex)
        {
            // 横切关注点 6：异常处理
            Log.Error(ex, "订单处理失败");
            throw;
        }
    }

    // 真正的业务逻辑
    private async Task<OrderResult> ProcessOrderCoreLogic(Order order)
    {
        // 这里才是真正的业务代码
        return await _repository.SaveAsync(order);
    }
}
```

## MediatR 管道行为如何实现 AOP

### 1. **管道行为的运行原理**

```csharp
// MediatR 内部的管道调用链
public class Mediator
{
    public async Task<TResponse> Send<TResponse>(IRequest<TResponse> request, CancellationToken ct)
    {
        // 构建管道：行为按注册顺序包装
        RequestHandlerDelegate<TResponse> handler = () =>
            HandleRequestWithHandler(request, ct);

        // 反向包装：最后一个注册的行为最先执行
        foreach (var behavior in _behaviors.Reverse())
        {
            var currentHandler = handler;
            handler = () => behavior.Handle(request, currentHandler, ct);
        }

        // 执行管道
        return await handler();
    }
}

// 可视化管道结构
/*
执行顺序（假设注册了3个行为）：
LoggingBehavior
    ↓
ValidationBehavior
    ↓
TransactionBehavior
    ↓
业务处理器（真正的Handler）

调用过程：
1. 调用 LoggingBehavior.Handle()
2.  → 调用 ValidationBehavior.Handle()
3.     → 调用 TransactionBehavior.Handle()
4.        → 调用业务 Handler.Handle()
5.     ← TransactionBehavior 完成
6.  ← ValidationBehavior 完成
7. ← LoggingBehavior 完成
*/
```

### 2. **管道行为的代码实现细节**

```csharp
// 管道行为接口定义
public interface IPipelineBehavior<TRequest, TResponse>
{
    Task<TResponse> Handle(
        TRequest request,                    // 请求对象
        RequestHandlerDelegate<TResponse> next,  // 下一个行为或最终处理器的委托
        CancellationToken cancellationToken);
}

// 简单的管道行为实现
public class LoggingBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
{
    public async Task<TResponse> Handle(
        TRequest request,
        RequestHandlerDelegate<TResponse> next,
        CancellationToken cancellationToken)
    {
        Console.WriteLine($"开始处理 {typeof(TRequest).Name}");

        // 调用下一个行为或处理器
        var response = await next();

        Console.WriteLine($"完成处理 {typeof(TRequest).Name}");

        return response;
    }
}
```

### 3. **完整的 AOP 实现示例**

```csharp
// ============ 多个管道行为示例 ============

// 1. 日志行为
public class LoggingBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
{
    private readonly ILogger<LoggingBehavior<TRequest, TResponse>> _logger;

    public async Task<TResponse> Handle(
        TRequest request,
        RequestHandlerDelegate<TResponse> next,
        CancellationToken cancellationToken)
    {
        var requestName = typeof(TRequest).Name;
        var requestId = Guid.NewGuid();

        // 前置通知（Before Advice）
        _logger.LogInformation("[{RequestId}] 开始处理 {RequestName}", requestId, requestName);
        _logger.LogDebug("[{RequestId}] 请求数据: {@Request}", requestId, request);

        var stopwatch = Stopwatch.StartNew();

        try
        {
            // 调用链中的下一个处理器（可能是另一个行为或最终的Handler）
            var response = await next();
            stopwatch.Stop();

            // 后置通知（After Returning Advice）
            _logger.LogInformation("[{RequestId}] 成功处理 {RequestName}，耗时 {Elapsed}ms",
                requestId, requestName, stopwatch.ElapsedMilliseconds);

            return response;
        }
        catch (Exception ex)
        {
            stopwatch.Stop();

            // 异常通知（After Throwing Advice）
            _logger.LogError(ex, "[{RequestId}] 处理 {RequestName} 失败，耗时 {Elapsed}ms",
                requestId, requestName, stopwatch.ElapsedMilliseconds);

            throw;
        }
        finally
        {
            // 最终通知（After Finally Advice）
            _logger.LogDebug("[{RequestId}] {RequestName} 处理完成", requestId, requestName);
        }
    }
}

// 2. 验证行为
public class ValidationBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
    where TRequest : IRequest<TResponse>
{
    private readonly IEnumerable<IValidator<TRequest>> _validators;

    public async Task<TResponse> Handle(
        TRequest request,
        RequestHandlerDelegate<TResponse> next,
        CancellationToken cancellationToken)
    {
        // 前置验证
        if (_validators.Any())
        {
            var context = new ValidationContext<TRequest>(request);
            var validationResults = await Task.WhenAll(
                _validators.Select(v => v.ValidateAsync(context, cancellationToken)));
            var failures = validationResults.SelectMany(r => r.Errors).Where(f => f != null).ToList();

            if (failures.Count != 0)
                throw new ValidationException(failures);
        }

        // 验证通过，继续执行
        return await next();
    }
}

// 3. 事务行为
public class TransactionBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
    where TRequest : IRequest<TResponse>
{
    private readonly DbContext _dbContext;

    public async Task<TResponse> Handle(
        TRequest request,
        RequestHandlerDelegate<TResponse> next,
        CancellationToken cancellationToken)
    {
        var strategy = _dbContext.Database.CreateExecutionStrategy();

        return await strategy.ExecuteAsync(async () =>
        {
            await using var transaction = await _dbContext.Database.BeginTransactionAsync(cancellationToken);

            try
            {
                // 执行实际业务逻辑
                var response = await next();

                // 提交事务
                await _dbContext.SaveChangesAsync(cancellationToken);
                await transaction.CommitAsync(cancellationToken);

                return response;
            }
            catch
            {
                // 回滚事务
                await transaction.RollbackAsync(cancellationToken);
                throw;
            }
        });
    }
}

// 4. 性能监控行为
public class MetricsBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
{
    public async Task<TResponse> Handle(
        TRequest request,
        RequestHandlerDelegate<TResponse> next,
        CancellationToken cancellationToken)
    {
        var requestType = typeof(TRequest).Name;

        // 开始计时
        var timer = Metrics.Timer(requestType).NewTimer();

        try
        {
            var response = await next();

            // 记录成功指标
            Metrics.Meter(requestType, "success").Mark();
            timer.Record();

            return response;
        }
        catch
        {
            // 记录失败指标
            Metrics.Meter(requestType, "failure").Mark();
            throw;
        }
    }
}

// 5. 缓存行为
public class CachingBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
    where TRequest : ICacheableRequest
{
    private readonly IDistributedCache _cache;

    public async Task<TResponse> Handle(
        TRequest request,
        RequestHandlerDelegate<TResponse> next,
        CancellationToken cancellationToken)
    {
        // 检查缓存
        var cacheKey = request.GetCacheKey();
        var cachedData = await _cache.GetAsync<TResponse>(cacheKey, cancellationToken);

        if (cachedData != null)
        {
            // 返回缓存数据（方法环绕通知）
            return cachedData;
        }

        // 执行实际处理
        var response = await next();

        // 缓存结果
        await _cache.SetAsync(cacheKey, response,
            new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5)
            },
            cancellationToken);

        return response;
    }
}
```

## 4. **AOP 的不同通知类型在管道中的体现**

```csharp
// 传统的AOP通知类型在管道行为中的对应关系
public class CompleteAopBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
{
    public async Task<TResponse> Handle(
        TRequest request,
        RequestHandlerDelegate<TResponse> next,
        CancellationToken cancellationToken)
    {
        // ============ Before Advice (前置通知) ============
        Console.WriteLine("Before: 请求开始");
        var startTime = DateTime.UtcNow;

        try
        {
            // ============ Around Advice (环绕通知) ============
            Console.WriteLine("Around: 准备调用业务逻辑");

            // 这里可以决定是否调用业务逻辑
            if (ShouldSkip(request))
            {
                // 不调用next()，直接返回
                return default;
            }

            // 调用业务逻辑
            var response = await next();  // 这里就是业务逻辑的执行点

            Console.WriteLine("Around: 业务逻辑执行完成");
            // ============ Around Advice 结束 ============

            // ============ After Returning Advice (返回后通知) ============
            Console.WriteLine("AfterReturning: 业务逻辑成功返回");
            LogSuccess(request, response, startTime);

            return response;
        }
        catch (Exception ex)
        {
            // ============ After Throwing Advice (异常通知) ============
            Console.WriteLine("AfterThrowing: 业务逻辑抛出异常");
            LogFailure(request, ex, startTime);

            throw;
        }
        finally
        {
            // ============ After (Finally) Advice (最终通知) ============
            Console.WriteLine("After: 无论成功失败都会执行");
            CleanupResources();
        }
    }
}
```

## 5. **管道行为的配置和注册**

```csharp
// 注册管道行为（顺序很重要！）
services.AddTransient(typeof(IPipelineBehavior<,>), typeof(LoggingBehavior<,>));
services.AddTransient(typeof(IPipelineBehavior<,>), typeof(ValidationBehavior<,>));
services.AddTransient(typeof(IPipelineBehavior<,>), typeof(AuthorizationBehavior<,>));
services.AddTransient(typeof(IPipelineBehavior<,>), typeof(TransactionBehavior<,>));
services.AddTransient(typeof(IPipelineBehavior<,>), typeof(CachingBehavior<,>));

// 执行顺序可视化：
/*
请求进入
    ↓
LoggingBehavior（记录开始）
    ↓
ValidationBehavior（验证输入）
    ↓
AuthorizationBehavior（权限检查）
    ↓
TransactionBehavior（开始事务）
    ↓
CachingBehavior（检查缓存）
    ↓
实际的业务处理器（Handler）
    ↓
CachingBehavior（缓存结果）
    ↓
TransactionBehavior（提交事务）
    ↓
AuthorizationBehavior（清理权限）
    ↓
ValidationBehavior（后验证）
    ↓
LoggingBehavior（记录完成）
    ↓
返回响应
*/
```

## 6. **条件性管道行为**

```csharp
// 根据条件应用不同的管道行为
public class ConditionalPipelineBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
{
    public async Task<TResponse> Handle(
        TRequest request,
        RequestHandlerDelegate<TResponse> next,
        CancellationToken cancellationToken)
    {
        // 根据请求类型决定应用哪些AOP逻辑
        if (request is ICacheableRequest cacheableRequest)
        {
            // 应用缓存逻辑
            return await HandleWithCache(cacheableRequest, next, cancellationToken);
        }

        if (request is ISecuredRequest securedRequest)
        {
            // 应用安全逻辑
            return await HandleWithSecurity(securedRequest, next, cancellationToken);
        }

        // 默认处理
        return await next();
    }
}

// 或者使用特性标记
[AttributeUsage(AttributeTargets.Class)]
public class TransactionalAttribute : Attribute { }

public class TransactionalBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
{
    public async Task<TResponse> Handle(
        TRequest request,
        RequestHandlerDelegate<TResponse> next,
        CancellationToken cancellationToken)
    {
        // 检查请求类型是否有Transactional特性
        if (typeof(TRequest).GetCustomAttribute<TransactionalAttribute>() != null)
        {
            // 只有标记了Transactional的特性才应用事务
            return await ExecuteInTransaction(request, next, cancellationToken);
        }

        // 否则直接执行
        return await next();
    }
}
```

## 7. **实际业务代码的对比**

### 没有 AOP 的传统方式

```csharp
public class OrderService
{
    public async Task<OrderResult> PlaceOrder(PlaceOrderRequest request)
    {
        // 1. 验证
        Validate(request);

        // 2. 授权检查
        CheckAuthorization(request.UserId);

        // 3. 开始事务
        using var transaction = BeginTransaction();

        try
        {
            // 4. 业务逻辑
            var order = CreateOrder(request);
            await SaveOrder(order);

            // 5. 发布事件
            await PublishOrderCreatedEvent(order);

            // 6. 提交事务
            transaction.Commit();

            // 7. 记录成功日志
            LogSuccess(order);

            return new OrderResult(order.Id);
        }
        catch (Exception ex)
        {
            // 8. 回滚事务
            transaction.Rollback();

            // 9. 记录失败日志
            LogFailure(ex);

            throw;
        }
    }
}
```

### 使用 MediatR 管道行为的 AOP 方式

```csharp
// 业务处理器变得非常简洁
public class PlaceOrderHandler : IRequestHandler<PlaceOrderCommand, OrderResult>
{
    public async Task<OrderResult> Handle(PlaceOrderCommand command, CancellationToken ct)
    {
        // ⭐ 只关注核心业务逻辑！
        var order = Order.Create(command);
        await _orderRepository.AddAsync(order, ct);
        await _eventPublisher.PublishAsync(new OrderCreatedEvent(order), ct);

        return new OrderResult(order.Id);
    }
}

// 所有横切关注点都在管道行为中
/*
PlaceOrderCommand
    ↓→ [ValidationBehavior] 验证
    ↓→ [AuthorizationBehavior] 权限检查
    ↓→ [LoggingBehavior] 记录开始
    ↓→ [TransactionBehavior] 开始事务
    ↓→ PlaceOrderHandler（核心业务）
    ↓→ [TransactionBehavior] 提交事务
    ↓→ [EventPublishingBehavior] 发布事件
    ↓→ [LoggingBehavior] 记录完成
    ↓→ OrderResult
*/
```

## 总结：管道行为如何实现 AOP

1. **责任链模式**：管道行为基于责任链模式，每个行为处理请求并传递给下一个

2. **装饰器模式**：每个行为都包装了下一个处理器，可以添加额外的功能

3. **AOP 通知的完整支持**：

   - **Before**：在 `await next()` 之前执行
   - **After Returning**：在 `await next()` 之后，返回之前执行
   - **After Throwing**：通过 try-catch 实现
   - **After (Finally)**：通过 try-finally 实现
   - **Around**：可以控制是否调用 `next()`，或修改输入输出

4. **关注点分离**：

   - 业务代码只关注业务逻辑
   - 横切关注点（日志、验证、事务等）在管道行为中处理

5. **可重用性**：管道行为可以应用于所有请求，无需在每个处理器中重复代码

**这才是 MediatR 真正的威力所在**：通过管道行为实现全面的 AOP，让业务代码保持干净、专注，同时拥有完整的横切关注点支持。
