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
