# Short Differences

## app.Use vs app.Run
- `app.Use(...)` registers middleware that can call the next middleware in the pipeline.
- `app.Run(...)` registers a terminal middleware that does not call the next delegate.

## string vs StringBuilder
- `string` is immutable; each modification creates a new object.
- `StringBuilder` is mutable and more efficient for many concatenations.

## var vs dynamic
- `var` is inferred at compile time and still strongly typed:
```csharp
var name = "abc";
// name = 123; // compiler error
```
- `dynamic` defers binding to runtime.

## ref vs out
- `ref` requires an initial value; the method may change it.
- `out` does not require an initial value; the method must assign it.
- Example: `int.TryParse("123", out int value);`

## Model Binding
- ASP.NET Core automatically maps incoming HTTP request data to your method parameters.

## Global Exception Handling
- Instead of try/catch in every controller, handle exceptions centrally in middleware.


# .Net Core vs .Net Framework
- .NET Framework is the original Windows-only implementation. .NET Core is cross-platform, modular, and open-source. Since .NET 5, Microsoft unified them into a single product called .NET (currently .NET 6/7/8). For new development, everyone uses .NET Core or the later .NET versions. I built my e-commerce backend with ASP.NET Core 8 – it runs on Linux in Azure, which .NET Framework cannot do.”*

# JWT vs Session
- The One Real Tradeoff to Mention
Sessions win on invalidation — if a user gets compromised you delete their session instantly, they're logged out immediately.
JWT loses here — you cannot invalidate a JWT before it expires. If someone steals a token that expires in 7 days, they have 7 days of access. Solutions exist like token blacklists but they reintroduce server state which defeats the purpose.

- Sessions store login state on the server and give the client just an ID. JWT stores everything in the token itself so the server is completely stateless — no database lookup on every request, just signature validation. JWT scales better because any server can validate the token without shared storage. The tradeoff is invalidation — with sessions you can log someone out instantly by deleting their session, with JWT the token stays valid until it expires which is why I keep expiry times short.