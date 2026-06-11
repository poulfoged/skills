---
name: api-documentation
description: Enforce OpenAPI documentation standards for ASP.NET Core APIs including status code declarations, TypedResults usage, ProblemDetails contracts, and response metadata.
license: Proprietary
compatibility: opencode
metadata:
  audience: contributors
  focus: openapi
---

## What I do

- Enforce how API response types and status codes are declared and documented.
- Review controllers and endpoints for correct OpenAPI metadata.
- Provide compliant examples and direct fixes when documentation is missing or incorrect.

## When to use me

Use this skill whenever you create, modify, or review ASP.NET Core API endpoints — controllers or minimal APIs — where response types, status codes, or OpenAPI metadata are involved.

## Primary approach: TypedResults

Use `TypedResults` with a `Results<T1, T2, ...>` return type. The return type itself becomes the OpenAPI contract at compile time — no additional attributes required for the common case.

```csharp
[HttpGet("{id}")]
public async Task<Results<Ok<OrderDto>, NotFound>> GetOrder(
    string id,
    CancellationToken cancellationToken)
{
    var order = await _orderService.GetOrderAsync(id, cancellationToken);

    return order is null
        ? TypedResults.NotFound()
        : TypedResults.Ok(order);
}
```

```csharp
[HttpPost]
public async Task<Results<Created<OrderDto>, BadRequest<ProblemDetails>>> CreateOrder(
    [FromBody] CreateOrderRequest request,
    CancellationToken cancellationToken)
{
    var result = await _orderService.CreateOrderAsync(request, cancellationToken);

    if (result.IsFailure)
    {
        return TypedResults.BadRequest(new ProblemDetails
        {
            Status = StatusCodes.Status400BadRequest,
            Title = "Invalid order",
            Detail = result.Error
        });
    }

    return TypedResults.Created($"/api/orders/{result.Value.Id}", result.Value);
}
```

Benefits:
- Compile-time guarantee that all declared response types are actually returned.
- OpenAPI metadata is generated automatically from the return type.
- Eliminates the need for `[ProducesResponseType]` attributes in most cases.
- Replaces the deprecated MVC API analyzers (`IncludeOpenAPIAnalyzers`, removed in .NET 10).

## Supplemental: [ProducesResponseType] for descriptions

Use `[ProducesResponseType]` only when you need to add a human-readable `Description` to a response, or when a status code cannot be expressed through the `Results<>` type (e.g. a status code returned by middleware).

```csharp
[HttpGet("{id}")]
[ProducesResponseType<OrderDto>(StatusCodes.Status200OK,
    Description = "The requested order.")]
[ProducesResponseType(StatusCodes.Status404NotFound,
    Description = "No order exists with the given ID.")]
[ProducesResponseType(StatusCodes.Status408RequestTimeout,
    Description = "The client disconnected before the response was sent.")]
public async Task<Results<Ok<OrderDto>, NotFound>> GetOrder(
    string id,
    CancellationToken cancellationToken)
{
    // ...
}
```

Rules:
- Use the generic form `[ProducesResponseType<T>]` when a response body type is involved.
- Use the non-generic form `[ProducesResponseType(statusCode)]` for bodyless responses.
- Always use `StatusCodes.Status*` constants — never raw integers.

## Do not use [SwaggerResponse]

`[SwaggerResponse]` is Swashbuckle-specific and not part of the ASP.NET Core standard. Do not use it.

Use `[ProducesResponseType]` for supplemental metadata instead.

## Standard status code catalogue

These are the agreed status codes for this codebase. Use only codes from this list unless a specific case requires otherwise, and document the reason in a comment.

| Code | Constant | When to use |
|------|----------|-------------|
| 200 | `Status200OK` | Successful GET or PUT with a response body. |
| 201 | `Status201Created` | Successful POST that creates a resource. Return a `Location` header. |
| 204 | `Status204NoContent` | Successful DELETE or action with no response body. |
| 400 | `Status400BadRequest` | Malformed request or model validation failure. Always return `ProblemDetails`. |
| 401 | `Status401Unauthorized` | Missing or invalid authentication credentials. |
| 403 | `Status403Forbidden` | Authenticated but not authorized to perform the action. |
| 404 | `Status404NotFound` | The requested resource does not exist. |
| 408 | `Status408RequestTimeout` | The client disconnected or aborted before the response was sent. |
| 409 | `Status409Conflict` | State conflict, e.g. duplicate resource or optimistic concurrency failure. |
| 422 | `Status422UnprocessableEntity` | Request is well-formed but fails semantic/business validation. Always return `ProblemDetails`. |
| 500 | `Status500InternalServerError` | Unexpected server fault. Always return `ProblemDetails`. |

### 400 vs 422

- `400 Bad Request` — the request itself is invalid: missing required fields, wrong types, malformed JSON, model binding failure.
- `422 Unprocessable Entity` — the request is structurally valid but violates a business rule: e.g. "order total cannot be negative", "end date must be after start date".

## ProblemDetails for all error responses

All 4xx and 5xx responses must return a `ProblemDetails` body with content type `application/problem+json`.

Rules:
- Never return a plain string or custom error object for error responses.
- Populate `Status`, `Title`, and `Detail` at minimum.
- Use `Extensions` for domain-specific error context when needed.

```csharp
TypedResults.BadRequest(new ProblemDetails
{
    Status = StatusCodes.Status400BadRequest,
    Title = "Validation failed",
    Detail = "The order total must be greater than zero."
});
```

```csharp
TypedResults.UnprocessableEntity(new ProblemDetails
{
    Status = StatusCodes.Status422UnprocessableEntity,
    Title = "Business rule violation",
    Detail = "End date must be after start date."
});
```

For 500 errors, use exception-handling middleware (e.g. `app.UseExceptionHandler`) to produce a `ProblemDetails` response centrally — do not catch and return 500 manually in controllers.

## Enforcement

- Use PR checklists that reference this skill.
- Enforce in code review.
- Prefer `TypedResults` return types as the primary documentation mechanism.
- Flag any use of `[SwaggerResponse]` as a violation.
- Flag raw integer status codes — always use `StatusCodes.Status*` constants.
