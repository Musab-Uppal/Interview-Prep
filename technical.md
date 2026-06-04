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
