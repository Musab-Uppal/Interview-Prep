# OOP

## Encapsulation

### Protecting Invariants
Encapsulation is binding data members and fucntionalities into one unit but that not enough it also allow us to prevent the object going into inconsistent state we define business rules and setter that product price cant be 0 or -ve etc

## Abstraction

### Dependency Inversion/Injection
Abstraction means programming to contracts not implementations. We provide interfaces to controllers or services rather than implementations. It not allow us to easily swap impelementations by registering new implementation in program.cs but also makes testing easier.
Example payments we make payment interface dont know or care about if its credit card or jazzcash
high level modules never depend on low level implementations 

## Inheritance

### is a relationship

Means object is having same properties and methods as base class plus its own but is must be used where there is an 'is a' relationship 
Example APIController:ControllerBase
NotificationBase
then email notification, payment etc

## Polymorphism

### open/close principle
Same function behaving differently depending on obect 
applies open close principle we define types of export pdf,csv,doc etc
game character draw object
used by override keyword 
abtract classes have functions that define basic behaviuor and we can override them in child classes

virtual keyword gives default behaviour 
abstract class and abstract function says i must be implemented in child class
override keyword must be used when implementing virtual funciton if we dont write it we are hiding functionality and compiler gives warning 

abstract class ka object ni bny ga isme pure virual b ho skta or implementations b ho skti

interface me only sigratures no implementations

### SOLID

#### S 
Single responsibility
#### O
open close
#### L
liskov substitution principle base class pointer can point derive class object (dependency injection)
#### I
inversion seggregation dont force classes to implement funciton that they will not use
#### D
dependency inversion high level modules should not depend on low level implementations use interfaces


# LINQ and EF Core

Linq is language integrated query it just performs some filtering or other functions on in memory enumrables while EF Core is a complete database framework it communicate with database for us performing read write quries it also converts linq queries into db query 
EF Core also have some special functions which Linq itself dont have like include which is for join and thenInclude for double join and asnotracking to get only readonly data its just a little faster
EF Core also have from raw sql function to directly write SQL
Pure LINQ uses .ToList(), EF Core uses .ToListAsync()

- LINQ is the query syntax that works on any collection. EF Core uses LINQ on IQueryable to translate C# expressions into SQL at the database level. The critical difference is IQueryable defers execution and runs on the DB, while IEnumerable runs in memory — using the wrong one on large tables is a classic performance bug.

## Types of LINQ Quries

### 1. Method Syntax (Lambda) — what everyone uses
- var products = _context.Products
    .Where(p => p.Price > 5000)
    .OrderBy(p => p.Name)
    .Select(p => p.Name)
    .ToListAsync();

### 2. Query Syntax (SQL-like) — rarely used in modern .NET
- var products = (from p in _context.Products
                where p.Price > 5000
                orderby p.Name
                select p.Name)
               .ToList();

## Dapper 
Dapper is a micro ORM. You write raw SQL yourself, Dapper just maps the results to your C# objects. Nothing more.
### Where to Use Dapper
- Not a replacement for EF Core — use both together. EF Core for standard CRUD, Dapper for complex heavy queries.
# DB First and Code First approaches

## Code First
- Use Code first when db does not exists yet and for greenfield projects
- easy to track, if team is small and project is not complex
- complex databases are difficult to map in code 
- make files then run command 
dotnet ef migrations add InitialCreate
dotnet ef database update

## DB First
When database already exists just run the command and class files are automatically generated 
dotnet ef dbcontext scaffold "ConnectionString" 
it also generate db context class 


# Database

## 1NF
- no repeating group like phone no: can be more than one if add new row then data redundancy if array then conflicts 
- Multiple values create problems in performing operations like select or join
- solution create another table with order items and add order id as foreign key
- remove multivalued attributes into another table

## 2NF

- no partial dependency every column should be completely depend on pk no any other column
like if order items contains product name it will depend on product id so we will move product name to product table only 
- remove partial dependency

## 3NF
- no transitive dependency the non-key should not depend on other non-key columns
- remove transitive dependency create a whole new table and put all dependents and determinent into it and keep the determinent in the orignal table as foriegn key.

## Indexing
- Data structure to faster the querying process on database 
- foreign keys should be indexed sql server does not do it by default
- also the columns which are used in where clauses
- the column which mostly contain unique data like email userid etc in orders table the column which narrow down to tens of rows 

