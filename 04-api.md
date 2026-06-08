# 🔌 API — Application Programming Interface

> **Interview Priority:** ⭐⭐⭐⭐ — Every .NET developer builds and consumes APIs.

---

## 📌 What Is an API?

**API** stands for **Application Programming Interface**. It's a **contract** between two systems that defines how they communicate.

In web development, a **REST API** exposes endpoints over HTTP:
- The **client** sends a request to a URL
- The **server** processes it and returns JSON

```
Client (Browser / Mobile App)
        ↓  HTTP Request (GET /api/products/1)
      [REST API]
        ↓  JSON Response { "id": 1, "name": "Laptop" }
Client receives data
```

---

## 🤔 Why Do APIs Exist?

| Reason | Explanation |
|--------|-------------|
| 🔀 **Separation** | Frontend and backend are **independent** — different teams, different technologies, different deployment cycles |
| ♻️ **Reusability** | The **same API** serves your web app, mobile app, and third-party integrations |
| 🔐 **Security** | The database is **never exposed** directly — the API controls what goes in and what comes out |

---

## 🌐 HTTP Methods

| Method | Action | Has Body | Idempotent | Safe |
|--------|--------|----------|-----------|------|
| `GET` | Read / fetch data | ❌ | ✅ | ✅ |
| `POST` | Create a new resource | ✅ | ❌ | ❌ |
| `PUT` | Replace an entire resource | ✅ | ✅ | ❌ |
| `PATCH` | Partially update a resource | ✅ | ✅ | ❌ |
| `DELETE` | Delete a resource | ❌ | ✅ | ❌ |

> 💡 **Idempotent** = calling it multiple times gives the same result.  
> 💡 **Safe** = has no side effects on server state.

### In an ASP.NET Core Controller

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]                        // GET  /api/products
    public async Task<IActionResult> GetAll() { ... }

    [HttpGet("{id}")]                // GET  /api/products/5
    public async Task<IActionResult> GetById(int id) { ... }

    [HttpPost]                       // POST /api/products
    public async Task<IActionResult> Create([FromBody] CreateProductDto dto) { ... }

    [HttpPut("{id}")]                // PUT  /api/products/5
    public async Task<IActionResult> Update(int id, [FromBody] UpdateProductDto dto) { ... }

    [HttpPatch("{id}")]              // PATCH /api/products/5
    public async Task<IActionResult> PartialUpdate(int id, [FromBody] PatchProductDto dto) { ... }

    [HttpDelete("{id}")]             // DELETE /api/products/5
    public async Task<IActionResult> Delete(int id) { ... }
}
```

---

## 📦 Common HTTP Status Codes

| Code | Meaning | When to Return |
|------|---------|---------------|
| `200 OK` | Success | Successful GET, PUT, PATCH, DELETE |
| `201 Created` | Resource created | Successful POST |
| `204 No Content` | Success, no body | DELETE, PUT with no body response |
| `400 Bad Request` | Validation failed | Invalid input, model binding errors |
| `401 Unauthorized` | Not authenticated | Missing or invalid token |
| `403 Forbidden` | Not authorized | Authenticated but insufficient permissions |
| `404 Not Found` | Resource doesn't exist | Id not found in DB |
| `500 Internal Server Error` | Server crash | Unhandled exception |

---

## 🔗 REST Principles

REST (**Representational State Transfer**) is an architectural style for APIs:

1. **Stateless** — each request contains all information needed; the server stores no session state.
2. **Resource-based** — URLs represent resources (nouns), not actions.
3. **HTTP methods as actions** — use `GET`, `POST`, `PUT`, `DELETE` semantically.
4. **Uniform interface** — consistent URL patterns.

```
✅ RESTful
GET    /api/orders          → get all orders
GET    /api/orders/1        → get order 1
POST   /api/orders          → create order
DELETE /api/orders/1        → delete order 1

❌ Not RESTful
GET /api/getOrders
POST /api/deleteOrder/1
GET /api/createOrder?name=...
```

---

## 📬 Model Binding

ASP.NET Core automatically maps incoming HTTP request data to your method parameters:

| Attribute | Source | Example |
|-----------|--------|---------|
| `[FromRoute]` | URL segment | `api/products/{id}` |
| `[FromQuery]` | Query string | `api/products?page=2&size=10` |
| `[FromBody]` | Request body (JSON) | `POST` with JSON payload |
| `[FromHeader]` | HTTP header | `Authorization: Bearer ...` |
| `[FromForm]` | Form data | File uploads |

```csharp
[HttpGet("{id}")]
public IActionResult Get(
    [FromRoute] int id,
    [FromQuery] string? include = null)
{ ... }
```

---

## 🔗 Related Topics

- Token storage and security tradeoffs → [06-authentication-authorization.md](06-authentication-authorization.md)
- Middleware pipeline for request handling → [03-aspnet-core.md](03-aspnet-core.md)
