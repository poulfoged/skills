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

## .NET 10 project conventions

For all .NET 10 projects in this repository, apply the following setup.

### global.json

Create `global.json` at the repository root:

```json
{
  "sdk": {
    "rollForward": "latestMinor",
    "version": "8.0.100"
  }
}
```

### Central package management

Create `src/Directory.Packages.props` with central package versioning:

```xml
<Project>
  <PropertyGroup>
    <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
    <CentralPackageTransitivePinningEnabled>true</CentralPackageTransitivePinningEnabled>
  </PropertyGroup>
  <ItemGroup>
    <!-- All package versions declared here only -->
  </ItemGroup>
</Project>
```

Rules:
- All package `Version` attributes are declared exclusively in `Directory.Packages.props`.
- Remove all `Version="..."` attributes from `PackageReference` entries in all `.csproj` files.
- Add or update versions only in `Directory.Packages.props`.

### Directory.Build.props

Create `src/Directory.Build.props` with common project properties:

```xml
<Project>
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <LangVersion>preview</LangVersion>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
    <NuGetAudit>true</NuGetAudit>
  </PropertyGroup>
</Project>
```

### OpenAPI with Scalar + Swagger UI

Use `Microsoft.AspNetCore.OpenApi` for OpenAPI spec generation.

Serve both Scalar and Swagger UI simultaneously as frontends — non-production only:

```csharp
if (!app.Environment.IsProduction())
{
    app.MapOpenApi();
    app.MapScalarApiReference();
    app.MapSwaggerUi();
}
```

### Testing

Use `xunit.v3.mtp-v2` as the test framework.

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

### No abbreviated names

Always use full, descriptive names for variables, parameters, and fields. Never shorten names for brevity.

Rules:
- Write out the full word or phrase that describes the concept.
- Do not use single-letter names, acronyms, or truncations except for universally understood loop counters (`i`, `j`) in trivial loops.
- This applies to parameters, locals, fields, and type parameters.

Bad examples:

```csharp
CancellationToken ct
HttpClient hc
IOrderRepository repo
ILogger l
string usr
```

Good examples:

```csharp
CancellationToken cancellationToken
HttpClient httpClient
IOrderRepository orderRepository
ILogger logger
string username
```

### Storage-agnostic naming for stores

Never name a class after the storage technology it uses. Classes like `UserStore`, `OrderRepository`, or `ProductStore` should reflect the domain concept, not the underlying infrastructure.

Bad examples:

```csharp
public class SqlServerUserStore : IUserStore { }
public class ElasticsearchOrderRepository : IOrderRepository { }
public class MongoDbProductStore : IProductStore { }
public class RedisCacheStore : ICacheStore { }
```

Good examples:

```csharp
public class UserStore : IUserStore { }
public class OrderRepository : IOrderRepository { }
public class ProductStore : IProductStore { }
public class CacheStore : ICacheStore { }
```

If you need to differentiate implementations (e.g., for testing), use the technology name in the namespace, not the class name:

```csharp
// File: Infrastructure/Stores/SqlServer/CacheStore.cs
namespace MyProject.Infrastructure.Stores.SqlServer;

public class CacheStore : ICacheStore { }
```

Rules:
- The class name must describe only the domain concept (`UserStore`, `OrderRepository`).
- Storage technology (`SqlServer`, `Elasticsearch`, `Redis`, `MongoDb`) goes in the namespace or project folder, never the type name.
- Test doubles follow the same rule: `FakeUserStore`, `InMemoryOrderRepository`, `SpyCacheStore` — not `FakeSqlServerUserStore`.

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

## Regular expressions — always include a timeout

All `Regex` usage must specify an explicit timeout to prevent catastrophic backtracking and ReDoS attacks.

Rules:
- Always pass a `TimeSpan` timeout to `Regex` constructor or static methods.
- When in doubt, use `TimeSpan.FromSeconds(1)`.
- Use `RegexOptions.NonBacktracking` when available (.NET 7+) as a safer alternative.
- Store frequently used patterns as compiled `static` `Regex` instances with timeout.

Bad (no timeout):

