# 🔐 Authentication & Authorization

> **Interview Priority:** ⭐⭐⭐⭐⭐ — JWT, cookies, and token security are always asked.

---

## 🔑 The Difference

| | Authentication | Authorization |
|-|---------------|--------------|
| **Question** | "Who are you?" | "What can you do?" |
| **When** | First (identify the user) | After authentication |
| **How** | Login → verify → issue token | Check roles, claims, or policies |
| **ASP.NET Core** | `UseAuthentication()` middleware | `UseAuthorization()` middleware |

### Request Flow

```
Incoming Request
      ↓
[Authentication Middleware]   → reads token, identifies the user (sets HttpContext.User)
      ↓
[Authorization Middleware]    → checks if this user can access this route
      ↓
[Controller / Endpoint]
```

---

## 🪙 JWT — JSON Web Token

### Structure

A JWT is **three Base64Url-encoded parts** separated by dots:

```
eyJhbGciOiJIUzI1NiJ9  .  eyJzdWIiOiIxMjMifQ  .  SflKxwRJS...
      Header                   Payload               Signature
```

### Header

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### Payload (Claims)

```json
{
  "sub": "user-guid-here",
  "email": "musab@gmail.com",
  "role": "Admin",
  "exp": 1718000000
}
```

### Signature

```
HMACSHA256(
  base64Url(header) + "." + base64Url(payload),
  secretKey
)
```

> ⚠️ **Critical:** JWT payload is **Base64 encoded, NOT encrypted**. Anyone can decode it and read the claims. **Never store passwords, credit card numbers, or sensitive secrets in a JWT payload.**

### How It Works

```
1. User logs in  →  POST /api/auth/login  { email, password }
2. Server verifies credentials
3. Server issues JWT  →  returns { token: "eyJ..." }
4. Client stores token (localStorage or HttpOnly cookie)
5. Client sends token on every request:
   Authorization: Bearer eyJhbGci...
6. Server validates signature → if valid, identifies user from claims
```

---

## 🛡️ Authorization — What Can You Do?

### Type 1 — Role-Based (Most Common)

```csharp
[Authorize]                              // any authenticated user
[Authorize(Roles = "Admin")]             // only Admin
[Authorize(Roles = "Admin,Manager")]     // Admin OR Manager
[AllowAnonymous]                         // skip auth entirely (public endpoint)
```

### Type 2 — Claims-Based

**Claims** are key-value pairs inside the JWT payload. More granular than roles.

```csharp
// Check specific claim in code
if (User.HasClaim("Department", "Engineering"))
{
    // allow access
}
```

### Type 3 — Policy-Based (Most Flexible)

Combine multiple requirements into a named policy:

```csharp
// Program.cs — define the policy
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("SeniorEmployee", policy =>
        policy.RequireClaim("YearsOfService", "5", "6", "7", "8", "9", "10")
              .RequireRole("Employee"));
});

// Controller — use the policy
[Authorize(Policy = "SeniorEmployee")]
public IActionResult GetSensitiveReport() { ... }
```

### Authorization Comparison

| Type | Flexibility | Complexity | Use When |
|------|-----------|-----------|---------|
| Role-based | Low | Simple | Basic access control |
| Claims-based | Medium | Medium | User attributes matter |
| Policy-based | High | More code | Complex, multi-rule requirements |

---

## 💾 Where Is the Token Stored?

Three options — each with tradeoffs:

### 1. `localStorage`

```javascript
localStorage.setItem("token", "eyJhbGci...");
// sent on every request:
Authorization: Bearer eyJhbGci...
```

| | |
|-|-|
| ✅ | Simple to implement; survives page refresh |
| ❌ | **Vulnerable to XSS** — malicious scripts can read it |

### 2. `sessionStorage`

Same API as `localStorage` but **cleared when the tab closes**. Still XSS-vulnerable.

### 3. `HttpOnly` Cookie (Recommended)

Server sets the cookie with the `HttpOnly` flag:

```
Set-Cookie: token=eyJ...; HttpOnly; Secure; SameSite=Strict
```

| | |
|-|-|
| ✅ | JavaScript **cannot read** `HttpOnly` cookies → protected from XSS |
| ✅ | Browser sends it automatically on every request |
| ❌ | Vulnerable to CSRF — mitigate with `SameSite=Strict` or CSRF tokens |

### Token Storage Comparison

| Method | XSS Safe | CSRF Safe | Persists on Close |
|--------|---------|---------|-----------------|
| `localStorage` | ❌ | ✅ | ✅ |
| `sessionStorage` | ❌ | ✅ | ❌ |
| `HttpOnly` Cookie | ✅ | ⚠️ (needs SameSite) | Depends on expiry |

> 💡 **Recommendation:** Use `HttpOnly` cookies with `SameSite=Strict` and HTTPS for session tokens.

---

## 🆔 ASP.NET Core Identity

ASP.NET Core Identity is a **built-in membership system** that handles:
- User creation, login, password hashing
- Account lockout
- Role management

### Key Classes

| Class | Purpose |
|-------|---------|
| `UserManager<TUser>` | User operations (create, delete, add to role) |
| `SignInManager<TUser>` | Login / logout, password verification |
| `RoleManager<TRole>` | Role creation and management |

```csharp
// Extend IdentityUser for custom fields
public class ApplicationUser : IdentityUser
{
    public string FullName { get; set; } = string.Empty;
    public string? ProfileUrl { get; set; }
}

// Program.cs — register Identity
builder.Services.AddIdentity<ApplicationUser, IdentityRole>()
    .AddEntityFrameworkStores<AppDbContext>();

// Seed roles on startup
public class DbSeeder(RoleManager<IdentityRole> roleManager)
{
    public async Task SeedRolesAsync()
    {
        foreach (var role in new[] { "Admin", "Manager", "Employee" })
            if (!await roleManager.RoleExistsAsync(role))
                await roleManager.CreateAsync(new IdentityRole(role));
    }
}

// Assign user to role
await _userManager.AddToRoleAsync(user, "Admin");
```

> 💡 EF Core adds `AspNetUsers`, `AspNetRoles`, `AspNetUserRoles` tables automatically via migration.

---

## ✅ Summary

| Concept | One-Liner |
|---------|-----------|
| **Authentication** | "Who are you?" — verify identity, issue token |
| **Authorization** | "What can you do?" — check role/claim/policy |
| **JWT** | Stateless token: Header.Payload.Signature |
| **JWT Payload** | Base64 encoded, NOT encrypted — don't put secrets |
| **Role-based auth** | `[Authorize(Roles = "Admin")]` |
| **Claims-based auth** | Key-value pairs in the token for granular control |
| **Policy-based auth** | Named rule combining multiple requirements |
| **localStorage** | Simple but XSS-vulnerable |
| **HttpOnly cookie** | XSS-safe, needs CSRF protection |
| **ASP.NET Identity** | Built-in membership: UserManager + SignInManager |
