# 📋 Junior .NET Developer — Interview Prep

**Complete, Structured, Interview-Ready Notes**

A focused collection of everything you need to crack a junior .NET developer interview — OOP, databases, ASP.NET Core, APIs, architectures, authentication, C# fundamentals, and behavioural prep.

---

## 📚 Table of Contents

| # | Topic | File | What's Inside |
|---|-------|------|---------------|
| 1 | 🧠 OOP | [01-oop.md](01-oop.md) | 4 Pillars, SOLID, C# examples |
| 2 | 🗄️ Database | [02-database.md](02-database.md) | Normalization, Indexing, LINQ, EF Core, Dapper |
| 3 | ⚙️ ASP.NET Core | [03-aspnet-core.md](03-aspnet-core.md) | Middleware, DI lifetimes, Pipeline |
| 4 | 🔌 API | [04-api.md](04-api.md) | REST, HTTP Methods, Why APIs |
| 5 | 🏗️ Architectures | [05-architectures.md](05-architectures.md) | MVC, Clean Architecture, CQRS, Microservices |
| 6 | 🔐 Auth | [06-authentication-authorization.md](06-authentication-authorization.md) | JWT, Roles, Claims, Policies, Token Storage |
| 7 | 💻 C# Fundamentals | [07-csharp-fundamentals.md](07-csharp-fundamentals.md) | Short differences, .NET Core vs Framework, JWT vs Session |
| 8 | 🤝 Behavioural | [08-behavioural.md](08-behavioural.md) | Interview tips, resume advice, STAR method |


---

## ⚡ Quick Reference

### The 4 OOP Pillars
| Pillar | One-liner | C# Keyword |
|--------|-----------|------------|
| **Encapsulation** | Bind data + protect invariants | `private`, getters/setters |
| **Abstraction** | Program to contracts, not implementations | `interface`, `abstract` |
| **Inheritance** | "Is-a" relationship, reuse base class | `: BaseClass` |
| **Polymorphism** | Same method, different behavior | `virtual`, `override` |

### DI Lifetimes
| Lifetime | When to use |
|----------|-------------|
| `AddTransient` | New instance every time it's requested |
| `AddScoped` | One instance per HTTP request |
| `AddSingleton` | One instance for the application lifetime |

### HTTP Methods
| Method | Action | Has Body |
|--------|--------|----------|
| `GET` | Read | ❌ |
| `POST` | Create | ✅ |
| `PUT` | Replace entire resource | ✅ |
| `PATCH` | Partial update | ✅ |
| `DELETE` | Delete | ❌ |

---

## 🗺️ How to Use This Repo

1. **Start with OOP** — the most common interview topic for any .NET role.
2. **Then Database** — ERP and enterprise apps live and die by their DB layer.
3. **ASP.NET Core + API** — the runtime and contracts of your apps.
4. **Architectures** — shows you understand the bigger picture.
5. **Auth** — JWT/session is always asked.
6. **C# Fundamentals** — quick-fire difference questions.
7. **Behavioural** — don't neglect this; it's half the interview.

---

*Last updated: June 2026*
