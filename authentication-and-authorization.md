# Authentication and Authorization

Request flow:

- Authentication middleware — reads token, identifies the user.
- Authorization middleware — checks if this user can access the route.
- Controller handles the request.

## Authentication — Who are you?
- Login → verify identity → issue token.

### JWT (JSON Web Token)
A JWT has three parts separated by dots: header, payload, signature.

Example token structure:
`eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjMifQ.SflKxwRJS`

Header:
```json
{ "alg": "HS256", "typ": "JWT" }
```

Payload (claims):
```json
{
  "sub": "user-guid-here",
  "email": "musab@gmail.com",
  "role": "Admin",
  "exp": 1718000000
}
```

Signature:
```
HMACSHA256(base64(header) + "." + base64(payload), secretKey)
```

Important: the JWT payload is base64 encoded, not encrypted. Never put sensitive data like passwords in it.

## Authorization — What can you do?
- **Role-based:** `[Authorize]`, `[Authorize(Roles = "Admin")]`.
- **Claims-based:** use claims inside tokens for more granular control.
- **Policy-based:** define policies combining multiple requirements.

## Where Is Token Stored

Three options — each with tradeoffs:

### localStorage
Frontend stores the token:

```javascript
localStorage.setItem("token", "eyJhbGci...");
```

The client then sends it in the `Authorization` header on each request:

```
Authorization: Bearer eyJhbGci...
```

Problem: vulnerable to XSS (cross-site scripting). If a malicious script runs on the page it can read `localStorage` and steal the token.

### sessionStorage
Same API as `localStorage` but cleared when the browser tab or window closes. Still vulnerable to XSS.

### HttpOnly cookie
The server sets a cookie with the `HttpOnly` flag. Example header:

```
Set-Cookie: token=eyJ...; HttpOnly; Secure; SameSite=Strict
```

The browser stores the cookie and sends it automatically on requests. JavaScript cannot read `HttpOnly` cookies, so they are protected from XSS. They are vulnerable to CSRF (cross-site request forgery) unless you mitigate with `SameSite` settings, CSRF tokens, or other defenses.

Recommendation: prefer `HttpOnly` cookies with appropriate `SameSite` and CSRF protections for session tokens. If you must store tokens in `localStorage`, ensure strict Content Security Policy (CSP) and minimize XSS risks.

# Identity Package of ASP.Net
- ASP.NET Core Identity is a membership system that handles users, roles, password hashing, and lockout out of the box. You extend IdentityUser to add custom fields like ProfileUrl or FullName and EF Core adds those columns to AspNetUsers table via migration. Roles are seeded on startup using RoleManager and assigned to users via UserManager.AddToRoleAsync. The key classes are UserManager for user operations and SignInManager for login — they handle all the password verification and lockout logic so you don't build it from scratch.

# HTTP Methods
GET     → read data, no body, safe and idempotent
POST    → create new resource, has body
PUT     → update entire resource, has body
PATCH   → update partial resource, has body
DELETE  → delete resource, no body