```csharp
if (Regex.IsMatch(input, @"^[a-z]+$"))
{
    // ...
}
```

Good (explicit timeout):

```csharp
if (Regex.IsMatch(input, @"^[a-z]+$", RegexOptions.None, TimeSpan.FromSeconds(1)))
{
    // ...
}
```

Better (compiled static instance with timeout):

```csharp
private static readonly Regex ValidNamePattern = new(
    @"^[a-z]+$",
    RegexOptions.Compiled | RegexOptions.CultureInvariant,
    TimeSpan.FromSeconds(1));

public bool IsValidName(string input) => ValidNamePattern.IsMatch(input);
```

Best (non-backtracking engine):

```csharp
private static readonly Regex ValidNamePattern = new(
    @"^[a-z]+$",
    RegexOptions.Compiled | RegexOptions.NonBacktracking | RegexOptions.CultureInvariant,
    TimeSpan.FromSeconds(1));
```

Do not:
- Use `Regex.Initialize` or `Regex.CompileToAssembly` (deprecated).
- Omit the timeout argument.
- Use `Timeout.Infinite` unless the pattern is proven safe by construction.

## Pattern matching over null checks

Use `is not null` and `is null` instead of `!= null` and `== null`.

Rules:
- Prefer `is null` / `is not null` for all null checks.
- Use pattern matching (`is`, `is not`, `switch` expressions) over equality operators wherever it improves clarity.
- Extend this to type checks: prefer `if (x is OrderDto dto)` over `if (x is OrderDto) { var dto = (OrderDto)x; }`.

Bad example:

```csharp
if (descriptorToRemove != null)
{
    Remove(descriptorToRemove);
}
```

Good example:

```csharp
if (descriptorToRemove is not null)
{
    Remove(descriptorToRemove);
}
```

More examples:

```csharp
// Null check
if (order is null) return;

// Type pattern
if (result is ValidationError error)
{
    return BadRequest(error.Message);
}

// Switch expression
var label = status switch
{
    OrderStatus.Pending   => "Awaiting confirmation",
    OrderStatus.Shipped   => "On the way",
    OrderStatus.Delivered => "Delivered",
    _                     => "Unknown"
};
```

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

## Early returns

Always prefer early returns to reduce nesting and improve readability. Exit methods as soon as conditions are met.

Rules:
- Return early when failure conditions are met.
- Avoid wrapping the entire method body in an if statement when the method ends with it.
- Invert conditions to enable early returns when the negative case should exit.
- Reduce indentation by handling edge cases first.

Bad example (nested if at the end):

```csharp
public async Task ValidateResponse(HttpResponseMessage response, HttpStatusCode expectedStatus, string clientId)
{
    response.StatusCode.ShouldBe(expectedStatus);
    
    if (expectedStatus == OK)
    {
        var result = await response.Content.ReadFromJsonAsync<ItemResponse<WorkloadClientDto>>(JsonOptions);
        result.ShouldNotBeNull();
        result.Item.ShouldNotBeNull();
        result.Item.ClientId.ShouldBe(clientId);
    }
}
```

Good example (early return):

```csharp
public async Task ValidateResponse(HttpResponseMessage response, HttpStatusCode expectedStatus, string clientId)
{
    response.StatusCode.ShouldBe(expectedStatus);
    
    if (expectedStatus != OK)
    {
        return;
    }
    
    var result = await response.Content.ReadFromJsonAsync<ItemResponse<WorkloadClientDto>>(JsonOptions);
    result?.Item?.ClientId.ShouldBe(clientId);
}
```

Benefits:
- Reduces nesting levels and improves readability.
- Makes the happy path clear by handling edge cases first.
- Easier to add additional logic without increasing complexity.
- Aligns with guard clause patterns.

More examples:

```csharp
// Bad: nested validation
public void ProcessOrder(Order order)
{
    if (order != null)
    {
        if (order.IsValid)
        {
            if (order.Total > 0)
            {
                // Process order...
            }
        }
    }
}

// Good: early returns
public void ProcessOrder(Order order)
{
    if (order == null)
        return;
    
    if (!order.IsValid)
        return;
    
    if (order.Total <= 0)
        return;
    
    // Process order...
}
```

