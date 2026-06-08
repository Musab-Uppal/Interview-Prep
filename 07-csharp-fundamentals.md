# 💻 C# Fundamentals & Quick Differences

> **Interview Priority:** ⭐⭐⭐⭐ — Expect rapid-fire "what's the difference between X and Y" questions.

---

## 🔄 .NET Framework vs .NET Core vs .NET

| | .NET Framework | .NET Core | .NET 5+ |
|-|--------------|----------|--------|
| **Platform** | Windows only | Cross-platform | Cross-platform |
| **Open source** | ❌ | ✅ | ✅ |
| **Performance** | Moderate | High | High |
| **Cloud / Docker** | Limited | ✅ Native | ✅ Native |
| **Status** | Legacy (no new features) | Succeeded by .NET 5+ | Current standard |

> 💡 **In your own words:** ".NET Framework is the original Windows-only runtime. .NET Core is cross-platform and open-source. Since .NET 5, Microsoft unified them into a single product simply called '.NET'. I built my e-commerce backend on ASP.NET Core 8 — it runs on Linux in Azure, which .NET Framework cannot do."

---

## ⚡ `app.Use` vs `app.Run`

| | `app.Use(...)` | `app.Run(...)` |
|-|--------------|--------------|
| **Calls next middleware?** | ✅ Yes — passes control forward | ❌ No — terminal handler |
| **Position in pipeline** | Middle | End |

```csharp
// Use — passes on after doing its work
app.Use(async (context, next) =>
{
    Console.WriteLine("Before next middleware");
    await next();
    Console.WriteLine("After next middleware");
});

// Run — terminal, never calls next
app.Run(async context =>
{
    await context.Response.WriteAsync("Final response");
});
```

---

## 🔤 `string` vs `StringBuilder`

| | `string` | `StringBuilder` |
|-|---------|---------------|
| **Mutability** | Immutable — each operation creates a new object | Mutable — modifies in place |
| **Memory** | Lots of allocations in loops | One allocation, reused buffer |
| **Use when** | Few concatenations or fixed strings | Many concatenations in a loop |

```csharp
// ❌ Inefficient for many concatenations — creates a new string each iteration
string result = "";
for (int i = 0; i < 1000; i++)
    result += i.ToString();

// ✅ Efficient
var sb = new StringBuilder();
for (int i = 0; i < 1000; i++)
    sb.Append(i);
string result = sb.ToString();
```

---

## 🔬 `var` vs `dynamic`

| | `var` | `dynamic` |
|-|------|---------|
| **Type resolved** | Compile time | Runtime |
| **Type-safe** | ✅ Yes — still strongly typed | ❌ No — type errors at runtime |
| **IntelliSense** | ✅ Full | ❌ None |
| **Use when** | Always (preferred for local variables) | Interop with COM, reflection, JSON parsing |

```csharp
// var — compiler infers string, enforces it at compile time
var name = "Musab";
// name = 123; // ❌ compiler error

// dynamic — no compile-time type checking
dynamic value = "hello";
value = 123;      // ✅ no error at compile time
value = true;     // ✅ still no error — danger!
```

---

## 📤 `ref` vs `out`

| | `ref` | `out` |
|-|------|------|
| **Initial value required?** | ✅ Must be assigned before passing | ❌ Not required |
| **Method must assign?** | No (may or may not modify) | ✅ Yes — must assign before returning |
| **Direction** | In and out | Out only |

```csharp
// ref — value must exist before call
int number = 5;
Increment(ref number);  // number is now 6

void Increment(ref int x) => x++;

// out — method is responsible for the value
bool success = int.TryParse("123", out int parsed);
// 'parsed' has a value because TryParse assigned it
```

---

## 🌐 `IQueryable` vs `IEnumerable`

