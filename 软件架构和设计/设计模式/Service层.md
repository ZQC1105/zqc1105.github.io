好的，我来详细举例说明这两个层的职责分离：

## **Repository 层 - 只负责数据访问**

### 用户仓库示例：

```csharp
// Repositories/Interfaces/IUserRepository.cs
public interface IUserRepository
{
    // 纯粹的CRUD操作
    Task<User> GetByIdAsync(int id);
    Task<User> GetByEmailAsync(string email);
    Task<IEnumerable<User>> GetAllAsync();
    Task<IEnumerable<User>> GetByRoleAsync(string role);
    Task AddAsync(User user);
    Task UpdateAsync(User user);
    Task DeleteAsync(int id);
    Task<bool> ExistsAsync(int id);
    Task<bool> EmailExistsAsync(string email);

    // 分页查询
    Task<(IEnumerable<User> Users, int TotalCount)> GetPagedAsync(
        int pageNumber,
        int pageSize,
        string sortBy = "Id",
        bool ascending = true);
}

// Repositories/UserRepository.cs
public class UserRepository : IUserRepository
{
    private readonly ApplicationDbContext _context;

    public UserRepository(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<User> GetByIdAsync(int id)
    {
        // 纯粹的数据库查询，不包含业务逻辑
        return await _context.Users
            .AsNoTracking()
            .FirstOrDefaultAsync(u => u.Id == id);
    }

    public async Task<User> GetByEmailAsync(string email)
    {
        return await _context.Users
            .FirstOrDefaultAsync(u => u.Email == email);
    }

    public async Task<IEnumerable<User>> GetByRoleAsync(string role)
    {
        return await _context.Users
            .Where(u => u.Role == role)
            .OrderBy(u => u.LastName)
            .ToListAsync();
    }

    public async Task AddAsync(User user)
    {
        await _context.Users.AddAsync(user);
        await _context.SaveChangesAsync();
    }

    public async Task UpdateAsync(User user)
    {
        _context.Users.Update(user);
        await _context.SaveChangesAsync();
    }

    public async Task<(IEnumerable<User> Users, int TotalCount)> GetPagedAsync(
        int pageNumber,
        int pageSize,
        string sortBy = "Id",
        bool ascending = true)
    {
        var query = _context.Users.AsQueryable();

        // 排序逻辑
        query = sortBy switch
        {
            "Name" => ascending ? query.OrderBy(u => u.LastName) : query.OrderByDescending(u => u.LastName),
            "Email" => ascending ? query.OrderBy(u => u.Email) : query.OrderByDescending(u => u.Email),
            "CreatedAt" => ascending ? query.OrderBy(u => u.CreatedAt) : query.OrderByDescending(u => u.CreatedAt),
            _ => ascending ? query.OrderBy(u => u.Id) : query.OrderByDescending(u => u.Id)
        };

        var totalCount = await query.CountAsync();
        var users = await query
            .Skip((pageNumber - 1) * pageSize)
            .Take(pageSize)
            .ToListAsync();

        return (users, totalCount);
    }
}
```

### 产品仓库示例：

```csharp
// Repositories/ProductRepository.cs
public class ProductRepository : IProductRepository
{
    public async Task<Product> GetByIdWithCategoryAsync(int id)
    {
        // 包含关联数据查询
        return await _context.Products
            .Include(p => p.Category)
            .Include(p => p.Supplier)
            .FirstOrDefaultAsync(p => p.Id == id);
    }

    public async Task<IEnumerable<Product>> GetLowStockProductsAsync(int threshold = 10)
    {
        // 简单的过滤查询，仍然是数据访问职责
        return await _context.Products
            .Where(p => p.StockQuantity < threshold)
            .ToListAsync();
    }

    public async Task UpdateStockAsync(int productId, int quantityChange)
    {
        // 直接的数据更新操作
        var product = await _context.Products.FindAsync(productId);
        if (product != null)
        {
            product.StockQuantity += quantityChange;
            product.LastStockUpdated = DateTime.UtcNow;
            await _context.SaveChangesAsync();
        }
    }
}
```

## **Service 层 - 专注于业务规则和流程**

### 用户服务示例：