# Asp .Net Core
- cross platform to create optimized web apps or web apis developed by microsoft
- program.cs is the entry point we create builder register services then define middleware pipeline 
- middle are the functions that executes in the https request response pipeline used to filter out or verify requests
- Asp.Net Core built in dependency injection
- AddTransient when new instance for every new request
- AddScoped one instance for one HTTP request
- AddSingleton one insance for the whole app
- AllowAnonymous for the end points that completely public not required authentication like home page
- Dependency injection is when a class recieves its dependencies via constructor instead of creating inside it itself, it makes classes losely coupled, make independent testing possible and easy to swap whenever needed

# What is API?

- "API stands for Application Programming Interface. It's a contract between two systems that defines how they communicate. In web development, a REST API exposes endpoints over HTTP — the client sends a request to a URL, the server processes it and returns JSON."

## Why APIs Exist — 3 Points
###  Separation 
- frontend and backend are independent, different teams can work on each
###  Reusability 
- same API serves web app, mobile app, and third party integrations
### Security 
- database is never exposed directly, API controls what goes in and out

# Architectures

## MVC
- Stands for model view controller full stack architecture the controllers map the url and takes request process then return .cshtml files as response rather than json 
- tightly couples used for monolithic apps and small apps because as app grows code become complex

## Monolith
- Everything in a single repo the frontend,backend and database tightly couples
## N Tier
- Mono repo but we have layers like presentation, database, services etc
## Clean Architecture
WebAPI → Application ← Infrastructure
              ↓
           Domain
- Dependency goes inward only the business layer is not dependent on any other layer but still a monolithic architecture
### Layers
Domain and application does not cummunicate with external libs like EF Core Infrastructure does that
#### Domain Layer
- Define entities and business rules here 
- Business rules inside entities

#### Application Layer
- Define Services,interfaces and DTO's here but not implementations

#### Infrastructure Layer
- Implement the interfaces here, write sql server queries in repositories here

#### Presentation Layer
- Write controllers here which exposes end point to clients
### CQRS (Command Query Responsibility Segregation)
- Often used alongside Clean Architecture. Separates read operations from write operations.
- Write id treated as command it changes state add or modify some data return nothing or an ID does not require to be fast
- Read is treated as a query and it should be fast and it never changes state
- In .NET this is usually implemented with MediatR library — handlers for each command/query. Just know the concept, mention MediatR if asked.
## Microservice
- Each service has its own responsibility and own database and each service is deployed separately 
### Cummunication

#### via HTTP
- This is called synchrnous cummunication each service waits untill the called service reponds

#### via Message Bus
- This is asynchrnous cummunication each service does its part of the job and put the request in the message bus other service consume it from the bus
- One service fail does not crash the whole app
- Complex required mature devOps engineers


# Authentication and Authorization
Request comes in
      ↓
Authentication middleware — reads token, identifies user
      ↓
Authorization middleware — checks if this user can access this route
      ↓
Controller
- They play their roles in the middleware
## Authentication Who are you?
- Login -> verify identity -> issue token
- It asks the user who are you do you belong here or not

### JWT (Json Web Token)

- A JWT is 3 parts separated by dots: 
- eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjMifQ.SflKxwRJS
-       Header                  Payload          Signature

// Header — algorithm used
{ "alg": "HS256", "typ": "JWT" }

// Payload — claims (user data)
{
  "sub": "user-guid-here",
  "email": "musab@gmail.com",
  "role": "Admin",
  "exp": 1718000000   // expiry timestamp
}

// Signature — Header + Payload signed with secret key
// If anyone tampers with payload, signature won't match → rejected
HMACSHA256(base64(header) + "." + base64(payload), secretKey)

- Important: JWT payload is base64 encoded, not encrypted. Anyone can decode and read it. Never put sensitive data like passwords in it.

## AUTHORIZATION What can you do?
- 3 types in ASP.NET Core — Role, Claims, Policy.
### Type 1 — Role Based (Most Common)
[Authorize]                       // any authenticated user
[Authorize(Roles = "Admin")]      // only Admin
[Authorize(Roles = "Admin,Manager")] // Admin OR Manager

### Type 2 — Claims Based
- Claims are key-value pairs inside the token. More granular than roles.

### Type 3 — Policy Based (Most Flexible)
- Policies let you combine multiple rules into one named requirement.