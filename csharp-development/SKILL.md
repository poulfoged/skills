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