```csharp
// Services/UserService.cs
public class UserService : IUserService
{
    private readonly IUserRepository _userRepository;
    private readonly IPasswordHasher _passwordHasher;
    private readonly IEmailService _emailService;
    private readonly IAuditService _auditService;

    public UserService(
        IUserRepository userRepository,
        IPasswordHasher passwordHasher,
        IEmailService emailService,
        IAuditService auditService)
    {
        _userRepository = userRepository;
        _passwordHasher = passwordHasher;
        _emailService = emailService;
        _auditService = auditService;
    }

    // 注册用户 - 包含完整的业务逻辑
    public async Task<UserRegistrationResult> RegisterUserAsync(RegisterUserRequest request)
    {
        // 1. 业务规则验证：邮箱格式
        if (!IsValidEmail(request.Email))
            throw new BusinessException("邮箱格式不正确");

        // 2. 业务规则验证：密码强度
        if (!IsPasswordStrongEnough(request.Password))
            throw new BusinessException("密码必须包含至少8个字符，包括大小写字母和数字");

        // 3. 业务规则：检查邮箱是否已注册
        if (await _userRepository.EmailExistsAsync(request.Email))
            throw new BusinessException("该邮箱已被注册");

        // 4. 业务逻辑：年龄限制
        if (CalculateAge(request.BirthDate) < 18)
            throw new BusinessException("用户必须年满18岁");

        // 5. 业务处理：创建用户实体
        var user = new User
        {
            Email = request.Email,
            PasswordHash = _passwordHasher.Hash(request.Password), // 密码哈希是业务逻辑
            FirstName = request.FirstName,
            LastName = request.LastName,
            BirthDate = request.BirthDate,
            IsActive = true,
            CreatedAt = DateTime.UtcNow,
            EmailConfirmed = false,
            Role = "Customer" // 默认角色
        };

        // 6. 调用Repository进行数据保存
        await _userRepository.AddAsync(user);

        // 7. 业务操作：发送确认邮件
        var confirmationToken = GenerateEmailConfirmationToken();
        await _emailService.SendConfirmationEmailAsync(user.Email, confirmationToken);

        // 8. 业务操作：记录审计日志
        await _auditService.LogUserRegistrationAsync(user.Id, "用户注册成功");

        // 9. 返回业务结果
        return new UserRegistrationResult
        {
            UserId = user.Id,
            Email = user.Email,
            RequiresEmailConfirmation = true,
            WelcomeMessage = $"欢迎{user.FirstName}加入我们的平台！"
        };
    }

    // 重置密码 - 复杂的业务流程
    public async Task ResetPasswordAsync(ResetPasswordRequest request)
    {
        // 1. 验证重置令牌（业务规则）
        var user = await _userRepository.GetByIdAsync(request.UserId);
        if (user == null || user.PasswordResetToken != request.Token)
            throw new BusinessException("无效的重置令牌");

        // 2. 检查令牌是否过期（业务规则）
        if (user.PasswordResetTokenExpiry < DateTime.UtcNow)
            throw new BusinessException("重置令牌已过期");

        // 3. 密码策略验证（业务规则）
        if (request.NewPassword == request.ConfirmPassword)
            throw new BusinessException("两次输入的密码不一致");

        // 4. 密码历史检查（业务规则）
        if (await IsPasswordInHistoryAsync(user.Id, request.NewPassword))
            throw new BusinessException("不能使用最近用过的密码");

        // 5. 更新密码
        user.PasswordHash = _passwordHasher.Hash(request.NewPassword);
        user.PasswordResetToken = null;
        user.PasswordResetTokenExpiry = null;
        user.LastPasswordChange = DateTime.UtcNow;

        // 6. 保存到数据库
        await _userRepository.UpdateAsync(user);

        // 7. 发送通知邮件
        await _emailService.SendPasswordChangedNotificationAsync(user.Email);

        // 8. 安全审计
        await _auditService.LogPasswordResetAsync(user.Id, "密码重置成功");
    }
}
```

### 订单服务示例 - 复杂的业务流程：

