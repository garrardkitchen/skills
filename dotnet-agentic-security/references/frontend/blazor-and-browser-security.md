# Blazor and Browser-Facing ASP.NET Core Security

Use this file when the user builds Blazor, Razor Pages, MVC, browser-hosted API clients, cookie-authenticated apps, or BFF-style front ends.

## Must implement

- Use antiforgery protection for cookie-authenticated unsafe HTTP methods.
- Use secure cookie flags: `Secure`, `HttpOnly`, appropriate `SameSite`.
- Avoid storing access tokens in browser local storage where a safer BFF/session pattern is viable.
- Apply CSP and output encoding for XSS risk.
- Protect SignalR and browser streaming endpoints.
- Use CORS narrowly and never as an authorization control.

## Cookie policy sketch

```csharp
builder.Services.ConfigureApplicationCookie(options =>
{
    options.Cookie.HttpOnly = true;
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
    options.Cookie.SameSite = SameSiteMode.Lax;
    options.SlidingExpiration = false;
});
```

## Antiforgery pattern for browser forms

```csharp
[Authorize]
[AutoValidateAntiforgeryToken]
public sealed class AccountController : Controller
{
    [HttpPost("/account/email")]
    public IActionResult ChangeEmail(ChangeEmailRequest request)
    {
        return Accepted();
    }
}
```

For Minimal API handlers that bind browser form data, register antiforgery services and call `app.UseAntiforgery()`. JSON APIs protected by cookies need an explicit CSRF strategy, such as a BFF framework or a validated antiforgery token header.

## Verification

- unsafe form posts require antiforgery tokens
- cookies are secure and HttpOnly
- unauthorized browser user cannot call protected APIs
- CSP does not allow unsafe script in production without explicit review
- local storage does not contain bearer tokens unless risk accepted

## Avoid

- disabling antiforgery globally for browser/cookie flows
- broad `AllowAnyOrigin` CORS on credentialed endpoints
- leaking tokens through SignalR query strings into logs
- trusting client-side authorization checks

## Sources

- Microsoft Learn ASP.NET Core Blazor security
- Microsoft Learn ASP.NET Core antiforgery, CORS, cookies, and SignalR auth
- OWASP XSS, CSRF, and Session Management Cheat Sheets
