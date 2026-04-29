---
name: csharp-development
description: Enforce C# coding standards and best practices for .NET applications.
license: Proprietary
compatibility: opencode
metadata:
  audience: contributors
  focus: csharp
---

## What I do

- Enforce mandatory C# coding conventions and best practices.
- Review new and modified C# code for style, structure, and maintainability.
- Provide compliant examples and direct fixes when code violates standards.

## When to use me

Use this skill whenever you create, modify, or review C# code in this repository.

## Core principles

### Zero warnings policy

The codebase must have zero compiler warnings at all times.

Rules:
- All code must compile without warnings.
- Fix warnings immediately; do not suppress them unless absolutely necessary.
- If suppression is required, document the reason with a comment.
- Treat warnings as errors in CI/CD pipelines.

### Boy Scout principle

Always leave the code better than you found it.

Rules:
- When touching existing code, improve it incrementally.
- Fix nearby code smells, outdated patterns, or technical debt when reasonable.
- Refactor for clarity, even if not directly related to your task.
- Update outdated comments and documentation.
- Remove unused code, imports, and variables.

Examples of Boy Scout improvements:
- Rename unclear variables while implementing a feature.
- Extract long methods into smaller, more focused methods.
- Add missing XML documentation to public APIs.
- Replace magic numbers with named constants.
- Update deprecated API usage to modern alternatives.

Do not:
- Make unrelated changes that obscure the main purpose of your work.
- Refactor entire modules when making a small fix.
- Introduce breaking changes without discussion.

The Boy Scout principle applies to every change, no matter how small.

## Naming conventions

### PascalCase for all public members

Always use PascalCase for public and protected members.

Rules:
- Classes: `CustomerService`, `OrderRepository`
- Interfaces: `IOrderService`, `ICustomerRepository`
- Methods: `GetCustomerById`, `ProcessOrder`
- Properties: `FirstName`, `TotalAmount`, `IsActive`
- Public fields (avoid if possible): `DefaultTimeout`
- Enums: `OrderStatus`, `UserRole`
- Enum values: `OrderStatus.Pending`, `UserRole.Administrator`
- Constants: `MaxRetryCount`, `DefaultPageSize`
- Events: `OrderCreated`, `CustomerUpdated`

### camelCase for private members

Use camelCase with leading underscore for private fields:
- Private fields: `_orderRepository`, `_logger`, `_httpClient`
- Method parameters: `customerId`, `orderDate`, `isActive`
- Local variables: `totalAmount`, `customerName`, `result`

### Complete naming examples

```csharp
public class OrderService : IOrderService
{
    private readonly IOrderRepository _orderRepository;
    private readonly ILogger<OrderService> _logger;
    private const int MaxRetryCount = 3;
    
    public OrderService(IOrderRepository orderRepository, ILogger<OrderService> logger)
    {
        _orderRepository = orderRepository;
        _logger = logger;
    }
    
    public async Task<Order?> GetOrderByIdAsync(string orderId, CancellationToken cancellationToken)
    {
        var order = await _orderRepository.FindByIdAsync(orderId, cancellationToken);
        return order;
    }
    
    public string CustomerName { get; set; }
    public bool IsActive { get; set; }
}

public enum OrderStatus
{
    Pending,
    Confirmed,
    Shipped,
    Delivered,
    Cancelled
}
```

## Global usings

Use a `Globals.cs` file with global usings when many files share the same using statements.

Rules:
- Create `Globals.cs` at the project root.
- Use `global using` directives for commonly used namespaces.
- Keep `Globals.cs` organized and sorted alphabetically.
- Remove redundant local usings from individual files once they are in `Globals.cs`.

Example `Globals.cs`:

```csharp
global using System;
global using System.Collections.Generic;
global using System.Linq;
global using System.Threading;
global using System.Threading.Tasks;
global using Microsoft.Extensions.DependencyInjection;
global using Microsoft.Extensions.Logging;
```

When to use global usings:
- Namespaces used in more than 30% of files in the project.
- Framework namespaces that are ubiquitous (System, System.Linq, System.Threading.Tasks).
- Project-wide infrastructure namespaces (logging, DI, configuration).

When NOT to use global usings:
- Feature-specific namespaces used in only a few files.
- Third-party library namespaces that may change.
- Namespaces that might create ambiguity or naming conflicts.

