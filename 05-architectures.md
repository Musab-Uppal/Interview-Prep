# 🏗️ Architectures

> **Interview Priority:** ⭐⭐⭐⭐ — Shows you understand the bigger picture beyond just writing controllers.

---

## 🗺️ Architecture Overview

| Architecture | Coupling | Scale | Best For |
|-------------|---------|-------|---------|
| **MVC (Monolith)** | Tight | Small–Medium | Full-stack web apps, admin panels |
| **N-Tier** | Medium | Medium | Separating concerns in a monolith |
| **Clean Architecture** | Low | Medium–Large | Enterprise apps, testable codebases |
| **Microservices** | Very Low | Large | High-scale distributed systems |

---

## 🏠 MVC (Monolith)

**Model-View-Controller** — a full-stack architecture where:
- **Model** → data classes and business logic
- **View** → `.cshtml` Razor pages rendered on the server
- **Controller** → maps URLs, processes requests, returns views

```csharp
public class ProductsController : Controller
{
    public IActionResult Index()
    {
        var products = _service.GetAll();
        return View(products); // returns .cshtml, not JSON
    }
}
```

> 🔹 Controllers return **Views** (HTML), not JSON. For APIs, you inherit from `ControllerBase` instead.  
> 🔹 Tightly coupled — frontend and backend change together. Good for small teams.

---

## 🏢 N-Tier Architecture

Everything in one **repository**, but organized into **layers**:

```
Presentation Layer    (Controllers, Views)
       ↓
Service / Business Layer   (Business logic)
       ↓
Data Access Layer     (Repositories, DbContext)
       ↓
Database
```

> 💡 Better than a single-layer monolith but still a single deployable unit.

---

## 🌀 Clean Architecture

Dependencies point **inward only** — the business logic has **no dependency** on external frameworks.

```
┌────────────────────────────────┐
│         Presentation            │  Controllers, API endpoints
│              ↓                  │
│         Infrastructure          │  EF Core, SQL, external APIs
│              ↓                  │
│         Application             │  Services, interfaces, DTOs
│              ↓                  │
│            Domain               │  Entities, business rules (pure C#)
└────────────────────────────────┘
         ← Dependencies point inward →
```

### Layers Explained

| Layer | Lives Here | Does NOT depend on |
|-------|-----------|-------------------|
| **Domain** | Entities, value objects, business rules | Anything external |
| **Application** | Interfaces, services, DTOs, commands/queries | EF Core, HTTP, SQL |
| **Infrastructure** | EF Core repos, SQL queries, email senders | Domain/Application interfaces |
| **Presentation** | Controllers, API endpoints, middleware | Business logic details |

```csharp
// Domain layer — pure C#, no frameworks
public class Order
{
    public int Id { get; private set; }
    public decimal Total { get; private set; }

    public void AddItem(OrderItem item)
    {
        if (item.Quantity <= 0)
            throw new DomainException("Quantity must be positive.");
        // business rule enforced in domain
    }
}

// Application layer — defines the contract
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(int id);
    Task AddAsync(Order order);
}

// Infrastructure layer — implements it
public class EfOrderRepository : IOrderRepository
{
    private readonly AppDbContext _context;
    // ...
}
```

### CQRS (Command Query Responsibility Segregation)

Often used **alongside Clean Architecture** to separate reads from writes.

| | Command | Query |
|-|---------|-------|
| **Purpose** | Change state (create/update/delete) | Read state |
| **Returns** | Nothing or an ID | Data (DTOs) |
| **Speed requirement** | Not critical | Must be fast |
| **Example** | `CreateOrderCommand` | `GetOrderByIdQuery` |

In .NET, typically implemented with **MediatR**:

```csharp
// Command
public record CreateOrderCommand(int CustomerId, List<OrderItemDto> Items) 
    : IRequest<int>;

public class CreateOrderCommandHandler : IRequestHandler<CreateOrderCommand, int>
{
    public async Task<int> Handle(CreateOrderCommand command, CancellationToken ct)
    {
        // write logic here
    }
}

// Query
public record GetOrderByIdQuery(int Id) : IRequest<OrderDto>;

public class GetOrderByIdQueryHandler : IRequestHandler<GetOrderByIdQuery, OrderDto>
{
    public async Task<OrderDto> Handle(GetOrderByIdQuery query, CancellationToken ct)
    {
        // fast read logic here (can use Dapper for performance)
    }
}
```

---

## 🔬 Microservices

Each **service** has its own:
- Responsibility and bounded context
- Database (no shared DB)
- Deployment pipeline

```
┌──────────┐    ┌──────────┐    ┌──────────┐
│  Orders  │    │ Inventory│    │Notifications│
│ Service  │    │ Service  │    │  Service  │
│  DB: SQL │    │  DB: SQL │    │ DB: NoSQL │
└──────────┘    └──────────┘    └──────────┘
     ↕                ↕                ↕
          [API Gateway / Message Bus]
```

### Communication Between Services

| Method | Type | Pros | Cons |
|--------|------|------|------|
| **HTTP (REST)** | Synchronous | Simple, familiar | Caller waits; tight coupling |
| **Message Bus** (RabbitMQ, Azure Service Bus) | Asynchronous | Resilient; decoupled | Complex; harder to debug |

> 🔹 **Synchronous HTTP:** Service A calls Service B and waits for a response. If B is down, A fails.  
> 🔹 **Message Bus:** Service A publishes an event; Service B consumes it when ready. One failure doesn't cascade.

---

## ✅ When to Use What

| Scenario | Recommended Architecture |
|---------|------------------------|
| Small team, rapid delivery | MVC Monolith |
| Growing app, need separation | N-Tier or Clean Architecture |
| Enterprise, testability critical | Clean Architecture + CQRS |
| High scale, independent teams | Microservices |