## CancellationToken usage

All async methods must accept and propagate a `CancellationToken`.

Rules:
- Add `CancellationToken cancellationToken` as the last parameter to every `async` method.
- Pass `cancellationToken` through to every downstream async call — never drop it.
- Do not default to `CancellationToken.None` inside a method body when a token was provided by the caller.
- Name the parameter `cancellationToken` (camelCase, no abbreviation).

Exceptions:
- Event handlers (signature is fixed by the event delegate).
- Top-level entry points where no caller token exists — use `CancellationToken.None` or a `CancellationTokenSource` tied to application lifetime.

Bad example (token not accepted or forwarded):

```csharp
public async Task<Order?> GetOrderAsync(string orderId)
{
    return await _repository.FindByIdAsync(orderId); // token dropped
}
```

Good example:

```csharp
public async Task<Order?> GetOrderAsync(string orderId, CancellationToken cancellationToken)
{
    return await _repository.FindByIdAsync(orderId, cancellationToken);
}
```

## Configuration — prefer strongly typed IOptions<T>

Never inject `IConfiguration` directly into services or controllers. Always bind configuration sections to strongly typed options classes.

Rules:
- Define a dedicated options class per configuration section.
- Declare a `public const string ConfigurationSection` on the class matching the `appsettings.json` key.
- Annotate required properties with data annotations (`[Required]`, `[Url]`, etc.).
- Register with `AddOptions<T>().BindConfiguration().ValidateDataAnnotations().ValidateOnStart()` so misconfiguration fails at startup, not at runtime.
- Inject `IOptions<T>` in constructors and access the value via `.Value`.
- Use `IOptionsSnapshot<T>` when the value must reflect per-request config reloads, or `IOptionsMonitor<T>` for long-running background services that need change notifications.

Bad example (raw IConfiguration):

```csharp
public class EmailService
{
    private readonly string _smtpHost;

    public EmailService(IConfiguration configuration)
    {
        _smtpHost = configuration["Email:SmtpHost"]!; // no validation, magic string
    }
}
```

Good example — options class:

```csharp
public class EmailOptions
{
    public const string ConfigurationSection = "Email";

    [Required]
    public string SmtpHost { get; init; } = string.Empty;

    [Range(1, 65535)]
    public int SmtpPort { get; init; } = 587;
}
```

Registration in `Program.cs`:

```csharp
services.AddOptions<EmailOptions>()
    .BindConfiguration(EmailOptions.ConfigurationSection)
    .ValidateDataAnnotations()
    .ValidateOnStart();
```

Injection and usage:

```csharp
public class EmailService
{
    private readonly EmailOptions _options;

    public EmailService(IOptions<EmailOptions> options)
    {
        _options = options.Value;
    }
}
```

When to use each interface:
- `IOptions<T>` — singleton lifetime; value is fixed for the application lifetime. Use in most cases.
- `IOptionsSnapshot<T>` — scoped lifetime; re-evaluated per request. Use when config can reload and the service is scoped.
- `IOptionsMonitor<T>` — singleton lifetime with change notifications. Use in hosted services or singletons that need live reloads.

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

For response type declarations, status code documentation, `ProblemDetails` contracts, and OpenAPI metadata, see the `api-design` skill.

## Model mapping

Mapping logic between domain models and request/response models belongs in those models themselves, not in services or controllers.

Rules:
- Never place mapping logic in services, controllers, or external mapper classes.
- The method must be named `Map` and be `static`.
- **Response models** define `static TResponse Map(TDomain model)` — they receive a domain model and return themselves.
- **Request models** define `static TDomain Map(this TRequest request)` — they map themselves to a domain model.

### Response mapping

```csharp
public class UserAccountResponse
{
    public string Id { get; init; }
    public string DisplayName { get; init; }
    public string Email { get; init; }

    public static UserAccountResponse Map(UserAccount account) => new()
    {
        Id = account.Id,
        DisplayName = account.FullName,
        Email = account.Email
    };
}

// Controller:
return Ok(UserAccountResponse.Map(account));
```

When mapping collections, use LINQ:

```csharp
var responses = accounts
    .Select(UserAccountResponse.Map)
    .ToList();
```

### Request mapping

```csharp
public class UserAccountRequest
{
    public string FullName { get; init; }
    public string Email { get; init; }

    public static UserAccount Map(UserAccountRequest request) => new()
    {
        FullName = request.FullName,
        Email = request.Email
    };
}

// Controller:
var account = UserAccountRequest.Map(request);
```

## Use real types over primitives

Prefer purpose-built types over raw primitives when a richer type exists and accurately represents the value's intent.

Rules:
- Use `System.Uri` instead of `string` for URLs and URIs.
- Use `Guid` instead of `string` for identifiers that are GUIDs.
- Use strongly-typed ID types (e.g., `TypeId`, `CustomerId`, `OrderId`) instead of `string` or `Guid` when the domain has specific ID types.
- Use `DateTimeOffset` instead of `string` for timestamps.
- Use `decimal` instead of `double` or `string` for monetary values.
- Use enums instead of `string` or `int` for fixed sets of values.

Bad examples:

```csharp
public string WebhookUrl { get; set; }          // should be Uri
public string CustomerId { get; set; }           // should be Guid or CustomerId
public string OrderId { get; set; }              // should be typed ID
public string CreatedAt { get; set; }            // should be DateTimeOffset
```

Good examples:

```csharp
public Uri WebhookUrl { get; set; }
public Guid CustomerId { get; set; }
public OrderId OrderId { get; set; }             // strongly-typed ID
public DateTimeOffset CreatedAt { get; set; }
```

When to keep `string`:
- The value truly has no more specific type (e.g., a free-text name or description).
- Interoperability constraints require it (e.g., serialization boundaries where the richer type cannot be used).
- The value may contain formats not representable by the target type.

At serialization boundaries (e.g., request/response models), accept primitives if needed and map to real types in the domain model. Never propagate raw primitives deeper than the boundary layer.

## Test assertions

Keep test assertions clean and minimal by asserting only what's necessary.

Rules:
- Do not chain multiple `ShouldNotBeNull()` assertions when a single assertion on a nested property is sufficient.
- Use the null-conditional operator (`?`) to access nested properties when needed.
- Assert the final value or condition directly rather than asserting each step of navigation.

Bad example (redundant null checks):

```csharp
var result = await _service.GetOrderAsync(orderId);

result.ShouldNotBeNull();
result.Item.ShouldNotBeNull();
result.Item.Name.ShouldBe(uniqueName);
```

Good example (direct assertion):

```csharp
var result = await _service.GetOrderAsync(orderId);

result?.Item?.Name.ShouldBe(uniqueName);
```

Benefits:
- Reduces test noise and improves readability.
- Focuses on the actual assertion rather than intermediate navigation.
- Makes test intent clearer by showing what truly matters.

When redundant null checks are acceptable:
- When the null state itself is part of the test scenario and you need to verify each step.
- When a more descriptive test failure message is needed to diagnose issues.

In most cases, use the null-conditional operator and assert the final value directly.

## Accessibility — prefer the most restrictive modifier

Always declare classes, types, and members with the most restrictive access modifier possible.

Rules:
- Default to `private`. Widen to `internal`, `protected`, or `public` only when genuinely required.
- Prefer `internal` over `public` for types that do not form part of a public API surface.
- Test projects that need access to `internal` types must use `[assembly: InternalsVisibleTo("MyProject.Tests")]` in the production project — do not widen to `public` just for tests.

```csharp
// Wrong — public when nothing outside the assembly needs it
public class OrderValidator { }

// Correct
internal class OrderValidator { }
```

```csharp
// Wrong — widened to public just for tests
public string BuildQuery(Filter filter) { ... }

// Correct — internal, tests access via InternalsVisibleTo
internal string BuildQuery(Filter filter) { ... }
```

`InternalsVisibleTo` declaration (in production `.csproj` or `AssemblyInfo.cs`):

```csharp
[assembly: InternalsVisibleTo("MyProject.Tests")]
```

## Enforcement

- Use PR checklists that reference this skill.
- Enforce in code review.
- Ensure `Globals.cs` is maintained as the project evolves.