```csharp
// Services/OrderService.cs
public class OrderService : IOrderService
{
    private readonly IOrderRepository _orderRepository;
    private readonly IProductRepository _productRepository;
    private readonly ICustomerRepository _customerRepository;
    private readonly IPaymentService _paymentService;
    private readonly IInventoryService _inventoryService;
    private readonly IShippingService _shippingService;
    private readonly INotificationService _notificationService;
    private readonly IDiscountCalculator _discountCalculator;

    public async Task<OrderResult> PlaceOrderAsync(PlaceOrderRequest request)
    {
        // 1. 验证客户存在（业务规则）
        var customer = await _customerRepository.GetByIdAsync(request.CustomerId);
        if (customer == null)
            throw new BusinessException("客户不存在");

        // 2. 验证每个产品（业务规则）
        foreach (var item in request.Items)
        {
            var product = await _productRepository.GetByIdAsync(item.ProductId);

            if (product == null)
                throw new BusinessException($"产品 {item.ProductId} 不存在");

            if (!product.IsActive)
                throw new BusinessException($"产品 {product.Name} 已下架");

            if (product.StockQuantity < item.Quantity)
                throw new BusinessException($"产品 {product.Name} 库存不足");

            // 3. 检查购买限制（业务规则）
            if (product.MaxPurchasePerCustomer > 0 &&
                item.Quantity > product.MaxPurchasePerCustomer)
                throw new BusinessException($"产品 {product.Name} 单次最多购买 {product.MaxPurchasePerCustomer} 件");
        }

        // 4. 计算价格（业务逻辑）
        var orderAmount = await CalculateOrderAmountAsync(request.Items);

        // 5. 应用折扣（业务逻辑）
        var discount = await _discountCalculator.CalculateDiscountAsync(
            customer.Id,
            request.Items,
            orderAmount);

        var finalAmount = orderAmount - discount;

        // 6. 处理支付（业务流程）
        var paymentResult = await _paymentService.ProcessPaymentAsync(
            new PaymentRequest
            {
                CustomerId = customer.Id,
                Amount = finalAmount,
                PaymentMethod = request.PaymentMethod
            });

        if (!paymentResult.Success)
            throw new BusinessException($"支付失败: {paymentResult.Message}");

        // 7. 创建订单
        var order = new Order
        {
            CustomerId = customer.Id,
            OrderDate = DateTime.UtcNow,
            Status = OrderStatus.Pending,
            TotalAmount = finalAmount,
            PaymentTransactionId = paymentResult.TransactionId
        };

        // 8. 保存订单
        await _orderRepository.AddAsync(order);

        // 9. 扣除库存（业务流程）
        await _inventoryService.ReserveInventoryAsync(request.Items);

        // 10. 安排发货（业务流程）
        var shippingInfo = await _shippingService.ScheduleDeliveryAsync(
            order.Id,
            request.ShippingAddress);

        // 11. 发送通知（业务操作）
        await _notificationService.SendOrderConfirmationAsync(customer.Email, order);

        // 12. 更新客户统计（业务逻辑）
        customer.TotalOrders++;
        customer.LastOrderDate = DateTime.UtcNow;
        await _customerRepository.UpdateAsync(customer);

        return new OrderResult
        {
            OrderId = order.Id,
            OrderNumber = GenerateOrderNumber(order.Id),
            Status = order.Status,
            EstimatedDelivery = shippingInfo.EstimatedDelivery,
            Message = "订单创建成功！我们将尽快处理您的订单。"
        };
    }

    private async Task<decimal> CalculateOrderAmountAsync(List<OrderItemRequest> items)
    {
        decimal total = 0;

        foreach (var item in items)
        {
            var product = await _productRepository.GetByIdAsync(item.ProductId);

            // 业务逻辑：促销价格计算
            decimal price = product.Price;

            if (product.IsOnSale && product.SalePrice.HasValue)
            {
                price = product.SalePrice.Value;
            }

            // 业务逻辑：批量折扣
            if (item.Quantity >= product.BulkDiscountThreshold)
            {
                price = price * (1 - product.BulkDiscountRate);
            }

            total += price * item.Quantity;
        }

        // 业务逻辑：增值税
        if (ShouldApplyTax(total))
        {
            total = total * 1.1m; // 10% 增值税
        }

        return total;
    }
}
```

## **关键区别总结**：

| **Repository**                     | **Service**                      |
| ---------------------------------- | -------------------------------- |
| `GetUserById(id)`                  | `RegisterUser(request)`          |
| `AddProduct(product)`              | `PlaceOrder(items)`              |
| `UpdateStock(productId, quantity)` | `ProcessCheckout(cart, payment)` |
| `FindUsersByRole(role)`            | `ApplyDiscount(user, order)`     |
| `DeleteOrder(orderId)`             | `CancelOrder(orderId, reason)`   |

**核心原则**：

