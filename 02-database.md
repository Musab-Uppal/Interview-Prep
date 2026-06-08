# 🗄️ Database

> **Interview Priority:** ⭐⭐⭐⭐⭐ — ERP and enterprise apps live and die by the database layer.

---

## 📐 Normalization

Normalization eliminates **data redundancy** and prevents **update anomalies** by organizing tables according to formal rules.

### 1NF — First Normal Form

**Rule:** No repeating groups; every column must hold a single, atomic value.

❌ **Bad — multi-valued column:**
```
Orders: OrderId | ProductIds
        1       | 10, 20, 30
```

✅ **Fixed — separate table:**
```
OrderItems: OrderItemId | OrderId | ProductId
            1           | 1       | 10
            2           | 1       | 20
```

> 💡 Storing comma-separated IDs makes `SELECT`, `JOIN`, and `UPDATE` extremely hard.

---

### 2NF — Second Normal Form

**Rule:** No partial dependency — every non-key column must depend on the **whole** primary key (relevant when the PK is composite).

❌ **Bad — `ProductName` depends only on `ProductId`, not the full PK `(OrderId, ProductId)`:**
```
OrderItems: OrderId | ProductId | Quantity | ProductName
```

✅ **Fixed — move `ProductName` to the `Products` table:**
```
OrderItems: OrderId | ProductId | Quantity
Products:   ProductId | ProductName | Price
```

---

### 3NF — Third Normal Form

**Rule:** No transitive dependency — non-key columns must not depend on other non-key columns.

❌ **Bad — `CityName` depends on `ZipCode`, not on `CustomerId`:**
```
Customers: CustomerId | ZipCode | CityName
```

✅ **Fixed — separate `ZipCodes` table:**
```
Customers: CustomerId | ZipCode
ZipCodes:  ZipCode    | CityName
```

### Normalization Quick Reference

| Form | Rule | Problem it Fixes |
|------|------|-----------------|
| **1NF** | Atomic values, no repeating groups | Multi-valued columns |
| **2NF** | No partial dependency on PK | Composite-key redundancy |
| **3NF** | No transitive dependency | Non-key columns depending on each other |

---

## 🔍 Indexing

Indexes are data structures that speed up queries by allowing the database engine to find rows without a full table scan.

### When to Add an Index

✅ Index these:
- **Foreign key columns** — speeding up `JOIN` operations
- **Columns used in `WHERE` clauses** — filtering
- **High-selectivity columns** — `email`, `userId`, `orderNumber` (few duplicates)

❌ Avoid indexing:
- Low-selectivity columns (e.g., `IsActive` with only `true`/`false`)
- Columns that are frequently updated (index maintenance overhead)

### Clustered vs Non-Clustered

| | Clustered | Non-Clustered |
|-|-----------|---------------|
| **Storage** | Data rows sorted and stored in index order | Separate structure with pointer to data row |
| **Count per table** | One (usually the PK) | Many allowed |
| **Speed** | Fastest for range queries | Fast for lookups on indexed column |

---

## 🔗 LINQ and EF Core

**LINQ** (Language Integrated Query) performs filtering and other operations on collections.  
**EF Core** is an ORM that translates LINQ expressions into SQL and handles read/write operations.

### Key Difference: `IQueryable` vs `IEnumerable`

| | `IQueryable<T>` | `IEnumerable<T>` |
|-|----------------|-----------------|
| **Execution** | Deferred — runs **in the database** | Runs **in memory** |
| **Use with** | EF Core (`DbSet<T>`) | In-memory collections, already-fetched data |
| **Performance** | Efficient — only fetches needed rows | Dangerous on large tables — loads everything first |

> ⚠️ Using `IEnumerable` on a large EF Core query is a common performance bug — it loads the entire table into memory before filtering.

### Execution is Deferred

Queries do not execute until a **terminal operator** is called:

```csharp
var query = _context.Products.Where(p => p.Price > 5000); // NOT executed yet

var results = await query.ToListAsync(); // executed here — SQL sent to DB
```

### LINQ Method Syntax (Lambda)

```csharp
var products = await _context.Products
    .Where(p => p.Price > 5000)
    .OrderBy(p => p.Name)
    .Select(p => new { p.Id, p.Name, p.Price })
    .ToListAsync();
```

### LINQ Query Syntax (SQL-like)

```csharp
var products = await (from p in _context.Products
                      where p.Price > 5000
                      orderby p.Name
                      select new { p.Id, p.Name, p.Price })
                     .ToListAsync();
```

> 💡 Both syntaxes compile to the same IL. Prefer **method syntax** — it's more composable and the industry standard in .NET projects.

### Useful EF Core Features

| Feature | Usage | Purpose |
|---------|-------|---------|
| `Include` / `ThenInclude` | `.Include(o => o.Items)` | Eager loading related data |
| `AsNoTracking()` | `.AsNoTracking()` | Read-only queries — faster, no change tracking |
| `FromSqlRaw` | `.FromSqlRaw("SELECT ...")` | Raw SQL when needed |
| `ToListAsync` / `ToArrayAsync` | Terminal operators | Execute the query asynchronously |

---

## 🛠️ Dapper

Dapper is a **micro-ORM**. You write raw SQL yourself; Dapper maps the result to C# objects.

```csharp
using var conn = new SqlConnection(_connectionString);
var products = await conn.QueryAsync<Product>(
    "SELECT Id, Name, Price FROM Products WHERE Price > @minPrice",
    new { minPrice = 5000 }
);
```

### EF Core vs Dapper

| | EF Core | Dapper |
|-|---------|--------|
| **SQL** | Auto-generated | You write it |
| **Change tracking** | ✅ Built-in | ❌ None |
| **Complex queries** | Can get verbose | ✅ Full SQL control |
| **Performance** | Good for CRUD | ✅ Faster for heavy reads |
| **Use when** | Standard CRUD + migrations | Complex reports / stored procedures |

> 💡 They complement each other — use **EF Core for CRUD** and **Dapper for heavy read queries** in the same project.

---

## 🔧 Code First vs DB First

### Code First

Use when the **database does not exist yet** (greenfield project).

1. Define C# entity classes.
2. Run migrations — EF Core creates the database schema.

```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
}
```

### DB First

Use when the **database already exists**.

Scaffold entity classes and `DbContext` from the existing database:

```bash
dotnet ef dbcontext scaffold "Server=.;Database=MyDb;Trusted_Connection=True;" \
    Microsoft.EntityFrameworkCore.SqlServer
```

EF Core generates model classes and the `DbContext` automatically.

### Comparison Table

| | Code First | DB First |
|-|-----------|---------|
| **Starting point** | C# classes | Existing database |
| **Schema control** | Via migrations | Via DBA / SQL scripts |
| **Source control** | Easy (migrations are code) | Harder (schema in DB) |
| **Best for** | New projects, small teams | Legacy databases, DBA-managed schemas |