### Global using static for tests

In test projects, use `global using static System.Net.HttpStatusCode` to make HTTP status code assertions more readable.

Add to test project `Globals.cs`:

```csharp
global using static System.Net.HttpStatusCode;
```

This eliminates the need to qualify status codes with `HttpStatusCode.` in test assertions.

Before:

```csharp
response.StatusCode.ShouldBe(HttpStatusCode.BadRequest);
response.StatusCode.ShouldBe(HttpStatusCode.NotFound);
```

After:

```csharp
response.StatusCode.ShouldBe(BadRequest);
response.StatusCode.ShouldBe(NotFound);
```

Benefits:
- Reduces visual noise in test assertions.
- Makes test expectations more readable and scannable.
- Follows the principle of making tests read like specifications.

## Code organization

### No regions

Do not use `#region` directives to organize code.

Rules:
- Never use `#region` / `#endregion`.
- If code needs organization, refactor it into smaller classes or methods.
- Use proper class design and separation of concerns instead.

Why regions are problematic:
- They hide code complexity instead of addressing it.
- They encourage large, unfocused classes.
- They make code harder to navigate and understand.
- Modern IDEs provide better ways to collapse and navigate code.

Bad example:

```csharp
public class OrderService
{
    #region Fields
    private readonly IOrderRepository _orderRepository;
    private readonly ILogger _logger;
    #endregion
    
    #region Constructor
    public OrderService(IOrderRepository orderRepository, ILogger logger)
    {
        _orderRepository = orderRepository;
        _logger = logger;
    }
    #endregion
    
    #region Public Methods
    public void CreateOrder() { }
    public void UpdateOrder() { }
    #endregion
    
    #region Private Methods
    private void ValidateOrder() { }
    private void SaveOrder() { }
    #endregion
}
```

Good example (no regions needed):

```csharp
public class OrderService
{
    private readonly IOrderRepository _orderRepository;
    private readonly ILogger _logger;
    
    public OrderService(IOrderRepository orderRepository, ILogger logger)
    {
        _orderRepository = orderRepository;
        _logger = logger;
    }
    
    public void CreateOrder() { }
    public void UpdateOrder() { }
    
    private void ValidateOrder() { }
    private void SaveOrder() { }
}
```

If a class needs regions to be "organized," it's a sign the class is doing too much and should be split into smaller, focused classes.

### Using directives

Using directives must be in a single block, sorted alphabetically.

Rules:
- Keep all `using` statements together in one block.
- Sort alphabetically (System namespaces first, then third-party, then project namespaces).
- Do not group or separate usings with blank lines.
- Remove unused usings.

Bad example (separated groups, not sorted):

```csharp
using System.Linq;
using System;

using Microsoft.AspNetCore.Mvc;

using MyProject.Services;
using MyProject.Models;
```

Good example (single block, alphabetically sorted):

```csharp
using System;
using System.Linq;
using Microsoft.AspNetCore.Mvc;
using MyProject.Models;
using MyProject.Services;
```

Note: If using `Globals.cs` with global usings, only include file-specific usings in individual files.

### Fluent API formatting

When an interface supports fluent method chaining, use it with proper formatting.

Rules:
- Chain multiple method calls when the API supports it.
- Place each method call on a new line.
- Indent chained methods to show continuation.
- The object/variable starts the chain on its own line.
- End with semicolon on the last method call.

Bad example (separate statements):

```csharp
builder.UseSetting("VAULT_URL", "");
builder.UseEnvironment("Development");
builder.ConfigureLogging();
```

Good example (fluent chain):

```csharp
builder
    .UseSetting("VAULT_URL", "")
    .UseEnvironment("Development")
    .ConfigureLogging();
```

More examples:

```csharp
// Service registration
services
    .AddScoped<IOrderService, OrderService>()
    .AddScoped<ICustomerService, CustomerService>()
    .AddTransient<IEmailSender, EmailSender>();

// LINQ queries
var results = customers
    .Where(c => c.IsActive)
    .OrderBy(c => c.Name)
    .Select(c => c.ToDto())
    .ToList();

// String builder
var message = new StringBuilder()
    .AppendLine("Order Details:")
    .AppendLine($"Order ID: {orderId}")
    .AppendLine($"Total: {total}")
    .ToString();
```