- **Repository**: "如何获取/保存数据"（技术细节）
- **Service**: "数据应该满足什么业务规则"（业务逻辑）

**Service 层应该知道的，Repository 层不应该知道**：

- 密码策略
- 折扣计算规则
- 支付处理流程
- 邮件通知逻辑
- 审计日志要求
- 业务验证规则

**完全正确！** 这正是 Service 层的核心作用——协调多个 Repository 和其他服务来完成复杂的业务逻辑。

## Service 层依赖关系的几个典型场景：

### 1. **单个 Repository 依赖**（简单场景）

```csharp
public class UserService : IUserService
{
    private readonly IUserRepository _userRepository;
    // 只依赖一个Repository，处理单一实体的业务逻辑
}
```

### 2. **多个 Repository 依赖**（常见场景）

```csharp
public class OrderService : IOrderService
{
    private readonly IOrderRepository _orderRepository;
    private readonly IProductRepository _productRepository;
    private readonly ICustomerRepository _customerRepository;
    // 协调多个实体的业务逻辑
}
```

### 3. **Repository + 其他 Service 依赖**（复杂业务流程）

```csharp
public class CheckoutService : ICheckoutService
{
    // 依赖Repositories
    private readonly IOrderRepository _orderRepository;
    private readonly ICartRepository _cartRepository;

    // 依赖其他Services
    private readonly IPaymentService _paymentService;
    private readonly IShippingService _shippingService;
    private readonly IInventoryService _inventoryService;
    private readonly INotificationService _notificationService;
    private readonly IDiscountService _discountService;

    // 可能还依赖基础设施
    private readonly ILogger<CheckoutService> _logger;
    private readonly IAuditService _auditService;
}
```

## 具体示例：电子商务下单流程

```csharp
public class CheckoutService : ICheckoutService
{
    private readonly IOrderRepository _orderRepo;
    private readonly IProductRepository _productRepo;
    private readonly ICustomerRepository _customerRepo;
    private readonly ICartRepository _cartRepo;
    private readonly IPaymentService _paymentService;
    private readonly IShippingCalculator _shippingCalculator;
    private readonly ITaxService _taxService;
    private readonly IPromotionService _promotionService;
    private readonly IEmailService _emailService;
    private readonly IInventoryService _inventoryService;

    public CheckoutService(
        IOrderRepository orderRepo,
        IProductRepository productRepo,
        ICustomerRepository customerRepo,
        ICartRepository cartRepo,
        IPaymentService paymentService,
        IShippingCalculator shippingCalculator,
        ITaxService taxService,
        IPromotionService promotionService,
        IEmailService emailService,
        IInventoryService inventoryService)
    {
        // 这里注入了4个Repository和6个Service
        // Service负责协调它们完成复杂的下单流程
    }

    public async Task<CheckoutResult> ProcessCheckoutAsync(CheckoutRequest request)
    {
        // 1. 验证用户（使用CustomerRepository）
        var customer = await _customerRepo.GetByIdAsync(request.CustomerId);
        if (customer == null) throw new BusinessException("用户不存在");

        // 2. 获取购物车（使用CartRepository）
        var cart = await _cartRepo.GetCartWithItemsAsync(request.CartId);

        // 3. 验证库存（使用ProductRepository）
        foreach (var item in cart.Items)
        {
            var product = await _productRepo.GetByIdAsync(item.ProductId);
            if (product.Stock < item.Quantity)
                throw new BusinessException($"{product.Name}库存不足");
        }

        // 4. 计算商品总价（业务逻辑）
        var subtotal = CalculateSubtotal(cart.Items);

        // 5. 应用促销（使用PromotionService）
        var discount = await _promotionService.CalculateDiscountAsync(
            customer.Id, cart.Items, subtotal);

        // 6. 计算运费（使用ShippingCalculator）
        var shippingCost = await _shippingCalculator.CalculateAsync(
            request.ShippingAddress, cart.Items);

        // 7. 计算税费（使用TaxService）
        var tax = await _taxService.CalculateTaxAsync(
            subtotal - discount,
            request.ShippingAddress);

        // 8. 计算总金额（业务逻辑）
        var totalAmount = subtotal - discount + shippingCost + tax;

        // 9. 处理支付（使用PaymentService）
        var paymentResult = await _paymentService.ProcessPaymentAsync(
            new PaymentRequest
            {
                CustomerId = customer.Id,
                Amount = totalAmount,
                PaymentMethod = request.PaymentMethod
            });

        if (!paymentResult.Success)
            throw new BusinessException("支付失败");

        // 10. 创建订单（使用OrderRepository）
        var order = await CreateOrderAsync(customer, cart, totalAmount, paymentResult);

        // 11. 扣除库存（使用InventoryService）
        await _inventoryService.ReserveItemsAsync(cart.Items);

        // 12. 清空购物车（使用CartRepository）
        await _cartRepo.ClearCartAsync(request.CartId);

        // 13. 发送确认邮件（使用EmailService）
        await _emailService.SendOrderConfirmationAsync(customer.Email, order);

        // 14. 更新用户统计数据（使用CustomerRepository）
        await UpdateCustomerStatsAsync(customer.Id, totalAmount);

        return new CheckoutResult { OrderId = order.Id, Success = true };
    }
}
```

