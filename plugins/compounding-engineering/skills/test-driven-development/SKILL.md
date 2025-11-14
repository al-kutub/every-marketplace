---
name: test-driven-development
description: Master Test-Driven Development (TDD) methodology with Red-Green-Refactor cycle. Use when implementing features, writing unit tests, integration tests, or building complex systems where quality and maintainability are critical.
---

# Test-Driven Development (TDD)

Master the Test-Driven Development (TDD) methodology with the Red-Green-Refactor cycle to ensure high-quality, maintainable code with comprehensive test coverage.

## When to Use This Skill

- Implementing new features with high quality requirements
- Building complex systems requiring reliable behavior
- Writing APIs and services with multiple integration points
- Refactoring legacy code to add test coverage
- Building components in React, Vue, Angular with confidence
- Implementing business logic in .NET, Python, TypeScript
- Creating database access layers and repositories
- Building microservices with proper testing
- When minimum 80% test coverage is required

## TDD Core Principles

### 1. Red-Green-Refactor Cycle

**Red Phase: Write Failing Tests First**
- Write tests BEFORE implementation
- Tests should fail (because code doesn't exist yet)
- Focus on what the code SHOULD do
- Clarify requirements through tests
- Define the API/interface

**Green Phase: Write Minimal Code**
- Write ONLY code to make tests pass
- Don't over-engineer or optimize
- Focus on functionality, not perfection
- All tests should pass
- Ignore edge cases beyond test scope

**Refactor Phase: Clean and Improve**
- Improve code quality
- Remove duplication
- Optimize performance
- Enhance readability
- All tests still pass

### 2. Benefits of TDD

✅ **Quality Assurance**
- Catch bugs early in development
- Ensure code behaves as expected
- Higher overall code quality
- Fewer production issues

✅ **Documentation**
- Tests serve as living documentation
- Show how to use the code
- Clarify expected behavior
- Examples for other developers

✅ **Design Improvement**
- Forces better API design
- Loose coupling through dependency injection
- Testable code is better structured
- Encourages separation of concerns

✅ **Refactoring Confidence**
- Safe to refactor with test coverage
- Catch regressions immediately
- Enables continuous improvement
- Reduces fear of breaking things

✅ **Speed Over Time**
- Slower initially per feature
- Faster overall due to fewer bugs
- Less time debugging
- Faster feature delivery long-term

## TDD Workflow

### Step 1: Write Tests (RED)

```csharp
// ✅ EXAMPLE: .NET Unit Test (xUnit)
[TestClass]
public class UserServiceTests
{
    private IUserRepository _userRepositoryMock;
    private UserService _userService;

    [TestInitialize]
    public void Setup()
    {
        _userRepositoryMock = new Mock<IUserRepository>();
        _userService = new UserService(_userRepositoryMock.Object);
    }

    [TestMethod]
    [ExpectedException(typeof(UserNotFoundException))]
    public async Task GetUserAsync_WithInvalidId_ThrowsUserNotFoundException()
    {
        // Arrange: Set up test conditions
        int invalidUserId = 999;
        _userRepositoryMock
            .Setup(r => r.GetByIdAsync(invalidUserId))
            .ReturnsAsync((User)null);

        // Act: Execute the operation being tested
        await _userService.GetUserAsync(invalidUserId);

        // Assert: Verify expected exception (implicit via ExpectedException)
    }

    [TestMethod]
    public async Task GetUserAsync_WithValidId_ReturnsUser()
    {
        // Arrange
        var expectedUser = new User
        {
            Id = 1,
            Email = "test@example.com",
            Role = UserRoleEnum.User
        };

        _userRepositoryMock
            .Setup(r => r.GetByIdAsync(1))
            .ReturnsAsync(expectedUser);

        // Act
        var result = await _userService.GetUserAsync(1);

        // Assert
        Assert.IsNotNull(result);
        Assert.AreEqual(expectedUser.Email, result.Email);
        Assert.AreEqual(expectedUser.Role, result.Role);

        // Verify the mock was called exactly once
        _userRepositoryMock.Verify(r => r.GetByIdAsync(1), Times.Once);
    }

    [TestMethod]
    public async Task CreateUserAsync_WithValidData_ReturnsCreatedUser()
    {
        // Arrange
        var createRequest = new CreateUserRequest
        {
            Email = "newuser@example.com",
            FirstName = "John",
            LastName = "Doe"
        };

        var expectedUser = new User
        {
            Id = 1,
            Email = createRequest.Email,
            FirstName = createRequest.FirstName,
            LastName = createRequest.LastName,
            Role = UserRoleEnum.User
        };

        _userRepositoryMock
            .Setup(r => r.CreateAsync(It.IsAny<User>()))
            .ReturnsAsync(expectedUser);

        // Act
        var result = await _userService.CreateUserAsync(createRequest);

        // Assert
        Assert.AreEqual(expectedUser.Email, result.Email);
        Assert.AreEqual(expectedUser.FirstName, result.FirstName);
        _userRepositoryMock.Verify(r => r.CreateAsync(It.IsAny<User>()), Times.Once);
    }
}
```

```python
# ✅ EXAMPLE: Python Unit Test (pytest)
import pytest
from unittest.mock import Mock, patch
from app.services.user_service import UserService
from app.exceptions import UserNotFoundException

class TestUserService:
    @pytest.fixture
    def user_service(self):
        """Set up UserService with mocked repository"""
        mock_repository = Mock()
        return UserService(mock_repository), mock_repository

    def test_get_user_with_invalid_id_raises_exception(self, user_service):
        """Test that getting non-existent user raises UserNotFoundException"""
        # Arrange
        service, mock_repo = user_service
        invalid_user_id = 999
        mock_repo.find_by_id.return_value = None

        # Act & Assert
        with pytest.raises(UserNotFoundException):
            service.get_user(invalid_user_id)

    def test_get_user_with_valid_id_returns_user(self, user_service):
        """Test that getting existing user returns correct user"""
        # Arrange
        service, mock_repo = user_service
        expected_user = {
            'id': 1,
            'email': 'test@example.com',
            'role': 'user'
        }
        mock_repo.find_by_id.return_value = expected_user

        # Act
        result = service.get_user(1)

        # Assert
        assert result == expected_user
        assert result['email'] == 'test@example.com'
        mock_repo.find_by_id.assert_called_once_with(1)

    def test_create_user_with_valid_data_returns_created_user(self, user_service):
        """Test that creating user with valid data returns created user"""
        # Arrange
        service, mock_repo = user_service
        create_request = {
            'email': 'newuser@example.com',
            'first_name': 'John',
            'last_name': 'Doe'
        }
        expected_user = {
            'id': 1,
            **create_request,
            'role': 'user'
        }
        mock_repo.create.return_value = expected_user

        # Act
        result = service.create_user(create_request)

        # Assert
        assert result['email'] == 'newuser@example.com'
        assert result['first_name'] == 'John'
        mock_repo.create.assert_called_once()
```

```typescript
// ✅ EXAMPLE: TypeScript/React Test (Jest + React Testing Library)
import { render, screen, waitFor, fireEvent } from '@testing-library/react';
import { UserProfile } from './UserProfile';
import * as userService from '@/services/userService';

jest.mock('@/services/userService');

describe('UserProfile Component', () => {
    beforeEach(() => {
        jest.clearAllMocks();
    });

    it('should display loading state initially', () => {
        // Arrange: Mock async operation with delay
        (userService.getUser as jest.Mock).mockImplementation(
            () => new Promise(resolve => setTimeout(resolve, 1000))
        );

        // Act
        render(<UserProfile userId={1} />);

        // Assert
        expect(screen.getByTestId('loading-spinner')).toBeInTheDocument();
    });

    it('should display user data when loaded successfully', async () => {
        // Arrange
        const mockUser = {
            id: 1,
            email: 'test@example.com',
            firstName: 'John',
            lastName: 'Doe'
        };

        (userService.getUser as jest.Mock).mockResolvedValue(mockUser);

        // Act
        render(<UserProfile userId={1} />);

        // Assert
        await waitFor(() => {
            expect(screen.getByText('John Doe')).toBeInTheDocument();
            expect(screen.getByText('test@example.com')).toBeInTheDocument();
        });
    });

    it('should display error message when fetch fails', async () => {
        // Arrange
        (userService.getUser as jest.Mock).mockRejectedValue(
            new Error('Failed to fetch user')
        );

        // Act
        render(<UserProfile userId={1} />);

        // Assert
        await waitFor(() => {
            expect(screen.getByTestId('error-message')).toBeInTheDocument();
            expect(screen.getByText(/failed to fetch user/i)).toBeInTheDocument();
        });
    });

    it('should call onUpdate callback when update button is clicked', async () => {
        // Arrange
        const mockUser = {
            id: 1,
            email: 'test@example.com',
            firstName: 'John',
            lastName: 'Doe'
        };
        const onUpdateMock = jest.fn();

        (userService.getUser as jest.Mock).mockResolvedValue(mockUser);

        // Act
        render(<UserProfile userId={1} onUpdate={onUpdateMock} />);

        await waitFor(() => {
            expect(screen.getByText('John Doe')).toBeInTheDocument();
        });

        fireEvent.click(screen.getByRole('button', { name: /update/i }));

        // Assert
        expect(onUpdateMock).toHaveBeenCalledWith(mockUser);
    });
});
```

### Step 2: Watch Tests Fail (RED)

```bash
# Run tests - they should ALL FAIL
npm test

# Output:
# FAIL  src/__tests__/UserService.test.ts
# ✕ UserService › should create user
#   ReferenceError: UserService is not defined
#
# Tests: 0 passed, 1 failed
```

### Step 3: Write Minimal Implementation (GREEN)

```csharp
// ✅ MINIMAL IMPLEMENTATION - Just enough to pass tests
public class UserService
{
    private readonly IUserRepository _repository;

    public UserService(IUserRepository repository)
    {
        _repository = repository ?? throw new ArgumentNullException(nameof(repository));
    }

    public async Task<User> GetUserAsync(int userId)
    {
        var user = await _repository.GetByIdAsync(userId);

        if (user == null)
            throw new UserNotFoundException(userId);

        return user;
    }

    public async Task<User> CreateUserAsync(CreateUserRequest request)
    {
        var user = new User
        {
            Email = request.Email,
            FirstName = request.FirstName,
            LastName = request.LastName,
            Role = UserRoleEnum.User
        };

        return await _repository.CreateAsync(user);
    }
}

public class UserNotFoundException : Exception
{
    public UserNotFoundException(int userId)
        : base($"User with ID {userId} not found.")
    {
    }
}
```

```bash
# Run tests - they should ALL PASS
npm test

# Output:
# PASS  src/__tests__/UserService.test.ts
# ✓ should create user
# ✓ should get user by id
#
# Tests: 3 passed, 3 passed
```

### Step 4: Refactor (REFACTOR)

```csharp
// ✅ IMPROVED: Extract validation, add logging, improve structure
public class UserService : IUserService
{
    private readonly IUserRepository _repository;
    private readonly IValidator<CreateUserRequest> _validator;
    private readonly ILogger<UserService> _logger;

    public UserService(
        IUserRepository repository,
        IValidator<CreateUserRequest> validator,
        ILogger<UserService> logger)
    {
        _repository = repository ?? throw new ArgumentNullException(nameof(repository));
        _validator = validator ?? throw new ArgumentNullException(nameof(validator));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }

    public async Task<User> GetUserAsync(int userId)
    {
        _logger.LogInformation("Fetching user with ID: {UserId}", userId);

        var user = await _repository.GetByIdAsync(userId);

        if (user == null)
        {
            _logger.LogWarning("User not found. ID: {UserId}", userId);
            throw new UserNotFoundException(userId);
        }

        _logger.LogInformation("User fetched successfully. ID: {UserId}", userId);
        return user;
    }

    public async Task<User> CreateUserAsync(CreateUserRequest request)
    {
        _logger.LogInformation("Creating new user with email: {Email}", request.Email);

        await ValidateCreateRequestAsync(request);

        var user = new User
        {
            Email = request.Email,
            FirstName = request.FirstName,
            LastName = request.LastName,
            Role = UserRoleEnum.User
        };

        var createdUser = await _repository.CreateAsync(user);

        _logger.LogInformation("User created successfully. ID: {UserId}", createdUser.Id);
        return createdUser;
    }

    private async Task ValidateCreateRequestAsync(CreateUserRequest request)
    {
        var validationResult = await _validator.ValidateAsync(request);

        if (!validationResult.IsValid)
        {
            _logger.LogWarning("Validation failed for create user request");
            throw new ValidationException(validationResult.Errors);
        }
    }
}
```

```bash
# Run tests again - they should STILL PASS
npm test

# Output:
# PASS  src/__tests__/UserService.test.ts
# ✓ should create user
# ✓ should get user by id
# ✓ should validate input before creating
#
# Tests: 4 passed, 4 passed
```

## TDD Best Practices

### ✅ DO

- **Write tests first**, always
- **Use descriptive test names** that explain what is being tested
- **Keep tests simple** and focused on one thing
- **Mock external dependencies** to isolate the code under test
- **Use Arrange-Act-Assert pattern** for clarity
- **Test behavior, not implementation** - don't test private methods
- **Keep tests fast** - unit tests should run in milliseconds
- **Write edge case tests** for error handling
- **Maintain test suite** - delete obsolete tests
- **Run tests frequently** - before every commit

### ❌ DON'T

- Don't write implementation first
- Don't write tests after the fact (defeats TDD benefits)
- Don't test too many things in one test
- Don't test private implementation details
- Don't mock when you should test real behavior
- Don't skip edge cases because they're unlikely
- Don't leave failing tests in your codebase
- Don't write tests that depend on other tests
- Don't test the testing framework itself
- Don't ignore test failures

## Test Coverage Requirements

### Minimum Standards

- **New code**: 80%+ coverage
- **Modified code**: Maintain or improve coverage
- **Critical paths**: 100% coverage
- **Error handling**: All catch blocks tested

### Coverage Tools

**.NET (OpenCover, coverlet)**
```bash
dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=opencover
dotnet reportgenerator -reports:"coverage.opencover.xml" -targetdir:"coverage" -reporttypes:Html
```

**Python (pytest-cov)**
```bash
pytest --cov=app --cov-report=html tests/
# Coverage report in htmlcov/index.html
```

**TypeScript/JavaScript (Jest)**
```bash
jest --coverage
# Coverage report in coverage/index.html
```

## Test Types in TDD

### 1. Unit Tests
**What**: Test single function/method in isolation
**How**: Mock all dependencies, test one behavior
**Speed**: Very fast (milliseconds)
**Example**: Test UserService.GetUserAsync with mocked repository

### 2. Integration Tests
**What**: Test multiple components working together
**How**: Use real or integration test database
**Speed**: Slower (seconds per test)
**Example**: Test UserService with real database connection

### 3. End-to-End Tests
**What**: Test complete user workflows
**How**: Use actual application and browser/HTTP
**Speed**: Slowest (10+ seconds per test)
**Example**: Test login flow from UI to database

### Pyramid Principle
```
         /\
        /E2E\              Few - Slow - Expensive
       /-----\
      / Integration\       Some - Moderate - Medium cost
     /-----------\
    / Unit Tests \        Many - Fast - Cheap
   /________________\
```

## Common TDD Patterns

### Test Doubles Pattern

**Stub**: Returns fixed data
```csharp
var mockRepository = new Mock<IUserRepository>();
mockRepository
    .Setup(r => r.GetByIdAsync(1))
    .ReturnsAsync(new User { Id = 1, Email = "test@example.com" });
```

**Spy**: Records calls made
```csharp
mockRepository.Verify(r => r.GetByIdAsync(1), Times.Once);
```

**Fake**: Working implementation for testing
```csharp
var fakeRepository = new FakeUserRepository(); // In-memory implementation
```

**Mock**: Verifies interactions
```csharp
mockRepository
    .Setup(r => r.CreateAsync(It.IsAny<User>()))
    .ReturnsAsync(new User { Id = 1 });
```

## Troubleshooting TDD

### Problem: "My tests are too slow"
**Solution**:
- Use unit tests instead of integration tests
- Mock external calls (database, APIs)
- Use in-memory databases for integration tests
- Parallelize test execution

### Problem: "My tests are flaky"
**Solution**:
- Remove dependencies on time/randomness
- Use TestFixture setup/teardown
- Avoid test interdependencies
- Use proper async/await handling

### Problem: "I can't test my code"
**Solution**:
- Refactor to use dependency injection
- Extract business logic from infrastructure
- Use interfaces for external dependencies
- Keep methods small and focused

### Problem: "Tests take too long to write"
**Solution**:
- Start with happy path tests
- Add edge cases incrementally
- Reuse test fixtures
- Use test factories/builders for complex objects

## Integration with CI/CD

```yaml
# GitHub Actions Example
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Run tests
        run: npm test -- --coverage

      - name: Check coverage
        run: npm run coverage:check

      - name: Upload coverage
        uses: codecov/codecov-action@v2
```

## Key Takeaways

1. **Write tests first** - Test before implementation
2. **Red → Green → Refactor** - Follow the cycle strictly
3. **Keep tests simple** - One assertion per test ideally
4. **Mock dependencies** - Test in isolation
5. **Maintain test suite** - Delete obsolete tests
6. **Coverage matters** - Aim for 80%+ on new code
7. **Tests as documentation** - Show how code should be used
8. **Refactor safely** - Tests catch regressions
9. **Speed up feedback** - Fast tests enable TDD flow
10. **Continuous improvement** - Refactor to better designs