Benefits:
- Improves readability and scanning.
- Shows clear sequence of operations.
- Easier to add or remove steps.
- Reduces horizontal scrolling.

### Namespace and folder structure alignment

Namespaces must follow the folder structure of the project.

Rules:
- The namespace must mirror the directory path.
- Use the project root as the namespace root.
- Keep namespace and folder structure in sync.
- Exceptions are rare and must be justified.

Folder structure:

```text
MyProject/
  Features/
    Orders/
      OrderService.cs
      OrderRepository.cs
    Customers/
      CustomerService.cs
```

Corresponding namespaces:

```csharp
// File: Features/Orders/OrderService.cs
namespace MyProject.Features.Orders;

public class OrderService { }
```

```csharp
// File: Features/Customers/CustomerService.cs
namespace MyProject.Features.Customers;

public class CustomerService { }
```

Allowed exceptions:
- Extension methods that logically belong to a common namespace for discoverability.
- Shared types that are consumed across many features and benefit from a shorter namespace.

Example exception (extension methods):

```csharp
// File: Features/Orders/Extensions/StringExtensions.cs
namespace MyProject.Extensions; // Exception: common namespace for extensions

public static class StringExtensions
{
    public static string ToOrderNumber(this string value) => $"ORD-{value}";
}
```

When making exceptions:
- Document the reason in a comment at the namespace declaration.
- Ensure the exception provides clear value (discoverability, reduced coupling).
- Keep exceptions minimal and consistent.

## Default values over null

Prefer default values over nullable parameters when a sensible default exists.

Rules:
- Use default parameter values instead of nullable parameters with null checks.
- Specify the default value directly in the method signature.
- Only use nullable parameters when null has a distinct meaning or when no reasonable default exists.

Bad example (nullable parameter with null check):

```csharp
public void UpdateRole(Role? role = null)
{
    if (role == null)
    {
        role = Role.Default;
    }
    
    // Use role...
}
```

Good example (default parameter value):

```csharp
public void UpdateRole(Role role = Role.Default)
{
    // Use role directly...
}
```

When to use nullable parameters:
- When null has a distinct semantic meaning (e.g., "do not update this field").
- When there is no reasonable default value.
- When the caller must explicitly decide what value to provide.

Example where null is meaningful:

```csharp
// null means "don't update the email"
public void UpdateUser(string userId, string? email = null)
{
    var user = GetUser(userId);
    
    if (email != null)
    {
        user.Email = email;
    }
    
    Save(user);
}
```

## Exception handling and control flow

Avoid using exceptions for control flow. Prefer explicit return types that communicate success or failure.

Rules:
- Do not throw exceptions for expected conditions or business rule violations.
- Use result types (e.g., `Result<T>`, `OneOf<T, TError>`, or nullable types) to represent success/failure outcomes.
- Reserve exceptions for truly exceptional situations (infrastructure failures, unexpected states).
- Let unhandled exceptions flow up to the controller level.

Exception handling at controller level:
- Handle exceptions at the controller level using middleware or exception filters.
- Return appropriate HTTP status codes with `ProblemDetails`.
- Use standardized error responses for consistency.

Bad example (control flow via exceptions):

```csharp
public Order GetOrder(string orderId)
{
    var order = _repository.FindById(orderId);
    if (order == null)
    {
        throw new OrderNotFoundException(orderId); // Bad: using exception for control flow
    }
    return order;
}
```

Good example (explicit return type):

```csharp
public Order? GetOrder(string orderId)
{
    return _repository.FindById(orderId); // Caller checks for null
}

// Or with result type:
public Result<Order> GetOrder(string orderId)
{
    var order = _repository.FindById(orderId);
    return order != null 
        ? Result<Order>.Success(order)
        : Result<Order>.Failure("Order not found");
}
```

### Result type for flow control

If needed, you may add a small `Result<T, TError>` type to handle flow control rather than throwing exceptions.

Implementation:

```csharp
public readonly struct Result<T, TError> where TError : notnull
{
    private readonly T _value;
    private readonly TError _error;

    public bool IsSuccess { get; }
    public bool IsFailure => !IsSuccess;

    private Result(bool isSuccess, T value, TError error)
    {
        IsSuccess = isSuccess;
        _value = value;
        _error = error;
    }

    public static Result<T, TError> Success(T value) 
        => new(true, value, default!);
    
    public static Result<T, TError> Failure(TError error) 
        => new(false, default!, error);

    public T Value => _value;
    public TError Error => _error;

    public T ValueOr(T fallback) => IsSuccess ? _value : fallback;

    public static implicit operator Result<T, TError>(T value) => Success(value);
}
```

Usage examples:

```csharp
// With string errors
public Result<Order, string> GetOrder(string id)
{
    var order = _repository.FindById(id);
    return order != null 
        ? Result<Order, string>.Success(order)
        : Result<Order, string>.Failure("Order not found");
}

// With enum error codes
public enum ValidationError { Required, TooShort, InvalidFormat }

public Result<User, ValidationError> ValidateUser(string name)
{
    if (string.IsNullOrEmpty(name))
        return Result<User, ValidationError>.Failure(ValidationError.Required);
    
    return Result<User, ValidationError>.Success(new User(name));
}

// With exceptions (when catching and converting)
public Result<string, Exception> ReadFile(string path)
{
    try
    {
        return Result<string, Exception>.Success(File.ReadAllText(path));
    }
    catch (Exception ex)
    {
        return Result<string, Exception>.Failure(ex);
    }
}
```

## Controller conventions

All controllers and endpoints must follow these conventions:

Required attributes:
- Add a description to controllers using `[ApiController]` and XML comments or attributes.
- Decorate controllers or individual endpoints with Swagger response attributes.
- Always include `ProblemDetails` response for error cases.

Required Swagger response attribute:

```csharp
[SwaggerResponse(StatusCodes.Status400BadRequest, type: typeof(ProblemDetails), contentTypes: "application/problem+json")]
```

This attribute can be applied at:
- Controller class level (applies to all endpoints).
- Individual endpoint method level (specific to that endpoint).

Example controller with class-level attributes:

```csharp
[ApiController]
[Route("api/[controller]")]
[SwaggerResponse(StatusCodes.Status400BadRequest, type: typeof(ProblemDetails), contentTypes: "application/problem+json")]
[SwaggerResponse(StatusCodes.Status500InternalServerError, type: typeof(ProblemDetails), contentTypes: "application/problem+json")]
public class OrdersController : ControllerBase
{
    private readonly IOrderService _orderService;
    
    public OrdersController(IOrderService orderService)
    {
        _orderService = orderService;
    }

    /// <summary>
    /// Retrieves an order by ID.
    /// </summary>
    [HttpGet("{id}")]
    [SwaggerResponse(StatusCodes.Status200OK, type: typeof(OrderDto))]
    [SwaggerResponse(StatusCodes.Status404NotFound, type: typeof(ProblemDetails), contentTypes: "application/problem+json")]
    public async Task<ActionResult<OrderDto>> GetOrder(string id)
    {
        var order = await _orderService.GetOrderAsync(id);
        
        if (order == null)
        {
            return NotFound(new ProblemDetails
            {
                Status = StatusCodes.Status404NotFound,
                Title = "Order not found",
                Detail = $"No order found with ID: {id}"
            });
        }
        
        return Ok(order);
    }
}
```

Example endpoint with method-level attributes:

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    /// <summary>
    /// Creates a new order.
    /// </summary>
    [HttpPost]
    [SwaggerResponse(StatusCodes.Status201Created, type: typeof(OrderDto))]
    [SwaggerResponse(StatusCodes.Status400BadRequest, type: typeof(ProblemDetails), contentTypes: "application/problem+json")]
    public async Task<ActionResult<OrderDto>> CreateOrder([FromBody] CreateOrderRequest request)
    {
        var result = await _orderService.CreateOrderAsync(request);
        
        if (!result.IsSuccess)
        {
            return BadRequest(new ProblemDetails
            {
                Status = StatusCodes.Status400BadRequest,
                Title = "Invalid order",
                Detail = result.ErrorMessage
            });
        }
        
        return CreatedAtAction(nameof(GetOrder), new { id = result.Value.Id }, result.Value);
    }
}
```

## Enforcement

- Use PR checklists that reference this skill.
- Enforce in code review.
- Ensure `Globals.cs` is maintained as the project evolves.