| | `IQueryable<T>` | `IEnumerable<T>` |
|-|----------------|----------------|
| **Where runs** | Database (SQL WHERE) | Memory (C# LINQ) |
| **When to use** | EF Core queries — filter before fetching | Already-fetched in-memory collections |
| **Performance** | ✅ Only fetches matching rows | ❌ Loads all rows then filters |

```csharp
// ✅ IQueryable — WHERE translated to SQL
IQueryable<Product> query = _context.Products.Where(p => p.Price > 1000);

// ❌ IEnumerable — loads ALL products into memory first, then filters in C#
IEnumerable<Product> allProducts = _context.Products.ToList();
var filtered = allProducts.Where(p => p.Price > 1000);
```

---

## 🔁 `async`/`await`

| Term | Meaning |
|------|---------|
| `async` | Marks a method as asynchronous; allows `await` inside |
| `await` | Suspends execution until the awaited task completes, without blocking the thread |
| `Task` | Represents an ongoing asynchronous operation |
| `Task<T>` | Async operation that returns a value of type `T` |

```csharp
// Sync — blocks the thread while waiting for DB
public List<Product> GetProducts()
{
    return _context.Products.ToList(); // thread is blocked
}

// Async — frees the thread while waiting for DB
public async Task<List<Product>> GetProductsAsync()
{
    return await _context.Products.ToListAsync(); // thread freed, resumes when DB responds
}
```

> 💡 In ASP.NET Core, `async` allows the server to handle other requests while waiting for I/O (database, HTTP calls, file reads). This improves throughput under load.

---

## 🔒 JWT vs Session

| | JWT | Session |
|-|-----|--------|
| **State** | Stateless — all data in the token | Stateful — server stores session in DB/memory |
| **Server DB lookup** | ❌ None — just validate signature | ✅ Every request |
| **Scalability** | ✅ Any server can validate | ❌ Requires shared session store |
| **Invalidation** | ❌ Cannot invalidate before expiry | ✅ Delete session instantly |
| **Token size** | Larger (carries claims) | Small (just a session ID) |

> 💡 **The real tradeoff:** Sessions win on **invalidation** — if a user is compromised, you delete their session and they're logged out immediately. With JWT, the token stays valid until it expires. Solutions like token blacklists exist, but they reintroduce server state and defeat the stateless benefit. That's why I keep JWT expiry times short (15min–1hr) and use refresh tokens.

---

## 🛡️ Global Exception Handling

Instead of `try/catch` in every controller, handle all exceptions in **one place**:

```csharp
// ASP.NET Core 8 — minimal API approach
app.UseExceptionHandler(errorApp =>
{
    errorApp.Run(async context =>
    {
        var error = context.Features.Get<IExceptionHandlerFeature>();
        context.Response.StatusCode = 500;
        context.Response.ContentType = "application/json";
        await context.Response.WriteAsJsonAsync(new
        {
            error = "An unexpected error occurred.",
            detail = error?.Error.Message
        });
    });
});

// Or implement IExceptionHandler (cleaner approach in .NET 8+)
public class GlobalExceptionHandler : IExceptionHandler
{
    public async ValueTask<bool> TryHandleAsync(
        HttpContext context,
        Exception exception,
        CancellationToken ct)
    {
        context.Response.StatusCode = exception switch
        {
            NotFoundException => StatusCodes.Status404NotFound,
            ValidationException => StatusCodes.Status400BadRequest,
            _ => StatusCodes.Status500InternalServerError
        };
        await context.Response.WriteAsJsonAsync(new { error = exception.Message }, ct);
        return true;
    }
}
```

---

## ✅ Quick Reference Summary

| Question | Answer |
|---------|--------|
| `string` vs `StringBuilder` | `string` immutable; `StringBuilder` mutable — use for loops |
| `var` vs `dynamic` | `var` compile-time; `dynamic` runtime (no type safety) |
| `ref` vs `out` | `ref` needs value before call; `out` method must assign |
| `IQueryable` vs `IEnumerable` | `IQueryable` runs in DB; `IEnumerable` runs in memory |
| `async`/`await` | Non-blocking I/O; frees thread while waiting |
| JWT vs Session | JWT stateless/scalable; Session stateful/instantly invalidatable |
| `.NET Framework` vs `.NET Core` | Framework Windows-only legacy; Core cross-platform current |
| `app.Use` vs `app.Run` | `Use` passes to next; `Run` is terminal |
