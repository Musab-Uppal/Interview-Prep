# 🧪 Testing

A complete guide to unit testing in .NET — one of the most commonly asked topics in junior developer interviews.

---

## 📚 Table of Contents

- [Why Testing Matters](#why-testing-matters)
- [Types of Tests](#types-of-tests)
- [Testing Frameworks](#testing-frameworks)
- [AAA Pattern](#-aaa-pattern-arrange-act-assert)
- [xUnit Key Attributes](#xunit-key-attributes)
- [Mocking with Moq](#-mocking-with-moq)
- [Common Interview Questions](#-common-interview-questions)

---

## Why Testing Matters

Testing shows you write **production-quality code**. Junior devs are expected to know:
- The difference between unit, integration, and E2E tests
- How to structure a test using the AAA pattern
- How to mock dependencies using Moq

---

## Types of Tests

| Type        | What It Tests                        | Speed  | Example                          |
|-------------|--------------------------------------|--------|----------------------------------|
| Unit        | A single class/method in isolation   | Fast   | Testing a `Calculator.Add()` method |
| Integration | Multiple components working together | Medium | Controller + database layer      |
| E2E         | Full app flow from UI to DB          | Slow   | Selenium / Playwright tests      |

> 💡 **Rule of thumb:** Write lots of unit tests, fewer integration tests, and minimal E2E tests (the Testing Pyramid).

---

## Testing Frameworks

| Framework | Notes                                        |
|-----------|----------------------------------------------|
| xUnit     | Most popular in .NET, used by the ASP.NET team |
| NUnit     | Older, still widely used in enterprise apps  |
| MSTest    | Microsoft's built-in option, less common now |

> ✅ Default to **xUnit** unless the project already uses something else.

---

## 🔺 AAA Pattern (Arrange, Act, Assert)

The standard structure every unit test should follow.

```csharp
[Fact]
public void Add_TwoNumbers_ReturnsCorrectSum()
{
    // Arrange — set up objects and inputs
    var calculator = new Calculator();

    // Act — call the method being tested
    var result = calculator.Add(2, 3);

    // Assert — verify the result is correct
    Assert.Equal(5, result);
}
```

| Phase   | Purpose                                      |
|---------|----------------------------------------------|
| Arrange | Set up the system under test and its inputs  |
| Act     | Execute the method or behavior being tested  |
| Assert  | Verify the output matches the expectation    |

---

## xUnit Key Attributes

| Attribute      | Use                                          |
|----------------|----------------------------------------------|
| `[Fact]`       | A single test with no parameters             |
| `[Theory]`     | A test that runs with multiple data sets     |
| `[InlineData]` | Provides parameter values to a `[Theory]`    |

### Example — `[Theory]` with `[InlineData]`

```csharp
[Theory]
[InlineData(2, 3, 5)]
[InlineData(0, 0, 0)]
[InlineData(-1, 1, 0)]
public void Add_ReturnsCorrectSum(int a, int b, int expected)
{
    // Arrange
    var calculator = new Calculator();

    // Act
    var result = calculator.Add(a, b);

    // Assert
    Assert.Equal(expected, result);
}
```

### Common xUnit Assertions

| Assertion                        | Checks                              |
|----------------------------------|-------------------------------------|
| `Assert.Equal(expected, actual)` | Values are equal                    |
| `Assert.NotNull(obj)`            | Object is not null                  |
| `Assert.True(condition)`         | Condition is true                   |
| `Assert.Throws<T>(() => ...)`    | A specific exception is thrown      |
| `Assert.Contains(item, list)`    | Collection contains the item        |

---

## 🃏 Mocking with Moq

Mocking lets you **isolate the class under test** by replacing real dependencies with fakes.

### Why mock?

If your service depends on a database or an email sender, you don't want real calls in unit tests. Mocks simulate those dependencies without side effects.

### Basic Moq Example

```csharp
// The dependency interface
public interface IEmailService
{
    void SendEmail(string to, string message);
}

// The service that uses it
public class UserService
{
    private readonly IEmailService _emailService;

    public UserService(IEmailService emailService)
    {
        _emailService = emailService;
    }

    public void Register(string email)
    {
        // ... registration logic ...
        _emailService.SendEmail(email, "Welcome!");
    }
}

// The unit test
[Fact]
public void Register_ValidUser_SendsWelcomeEmail()
{
    // Arrange
    var mockEmail = new Mock<IEmailService>();
    var userService = new UserService(mockEmail.Object);

    // Act
    userService.Register("test@example.com");

    // Assert — verify the email was sent exactly once
    mockEmail.Verify(
        e => e.SendEmail("test@example.com", It.IsAny<string>()),
        Times.Once
    );
}
```

### Key Moq Members

| Member            | What It Does                                      |
|-------------------|---------------------------------------------------|
| `new Mock<T>()`   | Creates a fake implementation of interface T      |
| `.Object`         | Gets the fake object to inject into your class    |
| `.Setup()`        | Configures what the mock returns when called      |
| `.Returns()`      | Specifies the return value for a Setup            |
| `.Verify()`       | Asserts that a method was called                  |
| `It.IsAny<T>()`   | Matches any argument of type T                    |
| `Times.Once`      | Verifies the method was called exactly one time   |
| `Times.Never`     | Verifies the method was never called              |

### Setup with Return Value

```csharp
var mockRepo = new Mock<IUserRepository>();

mockRepo
    .Setup(r => r.GetById(1))
    .Returns(new User { Id = 1, Name = "Areeba" });

var user = mockRepo.Object.GetById(1);
Assert.Equal("Areeba", user.Name);
```

---

## 💡 FIRST Principles (What Makes a Good Unit Test)

| Letter | Principle   | Meaning                                         |
|--------|-------------|-------------------------------------------------|
| F      | Fast        | Tests should run in milliseconds                |
| I      | Isolated    | Tests should not depend on each other           |
| R      | Repeatable  | Same result every run, no flakiness             |
| S      | Self-validating | Pass or fail without manual inspection     |
| T      | Timely      | Written alongside or before the code (TDD)      |

---

## 🤝 Common Interview Questions

**Q: What is the difference between a mock and a stub?**
> A stub returns hardcoded data. A mock also **verifies behavior** — i.e., it checks that certain methods were called with specific arguments.

**Q: Why do we use interfaces for mocking?**
> Moq creates a fake class at runtime that implements the interface. Without an interface (or a virtual method), there's nothing for Moq to override.

**Q: What is code coverage?**
> The percentage of your production code that is executed during tests. 80%+ is a common target, but chasing 100% isn't always practical — focus on testing business logic, not trivial getters/setters.

**Q: What should you NOT unit test?**
> External systems like databases, APIs, and the file system — those belong in integration tests. Unit tests should be fast and fully isolated.

**Q: What is Test-Driven Development (TDD)?**
> Writing the test *before* the implementation. The cycle is: Red (test fails) → Green (write code to pass it) → Refactor (clean up).

**Q: What is the Testing Pyramid?**
> A model recommending: many unit tests at the base, fewer integration tests in the middle, and very few E2E tests at the top. It keeps the test suite fast and maintainable.

---

*Last updated: June 2026*
