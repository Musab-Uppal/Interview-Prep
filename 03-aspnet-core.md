# ⚙️ ASP.NET Core

> **Interview Priority:** ⭐⭐⭐⭐⭐ — The runtime of every .NET web project.

---

## 📌 What Is ASP.NET Core?

A **cross-platform, open-source** framework by Microsoft for building optimized web apps and Web APIs. Runs on Windows, Linux, and macOS.

---

## 🚀 Application Startup — `Program.cs`

`Program.cs` is the **entry point** of every ASP.NET Core app. It does two things:

1. **Register services** into the DI container (`builder.Services.*`).
2. **Configure the middleware pipeline** (`app.Use*`, `app.Map*`).

```csharp
var builder = WebApplication.CreateBuilder(args);

// 1️⃣ Register services
builder.Services.AddControllers();
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

var app = builder.Build();

// 2️⃣ Configure middleware pipeline
app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

---

## 🔗 Middleware Pipeline

**Middleware** are functions that execute in the HTTP **request → response** pipeline. Each middleware can:
- Process the request
- Pass it to the next middleware
- Short-circuit (return a response immediately)

```
Request
   ↓
[HTTPS Redirect]
   ↓
[Authentication]   ← reads token, sets User identity
   ↓
[Authorization]    ← checks if User can access route
   ↓
[Routing]
   ↓
[Controller Action]
   ↑
[Response flows back up through the same chain]
```

### `app.Use` vs `app.Run`

| | `app.Use(...)` | `app.Run(...)` |
|-|---------------|---------------|
| **Next middleware** | Calls the next delegate | Terminal — does NOT call next |
| **Use when** | Middleware in the middle of the pipeline | Final handler |

```csharp
// Middleware that logs and passes on
app.Use(async (context, next) =>
{
    Console.WriteLine($"Request: {context.Request.Path}");
    await next(); // pass to next middleware
    Console.WriteLine($"Response: {context.Response.StatusCode}");
});

// Terminal middleware
app.Run(async context =>
{
    await context.Response.WriteAsync("Done");
});
```

---

## 💉 Dependency Injection (DI)

**Dependency Injection** is when a class receives its dependencies via the constructor instead of creating them internally. This makes classes:

- **Loosely coupled** — easy to swap implementations
- **Independently testable** — mock dependencies in unit tests

```csharp
// ❌ Tightly coupled — hard to test, hard to swap
public class OrderService
{
    private readonly SqlOrderRepository _repo = new SqlOrderRepository();
}

// ✅ Loosely coupled — depends on abstraction
public class OrderService(IOrderRepository _repo)
{
    // inject whatever implementation is registered
}
```

### DI Lifetimes

| Lifetime | Registration | Instance Created | Use When |
|----------|-------------|-----------------|---------|
| **Transient** | `AddTransient<T>()` | Every time it's requested | Lightweight, stateless services |
| **Scoped** | `AddScoped<T>()` | Once per HTTP request | `DbContext`, services that use DB |
| **Singleton** | `AddSingleton<T>()` | Once for the app lifetime | Config readers, caches, expensive objects |

> ⚠️ **Common mistake:** injecting a Scoped service into a Singleton. The Scoped service gets "captured" and lives for the entire app lifetime, causing stale data bugs.

### Registration Examples

```csharp
builder.Services.AddTransient<IEmailSender, SmtpEmailSender>();
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddSingleton<IConfiguration>(builder.Configuration);
```

---

## 🔓 Authorization Attributes

```csharp
[AllowAnonymous]                          // No authentication required (public endpoint)
[Authorize]                               // Any authenticated user
[Authorize(Roles = "Admin")]              // Admin role only
[Authorize(Roles = "Admin,Manager")]      // Admin OR Manager
[Authorize(Policy = "MinimumAge")]        // Custom policy
```

---

## ✅ Summary

| Concept | What It Does |
|---------|-------------|
| `Program.cs` | Entry point — registers services + configures pipeline |
| Middleware | Functions in the HTTP pipeline; ordered and composable |
| `app.Use` | Passes control to next middleware |
| `app.Run` | Terminal middleware |
| DI | Classes receive dependencies via constructor |
| `AddTransient` | New instance every request |
| `AddScoped` | One instance per HTTP request |
| `AddSingleton` | One instance for the app lifetime |
| `[AllowAnonymous]` | Skip authentication for this endpoint |
| `[Authorize]` | Require authentication |
