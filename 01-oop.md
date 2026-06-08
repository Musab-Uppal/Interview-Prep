# 🧠 OOP — Object-Oriented Programming

> **Interview Priority:** ⭐⭐⭐⭐⭐ — Almost every .NET interview starts here.

---

## 📖 What is OOP?

**Object-Oriented Programming (OOP)** is a paradigm that organizes code around **objects** — each representing a real-world entity with data (properties) and behavior (methods).

### OOP vs Procedural — Why OOP?

| Feature | Procedural | Object-Oriented |
|---------|-----------|-----------------|
| **Structure** | Functions and procedures | Classes and objects |
| **Data** | Exposed, accessible anywhere | Hidden via encapsulation |
| **Reusability** | Functions only | Classes reused via inheritance |
| **Maintainability** | Hard to scale | Easier to maintain and extend |
| **Real-World Modeling** | Difficult | Natural fit |

---

## 🔒 Encapsulation

### Core Idea

Encapsulation **binds data and methods** into one unit (a class). More importantly, it **protects invariants** — it prevents an object from entering an inconsistent state.

> 💡 Example: a setter that ensures a product price can never be 0 or negative.

```csharp
public class Product
{
    private decimal _price;

    public decimal Price
    {
        get => _price;
        set
        {
            if (value <= 0)
                throw new ArgumentException("Price must be greater than zero.");
            _price = value;
        }
    }
}
```

### Access Modifiers

| Modifier | Visible To |
|----------|-----------|
| `public` | Everywhere |
| `private` | Same class only |
| `protected` | Same class + derived classes |
| `internal` | Same assembly |
| `protected internal` | Same assembly or derived classes |

---

## 🎭 Abstraction

### Core Idea

Abstraction means **programming to contracts, not implementations**. You expose what a thing *does*, not *how* it does it.

> 💡 Real-world: you press "Pay" — you don't care if it routes to CreditCard, JazzCash, or EasyPaisa internally.

### Using Interfaces

```csharp
public interface IPaymentService
{
    Task<bool> ProcessPayment(decimal amount);
}

public class CreditCardPayment : IPaymentService
{
    public Task<bool> ProcessPayment(decimal amount)
    {
        // Credit card specific logic
        return Task.FromResult(true);
    }
}

public class JazzCashPayment : IPaymentService
{
    public Task<bool> ProcessPayment(decimal amount)
    {
        // JazzCash specific logic
        return Task.FromResult(true);
    }
}
```

**Why it matters:** register a different implementation in `Program.cs` and your controller never changes:

```csharp
// Program.cs
builder.Services.AddScoped<IPaymentService, CreditCardPayment>();
// Swap anytime: builder.Services.AddScoped<IPaymentService, JazzCashPayment>();
```

### Interface vs Abstract Class

| | `interface` | `abstract class` |
|-|------------|-----------------|
| **Methods** | Signatures only (default impls from C# 8) | Can have both abstract and concrete methods |
| **Fields** | No fields | Can have fields |
| **Multiple inheritance** | ✅ A class can implement many | ❌ Single inheritance only |
| **Use when** | Defining a contract/capability | Sharing base implementation with extension points |

---

## 🧬 Inheritance

### Core Idea

Inheritance means a **derived class gets all properties and methods of a base class**, plus its own. Use it **only** where an "is-a" relationship exists.

> ✅ `EmailNotification` **is a** `Notification` — good  
> ❌ `Car` **is a** `Engine` — wrong; use composition instead

```csharp
public abstract class Notification
{
    public string Message { get; set; }
    public abstract Task SendAsync(); // must be implemented by child
}

public class EmailNotification : Notification
{
    public override async Task SendAsync()
    {
        // send via SMTP
    }
}

public class SmsNotification : Notification
{
    public override async Task SendAsync()
    {
        // send via SMS gateway
    }
}
```

**Real example in ASP.NET Core:**
```csharp
public class MyController : ControllerBase
{
    // ControllerBase gives us Ok(), BadRequest(), etc.
}
```

---

## 🔄 Polymorphism

### Core Idea

**Polymorphism** means "many forms" — the same method name behaves differently depending on the object that calls it.

### Types

| Type | How | When |
|------|-----|------|
| **Compile-time (Static)** | Method overloading | Different signatures, same name |
| **Runtime (Dynamic)** | Method overriding (`virtual`/`override`) | Same signature, different behavior |

```csharp
public abstract class ExportService
{
    // Template method — defines the algorithm
    public abstract byte[] Export(IEnumerable<object> data);
}

public class PdfExportService : ExportService
{
    public override byte[] Export(IEnumerable<object> data)
    {
        // PDF-specific logic
        return Array.Empty<byte>();
    }
}

public class CsvExportService : ExportService
{
    public override byte[] Export(IEnumerable<object> data)
    {
        // CSV-specific logic
        return Array.Empty<byte>();
    }
}
```

### `virtual` vs `abstract` vs `override`

| Keyword | Meaning |
|---------|---------|
| `virtual` | Base class provides a **default implementation**; can be overridden |
| `abstract` | No implementation; **must** be overridden in derived class |
| `override` | Used in derived class to **replace** the base behavior |
| `sealed` | Prevents further overriding |

> ⚠️ Hiding a virtual method without `override` compiles but the compiler issues a warning — always use `override` explicitly.

---

## 🏛️ SOLID Principles

| Letter | Principle | Summary |
|--------|-----------|---------|
| **S** | Single Responsibility | A class should have **one reason to change** |
| **O** | Open/Closed | **Open** for extension, **closed** for modification |
| **L** | Liskov Substitution | Derived classes must be **substitutable** for their base types |
| **I** | Interface Segregation | Don't force classes to implement methods **they won't use** |
| **D** | Dependency Inversion | High-level modules depend on **abstractions**, not concrete implementations |

### Quick Examples

```csharp
// ✅ S — OrderService only handles orders, not emails
public class OrderService { /* order logic */ }
public class EmailService  { /* email logic */ }

// ✅ O — Add new export types without touching existing code
public class PdfExport : IExportService { }
public class CsvExport : IExportService { }

// ✅ L — both work wherever INotification is expected
INotification n = new EmailNotification();
INotification n = new SmsNotification();

// ✅ I — separate interfaces so classes only implement what they need
public interface IPrintable { void Print(); }
public interface IScannable { void Scan(); }

// ✅ D — controller depends on interface, not implementation
public class OrdersController(IOrderService _orderService) { }
```

---

## ✅ Summary Table

| Concept | One-Liner | C# Keywords |
|---------|-----------|-------------|
| **Encapsulation** | Bind data + protect invariants | `private`, getters, setters |
| **Abstraction** | Contract over implementation | `interface`, `abstract class` |
| **Inheritance** | "Is-a" — reuse base class | `: BaseClass` |
| **Polymorphism** | Same method, different behavior | `virtual`, `abstract`, `override` |
| **SOLID-S** | One class, one job | — |
| **SOLID-O** | Extend without modifying | New class implements interface |
| **SOLID-L** | Derived replaces base safely | Proper inheritance |
| **SOLID-I** | Small, focused interfaces | Multiple `interface` declarations |
| **SOLID-D** | Depend on abstractions | DI + `interface` |