## Service 层的协调作用示意图：

```
[Controller]
    ↓
[CheckoutService] ← 业务逻辑协调者
    ├── [CustomerRepository] ← 获取用户数据
    ├── [CartRepository]    ← 获取购物车数据
    ├── [ProductRepository] ← 验证库存
    ├── [OrderRepository]   ← 保存订单
    ├── [PaymentService]    ← 处理支付
    ├── [ShippingService]   ← 计算运费
    ├── [TaxService]        ← 计算税费
    ├── [PromotionService]  ← 应用促销
    ├── [InventoryService]  ← 更新库存
    └── [EmailService]      ← 发送邮件
```

## 为什么这样设计？

### 1. **单一职责原则**

- Repository：只负责数据访问
- Service：只负责业务逻辑协调

### 2. **可测试性**

```csharp
// 可以轻松模拟所有依赖进行单元测试
[Test]
public async Task ProcessCheckout_Should_CreateOrder_When_PaymentSucceeds()
{
    // Arrange
    var mockOrderRepo = new Mock<IOrderRepository>();
    var mockPaymentService = new Mock<IPaymentService>();
    // ... 模拟所有依赖

    var service = new CheckoutService(
        mockOrderRepo.Object,
        mockPaymentService.Object,
        // ... 其他模拟对象
    );

    // Act
    var result = await service.ProcessCheckoutAsync(testRequest);

    // Assert
    mockOrderRepo.Verify(x => x.AddAsync(It.IsAny<Order>()), Times.Once);
}
```

### 3. **可维护性**

```csharp
// 如果需要添加新的业务规则，只需修改Service层
public async Task ProcessCheckoutAsync(CheckoutRequest request)
{
    // 新增业务规则：检查用户是否被限制购买
    if (await _fraudService.IsUserRestrictedAsync(request.CustomerId))
        throw new BusinessException("用户购买受限");

    // 新增业务规则：节假日特殊处理
    if (IsHolidaySeason())
        await ApplyHolidayDiscountAsync(cart);

    // 原有逻辑保持不变...
}
```

### 4. **事务管理**

```csharp
public async Task ProcessCheckoutAsync(CheckoutRequest request)
{
    // 使用事务确保多个Repository操作的原子性
    using var transaction = await _context.Database.BeginTransactionAsync();

    try
    {
        await _orderRepo.AddAsync(order);           // 保存订单
        await _inventoryRepo.UpdateStockAsync(items); // 更新库存
        await _paymentRepo.SaveTransactionAsync(payment); // 保存支付记录

        await transaction.CommitAsync();  // 所有操作成功才提交
    }
    catch
    {
        await transaction.RollbackAsync(); // 任何一个失败就回滚
        throw;
    }
}
```

## 总结

是的，**Service 层本质上是一个"协调者"或"编排者"**：

1. **依赖多个 Repository**：获取和保存不同实体的数据
2. **依赖其他 Service**：调用专业的业务服务
3. **包含核心业务逻辑**：实现具体的业务规则
4. **管理事务**：确保跨多个操作的原子性
5. **处理异常**：将技术异常转换为业务异常
6. **返回业务结果**：将数据封装为业务需要的格式

这种设计使得：

- **Controller 保持精简**：只处理 HTTP 相关的逻辑
- **Repository 保持纯粹**：只关注数据访问技术细节
- **Service 专注于业务**：实现复杂的业务规则和流程
- **代码更容易测试和维护**：各层职责清晰分离
