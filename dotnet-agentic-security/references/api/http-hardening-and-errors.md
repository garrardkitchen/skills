# HTTP Hardening, Middleware, and Error Contracts

Use this file when the user asks about secure ASP.NET Core API setup.

## Middleware ordering

```csharp
app.UseForwardedHeaders();
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler();
    app.UseHsts();
}
app.UseHttpsRedirection();
app.UseCors("ApiCors");
app.UseAuthentication();
app.UseAuthorization();
app.UseRateLimiter();
```

Configure forwarded headers only for known proxies/networks. Incorrect forwarded-header trust can corrupt scheme, host, client IP, redirect, and logging decisions.

## CORS

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("ApiCors", policy =>
    {
        policy.WithOrigins("https://app.contoso.com")
              .WithMethods("GET", "POST", "PUT", "DELETE")
              .WithHeaders("Authorization", "Content-Type")
              .AllowCredentials();
    });
});
```

CORS is not an authorization control. Use the narrowest allowed origins, methods, and headers. Only use `AllowCredentials()` with explicit trusted origins; never combine credentials with wildcard origins.

## ProblemDetails

```csharp
builder.Services.AddProblemDetails();

app.MapGet("/api/orders/{id:guid}", async (Guid id, IOrderReader reader, CancellationToken ct) =>
{
    OrderDto? order = await reader.GetAsync(id, ct);
    return order is null
        ? Results.Problem(statusCode: 404, title: "Order not found")
        : Results.Ok(order);
});
```

In production, error responses MUST avoid stack traces, raw exception messages, secrets, SQL details, file paths, and internal service names. Log details server-side with correlation identifiers.

## Security headers

```csharp
app.Use(async (context, next) =>
{
    context.Response.Headers["X-Content-Type-Options"] = "nosniff";
    context.Response.Headers["Referrer-Policy"] = "no-referrer";
    context.Response.Headers["X-Frame-Options"] = "DENY";
    context.Response.Headers["Content-Security-Policy"] = "default-src 'self'; frame-ancestors 'none'";
    await next();
});
```

For browser-facing apps, also review cookie flags (`Secure`, `HttpOnly`, `SameSite`), HSTS, CSP compatibility, and anti-clickjacking requirements. APIs consumed only by non-browser clients may need fewer browser headers, but they still need TLS and safe error contracts.

## Request size limits and safe uploads

This example is for non-browser clients using bearer tokens or equivalent non-cookie credentials. Cookie-authenticated browser uploads should keep antiforgery enabled.

```csharp
app.MapPost("/api/uploads", async (
    IFormFile file,
    IWebHostEnvironment env,
    ILogger<Program> logger) =>
{
    if (file.Length == 0 || file.Length > 10 * 1024 * 1024)
        return Results.BadRequest("Invalid file size.");

    string extension = Path.GetExtension(file.FileName);
    string[] allowedExtensions = [".txt", ".csv", ".json"];
    if (!allowedExtensions.Contains(extension, StringComparer.OrdinalIgnoreCase))
        return Results.BadRequest("File type is not allowed.");

    string trustedName = Path.GetRandomFileName();
    string path = Path.Combine(env.ContentRootPath, "quarantine", trustedName);

    Directory.CreateDirectory(Path.GetDirectoryName(path)!);

    await using FileStream fs = new(path, FileMode.CreateNew);
    await file.CopyToAsync(fs);

    logger.LogInformation("Stored uploaded file as {StoredName}", trustedName);
    return Results.Accepted();
})
.RequireAuthorization("Uploads.Write")
.DisableAntiforgery();
```

Only disable antiforgery for non-browser APIs protected by bearer tokens or equivalent non-cookie credentials. Browser forms and cookie-authenticated endpoints SHOULD use antiforgery protection.

Production upload handlers MUST also consider:

- server and reverse-proxy body limits
- content-type and extension allowlists
- malware scanning or quarantine workflows
- storage outside the web root
- randomized storage names
- no trust in the original filename
- authorization for who may upload and retrieve files

## Endpoint filters

```csharp
public sealed class CorrelationIdFilter : IEndpointFilter
{
    public async ValueTask<object?> InvokeAsync(EndpointFilterInvocationContext context, EndpointFilterDelegate next)
    {
        context.HttpContext.Response.Headers["X-Correlation-Id"] = context.HttpContext.TraceIdentifier;
        return await next(context);
    }
}
```

## Hardening checklist

- HTTPS redirection and HSTS in production
- developer exception page only in development
- authenticated and authorized sensitive endpoints
- CORS narrowed to known browser origins
- Swagger/OpenAPI protected outside development
- request body, multipart, and streaming limits
- consistent `ProblemDetails` and correlation IDs
- no sensitive details in client-facing errors

## Sources

- Microsoft Learn ASP.NET Core security, CORS, antiforgery, error handling, and rate limiting
- OWASP REST Security Cheat Sheet
- OWASP ASVS v5.0.0
