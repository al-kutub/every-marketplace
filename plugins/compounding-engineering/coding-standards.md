# Coding Standards

Comprehensive coding standards for all tech stacks used in the compounding-engineering ecosystem. These standards ensure consistency, maintainability, and quality across .NET, Python, TypeScript, and frontend projects.

## Table of Contents

1. [.NET & ASP.NET Core Standards](#net--aspnet-core-standards)
2. [Python Standards](#python-standards)
3. [TypeScript Standards](#typescript-standards)
4. [Frontend Standards (React, Next.js, shadcn)](#frontend-standards-react-nextjs-shadcn)
5. [Database & Entity Framework Standards](#database--entity-framework-standards)
6. [API Design Standards](#api-design-standards)
7. [Testing Standards](#testing-standards)
8. [Git & Commit Standards](#git--commit-standards)

---

## .NET & ASP.NET Core Standards

### Naming Conventions

**Classes, Interfaces, Methods, Properties - PascalCase:**
```csharp
✅ CORRECT
public class UserAuthenticationService
{
    public UserProfile CurrentUser { get; set; }
    public async Task<bool> AuthenticateUserAsync(string email, string password)
    {
        // Implementation
    }
}

public interface IAuthenticationService
{
    Task<AuthenticationResult> AuthenticateAsync(Credentials credentials);
}

❌ WRONG
public class user_authentication_service
{
    public user_profile currentUser { get; set; }
    public async Task authenticate_user_async(string email, string password)
    {
        // Implementation
    }
}
```

**Private Fields - camelCase with underscore prefix:**
```csharp
✅ CORRECT
public class OrderProcessor
{
    private readonly IPaymentGateway _paymentGateway;
    private List<Order> _pendingOrders;

    public OrderProcessor(IPaymentGateway paymentGateway)
    {
        _paymentGateway = paymentGateway;
    }
}

❌ WRONG
public class OrderProcessor
{
    private readonly IPaymentGateway paymentGateway;
    private List<Order> PendingOrders;
}
```

**Constants - UPPER_SNAKE_CASE:**
```csharp
✅ CORRECT
public static class AppConstants
{
    public const int MAX_LOGIN_ATTEMPTS = 5;
    public const string DEFAULT_TIMEZONE = "UTC";
    public const decimal TAX_RATE = 0.08m;
}

❌ WRONG
public static class AppConstants
{
    public const int maxLoginAttempts = 5;
    public const string defaultTimezone = "UTC";
}
```

**Async Methods - Suffix with "Async":**
```csharp
✅ CORRECT
public async Task<User> GetUserAsync(int userId)
{
    return await _context.Users.FindAsync(userId);
}

public async Task SaveChangesAsync()
{
    await _context.SaveChangesAsync();
}

❌ WRONG
public async Task<User> GetUser(int userId)
{
    return await _context.Users.FindAsync(userId);
}

public Task UpdateUserAsync(User user) // Missing await usage
{
    return _context.SaveChangesAsync();
}
```

### Type Safety - Enums Over Strings

**CRITICAL: Always use enums for fixed value sets:**
```csharp
✅ CORRECT
public enum UserRoleEnum
{
    Admin = 0,
    Manager = 1,
    User = 2,
    Guest = 3
}

public enum OrderStatusEnum
{
    Pending = 0,
    Processing = 1,
    Shipped = 2,
    Delivered = 3,
    Cancelled = 4
}

public class User
{
    public int Id { get; set; }
    public string Email { get; set; }
    public UserRoleEnum Role { get; set; } = UserRoleEnum.User;
}

public class Order
{
    public int Id { get; set; }
    public OrderStatusEnum Status { get; set; } = OrderStatusEnum.Pending;
}

❌ WRONG
public class User
{
    public int Id { get; set; }
    public string Email { get; set; }
    public string Role { get; set; } = "User"; // DON'T USE STRING
}

public class Order
{
    public int Id { get; set; }
    public string Status { get; set; } = "pending"; // DON'T USE STRING
}
```

**Database Mapping for Enums:**
```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<User>()
        .Property(e => e.Role)
        .HasConversion<int>()
        .HasDefaultValue(UserRoleEnum.User);

    modelBuilder.Entity<Order>()
        .Property(e => e.Status)
        .HasConversion<int>()
        .HasDefaultValue(OrderStatusEnum.Pending);
}
```

### Nullable Reference Types

**Enable nullable reference types and handle nullability explicitly:**
```csharp
✅ CORRECT
#nullable enable

public class UserService
{
    private readonly IUserRepository _repository;

    public UserService(IUserRepository repository)
    {
        _repository = repository ?? throw new ArgumentNullException(nameof(repository));
    }

    public async Task<User?> GetUserByEmailAsync(string email)
    {
        if (string.IsNullOrWhiteSpace(email))
            return null;

        return await _repository.FindByEmailAsync(email);
    }

    public string GetUserDisplayName(User? user)
    {
        return user?.Name ?? "Anonymous";
    }
}

❌ WRONG
#nullable disable

public class UserService
{
    private IUserRepository _repository; // Can be null, no validation

    public User GetUserByEmail(string email)
    {
        if (email == null) return null; // Implicit null handling
        return _repository.FindByEmail(email);
    }
}
```

### Dependency Injection & Constructor Injection

**Always use constructor injection for dependencies:**
```csharp
✅ CORRECT
public class OrderService
{
    private readonly IPaymentGateway _paymentGateway;
    private readonly IEmailService _emailService;
    private readonly IOrderRepository _orderRepository;

    public OrderService(
        IPaymentGateway paymentGateway,
        IEmailService emailService,
        IOrderRepository orderRepository)
    {
        _paymentGateway = paymentGateway ?? throw new ArgumentNullException(nameof(paymentGateway));
        _emailService = emailService ?? throw new ArgumentNullException(nameof(emailService));
        _orderRepository = orderRepository ?? throw new ArgumentNullException(nameof(orderRepository));
    }

    public async Task<Order> ProcessOrderAsync(Order order)
    {
        var payment = await _paymentGateway.ProcessAsync(order);
        await _emailService.SendConfirmationAsync(order);
        return await _orderRepository.SaveAsync(order);
    }
}

// Startup/DI Registration
public void ConfigureServices(IServiceCollection services)
{
    services.AddScoped<IOrderService, OrderService>();
    services.AddScoped<IPaymentGateway, PaymentGateway>();
    services.AddScoped<IEmailService, EmailService>();
    services.AddScoped<IOrderRepository, OrderRepository>();
}

❌ WRONG
public class OrderService
{
    private IPaymentGateway _paymentGateway;

    public OrderService()
    {
        _paymentGateway = new PaymentGateway(); // Hard-coded dependency
    }

    public Order ProcessOrder(Order order)
    {
        // Can't test, can't mock, tightly coupled
    }
}
```

### Error Handling & Exception Management

**Use custom exceptions and proper exception handling:**
```csharp
✅ CORRECT
public class UserNotFoundException : Exception
{
    public int UserId { get; }

    public UserNotFoundException(int userId)
        : base($"User with ID {userId} not found.")
    {
        UserId = userId;
    }
}

public class InsufficientFundsException : Exception
{
    public decimal RequiredAmount { get; }
    public decimal AvailableAmount { get; }

    public InsufficientFundsException(decimal required, decimal available)
        : base($"Insufficient funds. Required: {required}, Available: {available}")
    {
        RequiredAmount = required;
        AvailableAmount = available;
    }
}

public class UserService
{
    public async Task<User> GetUserAsync(int userId)
    {
        var user = await _repository.FindByIdAsync(userId);

        if (user == null)
            throw new UserNotFoundException(userId);

        return user;
    }
}

// In API Controller
[HttpGet("{id}")]
public async Task<IActionResult> GetUser(int id)
{
    try
    {
        var user = await _userService.GetUserAsync(id);
        return Ok(user);
    }
    catch (UserNotFoundException ex)
    {
        return NotFound(new { message = ex.Message });
    }
    catch (Exception ex)
    {
        // Log exception
        return StatusCode(500, new { message = "An unexpected error occurred" });
    }
}

❌ WRONG
public User GetUser(int userId)
{
    try
    {
        return _repository.FindById(userId);
    }
    catch (Exception) // Generic catch-all
    {
        return null; // Silent failure
    }
}
```

### LINQ Usage

**Use LINQ expressively and efficiently:**
```csharp
✅ CORRECT
public async Task<List<OrderSummary>> GetActiveOrdersSummaryAsync()
{
    return await _context.Orders
        .Where(o => o.Status == OrderStatusEnum.Processing || o.Status == OrderStatusEnum.Pending)
        .OrderByDescending(o => o.CreatedAt)
        .Select(o => new OrderSummary
        {
            Id = o.Id,
            CustomerName = o.Customer.Name,
            Amount = o.TotalAmount,
            Status = o.Status.ToString()
        })
        .ToListAsync();
}

// Using extension methods
public async Task<List<Order>> GetRecentOrdersAsync(int days = 30)
{
    var cutoffDate = DateTime.UtcNow.AddDays(-days);

    return await _context.Orders
        .Where(o => o.CreatedAt >= cutoffDate)
        .Include(o => o.Customer)
        .Include(o => o.Items)
        .ToListAsync();
}

❌ WRONG
public List<OrderSummary> GetActiveOrders()
{
    var orders = _context.Orders.ToList(); // Loads ALL into memory
    var active = new List<OrderSummary>();

    foreach (var order in orders)
    {
        if (order.Status == "processing" || order.Status == "pending") // String comparison
        {
            active.Add(new OrderSummary
            {
                Id = order.Id,
                CustomerName = order.Customer.Name,
                Amount = order.TotalAmount,
                Status = order.Status
            });
        }
    }

    return active.OrderByDescending(o => o.CreatedAt).ToList();
}
```

### Method Size & Complexity

**Keep methods focused and simple (max 20 lines ideally):**
```csharp
✅ CORRECT
public async Task<PaymentResult> ProcessPaymentAsync(Order order)
{
    ValidateOrder(order);
    var transaction = CreateTransaction(order);
    var result = await _gateway.ProcessAsync(transaction);

    await LogPaymentAsync(order, result);

    return result;
}

private void ValidateOrder(Order order)
{
    if (order == null)
        throw new ArgumentNullException(nameof(order));

    if (order.Items.Count == 0)
        throw new InvalidOperationException("Order has no items");

    if (order.TotalAmount <= 0)
        throw new InvalidOperationException("Order amount must be positive");
}

private PaymentTransaction CreateTransaction(Order order)
{
    return new PaymentTransaction
    {
        OrderId = order.Id,
        Amount = order.TotalAmount,
        Currency = order.Currency,
        Description = $"Order #{order.Id}",
        Metadata = new { CustomerId = order.CustomerId }
    };
}

❌ WRONG
public async Task<PaymentResult> ProcessPayment(Order order)
{
    if (order == null) throw new ArgumentNullException(nameof(order));
    if (order.Items.Count == 0) throw new InvalidOperationException("Order has no items");
    if (order.TotalAmount <= 0) throw new InvalidOperationException("Order amount must be positive");

    var transaction = new PaymentTransaction
    {
        OrderId = order.Id,
        Amount = order.TotalAmount,
        Currency = order.Currency,
        Description = $"Order #{order.Id}",
        Metadata = new { CustomerId = order.CustomerId }
    };

    var result = await _gateway.ProcessAsync(transaction);

    await _logger.LogAsync(new PaymentLog
    {
        OrderId = order.Id,
        Amount = order.TotalAmount,
        Status = result.Success ? "Success" : "Failed",
        Message = result.Message,
        TransactionId = result.TransactionId,
        Timestamp = DateTime.UtcNow
    });

    return result;
}
```

---

## Python Standards

### Naming Conventions

**Classes - PascalCase:**
```python
✅ CORRECT
class UserAuthenticationService:
    def authenticate_user(self, email: str, password: str) -> bool:
        pass

class OrderProcessor:
    pass

❌ WRONG
class user_authentication_service:
    def authenticate_user(self, email: str, password: str) -> bool:
        pass

class orderProcessor:
    pass
```

**Functions & Methods - snake_case:**
```python
✅ CORRECT
def get_user_by_email(email: str) -> User:
    return User.objects.get(email=email)

def process_payment(order_id: int) -> PaymentResult:
    pass

❌ WRONG
def GetUserByEmail(email: str) -> User:
    pass

def processPayment(order_id: int) -> PaymentResult:
    pass
```

**Constants - UPPER_SNAKE_CASE:**
```python
✅ CORRECT
MAX_LOGIN_ATTEMPTS = 5
DEFAULT_TIMEZONE = "UTC"
API_TIMEOUT_SECONDS = 30

❌ WRONG
maxLoginAttempts = 5
defaultTimezone = "UTC"
```

**Private Methods/Attributes - Single underscore prefix:**
```python
✅ CORRECT
class UserService:
    def __init__(self, repository):
        self._repository = repository

    def get_user(self, user_id: int) -> Optional[User]:
        return self._repository.find_by_id(user_id)

    def _validate_email(self, email: str) -> bool:
        # Internal helper method
        return "@" in email

❌ WRONG
class UserService:
    def __init__(self, repository):
        self.repository = repository  # No underscore for private

    def getUser(self, user_id: int):  # camelCase in Python
        pass
```

### Type Hints

**Always use type hints for function parameters and return values:**
```python
✅ CORRECT
from typing import Optional, List, Dict, Any
from datetime import datetime

def get_user_by_id(user_id: int) -> Optional[User]:
    """Fetch user by ID, returns None if not found."""
    return User.objects.filter(id=user_id).first()

def create_order(customer_id: int, items: List[Dict[str, Any]]) -> Order:
    """Create a new order with validation."""
    order = Order(customer_id=customer_id)
    order.add_items(items)
    order.save()
    return order

async def process_payment_async(
    order_id: int,
    amount: float,
    currency: str = "USD"
) -> PaymentResult:
    """Process payment with optional currency parameter."""
    gateway = PaymentGateway()
    return await gateway.process(order_id, amount, currency)

❌ WRONG
def get_user_by_id(user_id):  # No type hints
    return User.objects.filter(id=user_id).first()

def create_order(customer_id, items):  # Missing type hints
    order = Order(customer_id=customer_id)
    order.add_items(items)
    return order
```

### Error Handling

**Use custom exceptions with meaningful messages:**
```python
✅ CORRECT
class UserNotFoundException(Exception):
    def __init__(self, user_id: int):
        self.user_id = user_id
        super().__init__(f"User with ID {user_id} not found.")

class InsufficientFundsException(Exception):
    def __init__(self, required: float, available: float):
        self.required = required
        self.available = available
        super().__init__(
            f"Insufficient funds. Required: {required}, Available: {available}"
        )

class UserService:
    def get_user(self, user_id: int) -> User:
        user = User.objects.filter(id=user_id).first()

        if not user:
            raise UserNotFoundException(user_id)

        return user

# In view/controller
try:
    user = user_service.get_user(user_id)
    return {"success": True, "user": user.to_dict()}
except UserNotFoundException as e:
    return {"success": False, "error": str(e)}, 404
except Exception as e:
    logger.error(f"Unexpected error: {e}")
    return {"success": False, "error": "Internal server error"}, 500

❌ WRONG
class UserService:
    def get_user(self, user_id: int):
        user = User.objects.filter(id=user_id).first()
        return user  # Silent None return

    def process_payment(self, amount):
        try:
            return gateway.process(amount)
        except:  # Generic catch-all
            return None
```

### Django-Specific Standards

**Models - Use descriptive field names and Meta class:**
```python
✅ CORRECT
from django.db import models
from django.core.validators import EmailValidator
from datetime import datetime

class User(models.Model):
    class UserRoleChoices(models.TextChoices):
        ADMIN = "admin", "Administrator"
        MANAGER = "manager", "Manager"
        USER = "user", "Regular User"
        GUEST = "guest", "Guest"

    email = models.EmailField(unique=True, validators=[EmailValidator()])
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)
    role = models.CharField(
        max_length=20,
        choices=UserRoleChoices.choices,
        default=UserRoleChoices.USER
    )
    is_active = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        db_table = "users"
        ordering = ["-created_at"]
        indexes = [
            models.Index(fields=["email"]),
            models.Index(fields=["role", "is_active"]),
        ]

    def __str__(self) -> str:
        return f"{self.first_name} {self.last_name} ({self.email})"

    @property
    def full_name(self) -> str:
        return f"{self.first_name} {self.last_name}"

class Order(models.Model):
    class OrderStatusChoices(models.IntegerChoices):
        PENDING = 1, "Pending"
        PROCESSING = 2, "Processing"
        SHIPPED = 3, "Shipped"
        DELIVERED = 4, "Delivered"
        CANCELLED = 5, "Cancelled"

    customer = models.ForeignKey(User, on_delete=models.CASCADE, related_name="orders")
    status = models.IntegerField(
        choices=OrderStatusChoices.choices,
        default=OrderStatusChoices.PENDING
    )
    total_amount = models.DecimalField(max_digits=10, decimal_places=2)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        db_table = "orders"
        ordering = ["-created_at"]

❌ WRONG
class User(models.Model):
    email = models.EmailField()
    name = models.CharField(max_length=100)
    role = models.CharField(max_length=20)  # String instead of choices
    active = models.BooleanField()  # Ambiguous field name
```

**Serializers (Django REST Framework):**
```python
✅ CORRECT
from rest_framework import serializers
from .models import User, Order

class UserSerializer(serializers.ModelSerializer):
    full_name = serializers.SerializerMethodField()

    class Meta:
        model = User
        fields = ["id", "email", "first_name", "last_name", "full_name", "role", "is_active"]
        read_only_fields = ["id", "full_name"]

    def get_full_name(self, obj: User) -> str:
        return obj.full_name

class OrderSerializer(serializers.ModelSerializer):
    customer = UserSerializer(read_only=True)

    class Meta:
        model = Order
        fields = ["id", "customer", "status", "total_amount", "created_at"]
        read_only_fields = ["id", "created_at"]

❌ WRONG
class UserSerializer(serializers.Serializer):
    # Manual field definitions without model connection
    id = serializers.IntegerField()
    email = serializers.EmailField()
    name = serializers.CharField(max_length=100)
```

**Views (Django Ninja):**
```python
✅ CORRECT
from ninja import Router, Query, Body
from typing import List, Optional
from .models import User
from .schemas import UserSchema, CreateUserSchema

router = Router()

@router.get("/users", response=List[UserSchema])
def list_users(
    request,
    skip: int = Query(0),
    limit: int = Query(10),
    role: Optional[str] = Query(None)
):
    """List all users with pagination and optional filtering."""
    query = User.objects.all()

    if role:
        query = query.filter(role=role)

    return query[skip:skip + limit]

@router.get("/users/{user_id}", response=UserSchema)
def get_user(request, user_id: int):
    """Get single user by ID."""
    try:
        return User.objects.get(id=user_id)
    except User.DoesNotExist:
        return {"error": "User not found"}, 404

@router.post("/users", response=UserSchema, status_code=201)
def create_user(request, payload: CreateUserSchema):
    """Create a new user."""
    user = User.objects.create(**payload.dict())
    return user

@router.put("/users/{user_id}", response=UserSchema)
def update_user(request, user_id: int, payload: CreateUserSchema):
    """Update user by ID."""
    try:
        user = User.objects.get(id=user_id)
        for attr, value in payload.dict(exclude_unset=True).items():
            setattr(user, attr, value)
        user.save()
        return user
    except User.DoesNotExist:
        return {"error": "User not found"}, 404

@router.delete("/users/{user_id}", status_code=204)
def delete_user(request, user_id: int):
    """Delete user by ID."""
    try:
        user = User.objects.get(id=user_id)
        user.delete()
    except User.DoesNotExist:
        return {"error": "User not found"}, 404

❌ WRONG
@router.get("/users")
def list_users(request):
    return User.objects.all()  # No pagination, no filtering

@router.post("/users")
def create_user(request):
    # No schema validation
    data = request.POST
    return User.objects.create(**data)
```

---

## TypeScript Standards

### Naming Conventions

**Interfaces & Types - PascalCase with prefix:**
```typescript
✅ CORRECT
interface IUser {
  id: number;
  email: string;
  firstName: string;
  lastName: string;
}

interface IAuthenticationService {
  authenticate(email: string, password: string): Promise<IUser | null>;
  logout(): void;
}

type UserRole = "admin" | "manager" | "user" | "guest";
type OrderStatus = "pending" | "processing" | "shipped" | "delivered" | "cancelled";

❌ WRONG
interface user {
  id: number;
  email: string;
}

type user_role = "admin" | "manager" | "user";
```

**Functions & Variables - camelCase:**
```typescript
✅ CORRECT
const getUserById = async (userId: number): Promise<IUser | null> => {
  return await userService.fetchUser(userId);
};

let isLoading = false;
const MAX_RETRIES = 3;

❌ WRONG
const GetUserById = async (userId: number) => {
  pass
};

let IsLoading = false;
```

**Classes - PascalCase:**
```typescript
✅ CORRECT
class UserAuthenticationService implements IAuthenticationService {
  constructor(private http: HttpClient) {}

  async authenticate(email: string, password: string): Promise<IUser | null> {
    return await this.http.post("/auth/login", { email, password });
  }
}

class OrderProcessor {
  private readonly _paymentGateway: IPaymentGateway;

  constructor(paymentGateway: IPaymentGateway) {
    this._paymentGateway = paymentGateway;
  }
}

❌ WRONG
class userAuthenticationService {
  authenticate(email, password) {
    // Implementation
  }
}
```

### Strict Type Safety

**Always enable strict mode and avoid `any`:**
```typescript
✅ CORRECT
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true
  }
}

// Usage
interface IUser {
  id: number;
  email: string;
  role: UserRole;
}

const processUser = (user: IUser | null, callback: (user: IUser) => void): void => {
  if (!user) {
    console.error("User is null");
    return;
  }

  callback(user);
};

const handleResponse = (response: IUser): void => {
  console.log(response.email);
};

processUser(userObject, handleResponse);

❌ WRONG
const processUser = (user: any, callback: any): any => {
  return callback(user);
};

interface IUser {
  id: any;
  email: any;
}
```

### Error Handling

**Use custom error classes and proper error handling:**
```typescript
✅ CORRECT
class AppError extends Error {
  constructor(
    public statusCode: number,
    public message: string,
    public details?: Record<string, any>
  ) {
    super(message);
    Object.setPrototypeOf(this, AppError.prototype);
  }
}

class UserNotFoundError extends AppError {
  constructor(userId: number) {
    super(404, `User with ID ${userId} not found`);
    Object.setPrototypeOf(this, UserNotFoundError.prototype);
  }
}

class ValidationError extends AppError {
  constructor(fieldErrors: Record<string, string[]>) {
    super(400, "Validation failed", fieldErrors);
    Object.setPrototypeOf(this, ValidationError.prototype);
  }
}

const getUserById = async (userId: number): Promise<IUser> => {
  const user = await userService.fetchUser(userId);

  if (!user) {
    throw new UserNotFoundError(userId);
  }

  return user;
};

// In component or controller
try {
  const user = await getUserById(123);
  console.log(user);
} catch (error) {
  if (error instanceof UserNotFoundError) {
    console.error(`User not found: ${error.message}`);
  } else if (error instanceof ValidationError) {
    console.error("Validation failed:", error.details);
  } else {
    console.error("Unexpected error:", error);
  }
}

❌ WRONG
const getUserById = (userId: number) => {
  return userService.fetchUser(userId) || null; // Silent null return
};

try {
  const user = getUserById(123);
} catch (error) {
  console.error(error); // Generic error handling
}
```

### Async/Await Best Practices

**Use async/await with proper error handling:**
```typescript
✅ CORRECT
const processOrder = async (orderId: number): Promise<IOrderResult> => {
  try {
    const order = await fetchOrder(orderId);
    const payment = await processPayment(order);
    const confirmation = await sendConfirmation(order, payment);

    return {
      success: true,
      orderId: order.id,
      paymentId: payment.id
    };
  } catch (error) {
    if (error instanceof PaymentError) {
      throw new OrderProcessingError("Payment failed", error);
    }
    throw error;
  }
};

// Parallel requests
const fetchUserAndOrders = async (userId: number) => {
  const [user, orders] = await Promise.all([
    fetchUser(userId),
    fetchUserOrders(userId)
  ]);

  return { user, orders };
};

❌ WRONG
const processOrder = (orderId: number) => {
  const order = fetchOrder(orderId);  // Missing await
  const payment = processPayment(order);  // Missing await
  return { orderId };
};

const fetchUserAndOrders = async (userId: number) => {
  const user = await fetchUser(userId);
  const orders = await fetchUserOrders(userId);  // Sequential, not parallel
  return { user, orders };
};
```

---

## Frontend Standards (React, Next.js, shadcn)

### Component Naming & Structure

**Use PascalCase for components, directory structure follows component name:**
```typescript
✅ CORRECT
// src/components/UserProfile/UserProfile.tsx
export const UserProfile: React.FC<IUserProfileProps> = ({
  userId,
  onUpdate
}) => {
  return (
    <div className="user-profile">
      {/* Component JSX */}
    </div>
  );
};

// src/components/UserProfile/UserProfile.test.tsx
describe("UserProfile", () => {
  it("should render user profile", () => {
    // Test implementation
  });
});

// src/components/UserProfile/index.ts
export { UserProfile } from "./UserProfile";

// src/hooks/useUserProfile.ts
export const useUserProfile = (userId: number) => {
  // Custom hook implementation
};

❌ WRONG
// src/components/userProfile.tsx
export const userProfile = (props) => {
  return <div>...</div>;
};

// Flat structure without organization
// src/components/UserProfile.tsx
// src/components/userProfileTest.tsx
```

### React Component Patterns

**Use functional components with hooks and proper TypeScript typing:**
```typescript
✅ CORRECT
interface IUserProfileProps {
  userId: number;
  onUpdate?: (user: IUser) => void;
}

interface IUserState {
  user: IUser | null;
  isLoading: boolean;
  error: AppError | null;
}

export const UserProfile: React.FC<IUserProfileProps> = ({
  userId,
  onUpdate
}) => {
  const [state, setState] = useState<IUserState>({
    user: null,
    isLoading: true,
    error: null
  });

  useEffect(() => {
    const fetchUser = async () => {
      try {
        setState(prev => ({ ...prev, isLoading: true }));
        const user = await userService.getUser(userId);
        setState({
          user,
          isLoading: false,
          error: null
        });
      } catch (error) {
        setState({
          user: null,
          isLoading: false,
          error: error instanceof AppError ? error : new AppError(500, "Failed to fetch user")
        });
      }
    };

    fetchUser();
  }, [userId]);

  if (state.isLoading) {
    return <LoadingSpinner />;
  }

  if (state.error) {
    return <ErrorMessage error={state.error} />;
  }

  if (!state.user) {
    return <NotFound />;
  }

  return (
    <div className="user-profile">
      <h1>{state.user.firstName} {state.user.lastName}</h1>
      <p>{state.user.email}</p>
      <button onClick={() => onUpdate?.(state.user)}>Update</button>
    </div>
  );
};

❌ WRONG
export const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    setLoading(true);
    fetch(`/api/users/${userId}`)
      .then(r => r.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err);
        setLoading(false);
      });
  }, [userId]);

  return (
    <div>
      {loading && <p>Loading...</p>}
      {error && <p>Error: {error}</p>}
      {user && <div>{user.firstName} {user.lastName}</div>}
    </div>
  );
};
```

### Custom Hooks Pattern

**Extract stateful logic into custom hooks:**
```typescript
✅ CORRECT
interface IUseUserResult {
  user: IUser | null;
  isLoading: boolean;
  error: AppError | null;
  refetch: () => Promise<void>;
}

export const useUser = (userId: number): IUseUserResult => {
  const [user, setUser] = useState<IUser | null>(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<AppError | null>(null);

  const fetchUser = useCallback(async () => {
    try {
      setIsLoading(true);
      setError(null);
      const data = await userService.getUser(userId);
      setUser(data);
    } catch (err) {
      setError(err instanceof AppError ? err : new AppError(500, "Failed to fetch user"));
    } finally {
      setIsLoading(false);
    }
  }, [userId]);

  useEffect(() => {
    fetchUser();
  }, [fetchUser]);

  return { user, isLoading, error, refetch: fetchUser };
};

// Usage in component
export const UserProfile: React.FC<{ userId: number }> = ({ userId }) => {
  const { user, isLoading, error, refetch } = useUser(userId);

  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} onRetry={refetch} />;
  if (!user) return <NotFound />;

  return <div>{user.firstName} {user.lastName}</div>;
};

❌ WRONG
export const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  // All logic mixed in component
  useEffect(() => {
    fetch(`/api/users/${userId}`).then(r => r.json()).then(setUser);
  }, [userId]);

  return <div>{user?.firstName}</div>;
};
```

### Form Handling with React Hook Form

**Use React Hook Form for efficient form management:**
```typescript
✅ CORRECT
interface IUserFormData {
  email: string;
  firstName: string;
  lastName: string;
  role: UserRole;
}

export const UserForm: React.FC<{ onSubmit: (data: IUserFormData) => Promise<void> }> = ({
  onSubmit
}) => {
  const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm<IUserFormData>({
    defaultValues: {
      email: "",
      firstName: "",
      lastName: "",
      role: "user"
    }
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register("email", {
          required: "Email is required",
          pattern: {
            value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
            message: "Invalid email address"
          }
        })}
        placeholder="Email"
      />
      {errors.email && <span>{errors.email.message}</span>}

      <input
        {...register("firstName", {
          required: "First name is required",
          minLength: { value: 2, message: "Minimum 2 characters" }
        })}
        placeholder="First Name"
      />
      {errors.firstName && <span>{errors.firstName.message}</span>}

      <select
        {...register("role", { required: "Role is required" })}
      >
        <option value="user">User</option>
        <option value="admin">Admin</option>
      </select>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? "Submitting..." : "Submit"}
      </button>
    </form>
  );
};

❌ WRONG
export const UserForm = ({ onSubmit }) => {
  const [email, setEmail] = useState("");
  const [firstName, setFirstName] = useState("");
  // Multiple state for each field

  return (
    <form onSubmit={e => {
      e.preventDefault();
      onSubmit({ email, firstName }); // No validation
    }}>
      <input value={email} onChange={e => setEmail(e.target.value)} />
      <input value={firstName} onChange={e => setFirstName(e.target.value)} />
      <button>Submit</button>
    </form>
  );
};
```

### State Management with Zustand

**Use Zustand for global state management:**
```typescript
✅ CORRECT
interface IAppStore {
  user: IUser | null;
  isAuthenticated: boolean;
  setUser: (user: IUser | null) => void;
  logout: () => void;
  authenticate: (email: string, password: string) => Promise<void>;
}

export const useAppStore = create<IAppStore>((set) => ({
  user: null,
  isAuthenticated: false,

  setUser: (user) => set({ user, isAuthenticated: !!user }),

  logout: () => set({ user: null, isAuthenticated: false }),

  authenticate: async (email, password) => {
    try {
      const user = await authService.authenticate(email, password);
      set({ user, isAuthenticated: true });
    } catch (error) {
      set({ user: null, isAuthenticated: false });
      throw error;
    }
  }
}));

// Usage in component
export const Header: React.FC = () => {
  const { user, isAuthenticated, logout } = useAppStore();

  if (!isAuthenticated) {
    return <LoginLink />;
  }

  return (
    <header>
      <span>Hello, {user?.firstName}</span>
      <button onClick={logout}>Logout</button>
    </header>
  );
};

❌ WRONG
// Redux, Redux Toolkit, or other overly complex solutions for simple state
// Multiple context providers nested
// Prop drilling
```

### shadcn/ui Component Usage

**Use shadcn components for consistent UI:**
```typescript
✅ CORRECT
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";

export const UserCard: React.FC<{ user: IUser }> = ({ user }) => {
  return (
    <Card>
      <CardHeader>
        <CardTitle>{user.firstName} {user.lastName}</CardTitle>
      </CardHeader>
      <CardContent>
        <p>{user.email}</p>
        <Button variant="outline">Edit Profile</Button>
      </CardContent>
    </Card>
  );
};

export const UserFormDialog: React.FC = () => {
  const [open, setOpen] = useState(false);

  return (
    <Dialog open={open} onOpenChange={setOpen}>
      <DialogTrigger asChild>
        <Button>Add User</Button>
      </DialogTrigger>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>Create New User</DialogTitle>
        </DialogHeader>
        <UserForm onSubmit={async (data) => {
          await userService.create(data);
          setOpen(false);
        }} />
      </DialogContent>
    </Dialog>
  );
};

❌ WRONG
// Custom button, input, select without consistency
// Inline styles instead of Tailwind
// No component composition
```

### URL & Routing

**Use Next.js routing with type-safe URLs:**
```typescript
✅ CORRECT
// src/app/users/[userId]/page.tsx
interface IUserPageProps {
  params: {
    userId: string;
  };
}

export default async function UserPage({ params }: IUserPageProps) {
  const user = await userService.getUser(parseInt(params.userId));

  if (!user) {
    notFound();
  }

  return <UserProfile user={user} />;
}

// src/app/users/[userId]/edit/page.tsx
export default function UserEditPage({ params }: IUserPageProps) {
  return <UserEditForm userId={parseInt(params.userId)} />;
}

// Use Link with type-safe URLs
import Link from "next/link";

export const UserList: React.FC<{ users: IUser[] }> = ({ users }) => {
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          <Link href={`/users/${user.id}`}>
            {user.firstName} {user.lastName}
          </Link>
        </li>
      ))}
    </ul>
  );
};

❌ WRONG
<Link href={"/users/" + user.id}>  // String concatenation
export default function UserPage(props) {
  // No type safety for params
  const userId = props.params.userId;
}
```

---

## Database & Entity Framework Standards

### DbContext Configuration

**Properly configure DbContext with fluent API:**
```csharp
✅ CORRECT
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    public DbSet<User> Users { get; set; }
    public DbSet<Order> Orders { get; set; }
    public DbSet<OrderItem> OrderItems { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // User configuration
        modelBuilder.Entity<User>(entity =>
        {
            entity.ToTable("users");
            entity.HasKey(e => e.Id);

            entity.Property(e => e.Email)
                .IsRequired()
                .HasMaxLength(255);

            entity.Property(e => e.Role)
                .HasConversion<int>()
                .HasDefaultValue(UserRoleEnum.User);

            entity.HasIndex(e => e.Email)
                .IsUnique();

            entity.HasMany(e => e.Orders)
                .WithOne(o => o.Customer)
                .HasForeignKey(o => o.CustomerId)
                .OnDelete(DeleteBehavior.Cascade);
        });

        // Order configuration
        modelBuilder.Entity<Order>(entity =>
        {
            entity.ToTable("orders");
            entity.HasKey(e => e.Id);

            entity.Property(e => e.Status)
                .HasConversion<int>()
                .HasDefaultValue(OrderStatusEnum.Pending);

            entity.Property(e => e.TotalAmount)
                .HasPrecision(10, 2);

            entity.HasMany(e => e.Items)
                .WithOne(i => i.Order)
                .HasForeignKey(i => i.OrderId)
                .OnDelete(DeleteBehavior.Cascade);
        });
    }
}

❌ WRONG
public class ApplicationDbContext : DbContext
{
    public DbSet<User> Users { get; set; }
    public DbSet<Order> Orders { get; set; }

    // No explicit configuration
    // Uses default conventions which may not be optimal
}
```

### Repository Pattern

**Use repository pattern for data access:**
```csharp
✅ CORRECT
public interface IUserRepository
{
    Task<User?> GetByIdAsync(int userId);
    Task<User?> GetByEmailAsync(string email);
    Task<List<User>> GetAllAsync(int skip = 0, int take = 10);
    Task<User> CreateAsync(User user);
    Task<User> UpdateAsync(User user);
    Task DeleteAsync(int userId);
}

public class UserRepository : IUserRepository
{
    private readonly ApplicationDbContext _context;

    public UserRepository(ApplicationDbContext context)
    {
        _context = context ?? throw new ArgumentNullException(nameof(context));
    }

    public async Task<User?> GetByIdAsync(int userId)
    {
        return await _context.Users
            .AsNoTracking()
            .FirstOrDefaultAsync(u => u.Id == userId);
    }

    public async Task<User?> GetByEmailAsync(string email)
    {
        if (string.IsNullOrWhiteSpace(email))
            return null;

        return await _context.Users
            .AsNoTracking()
            .FirstOrDefaultAsync(u => u.Email == email);
    }

    public async Task<List<User>> GetAllAsync(int skip = 0, int take = 10)
    {
        return await _context.Users
            .AsNoTracking()
            .OrderByDescending(u => u.CreatedAt)
            .Skip(skip)
            .Take(take)
            .ToListAsync();
    }

    public async Task<User> CreateAsync(User user)
    {
        _context.Users.Add(user);
        await _context.SaveChangesAsync();
        return user;
    }

    public async Task<User> UpdateAsync(User user)
    {
        _context.Users.Update(user);
        await _context.SaveChangesAsync();
        return user;
    }

    public async Task DeleteAsync(int userId)
    {
        var user = await _context.Users.FindAsync(userId);
        if (user != null)
        {
            _context.Users.Remove(user);
            await _context.SaveChangesAsync();
        }
    }
}

// DI Configuration
services.AddScoped<IUserRepository, UserRepository>();

❌ WRONG
// Direct DbContext usage in services
public class UserService
{
    public User GetUser(int userId)
    {
        return _context.Users.FirstOrDefault(u => u.Id == userId);
    }

    public void SaveUser(User user)
    {
        _context.Users.Add(user);
        _context.SaveChanges();
    }
}
```

### Migrations

**Use migrations for database version control:**
```bash
# Create migration
dotnet ef migrations add AddUserTable

# Apply migrations
dotnet ef database update

# Revert migration
dotnet ef migrations remove
```

**Forward-only migrations (production-safe):**
```csharp
✅ CORRECT
public partial class AddUserTable : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "users",
            columns: table => new
            {
                id = table.Column<int>(type: "int", nullable: false)
                    .Annotation("SqlServer:Identity", "1, 1"),
                email = table.Column<string>(type: "nvarchar(255)", nullable: false),
                role = table.Column<int>(type: "int", nullable: false, defaultValue: 2),
                created_at = table.Column<DateTime>(type: "datetime2", nullable: false),
                updated_at = table.Column<DateTime>(type: "datetime2", nullable: false)
            },
            constraints: table =>
            {
                table.PrimaryKey("pk_users", x => x.id);
            });

        migrationBuilder.CreateIndex(
            name: "ix_users_email",
            table: "users",
            column: "email",
            unique: true);
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        // Never remove data, only schema
        migrationBuilder.DropTable(name: "users");
    }
}
```

---

## API Design Standards

### RESTful Endpoints

**Use RESTful conventions with proper HTTP methods:**
```
✅ CORRECT
GET    /api/users                    - List all users
GET    /api/users/{userId}           - Get single user
POST   /api/users                    - Create user
PUT    /api/users/{userId}           - Update user
DELETE /api/users/{userId}           - Delete user

GET    /api/users/{userId}/orders    - List user's orders
POST   /api/orders                   - Create order
GET    /api/orders/{orderId}         - Get order details
PUT    /api/orders/{orderId}         - Update order
DELETE /api/orders/{orderId}         - Delete order

❌ WRONG
GET    /api/getUsers
GET    /api/getUserById?id=123
POST   /api/createUser
POST   /api/updateUser
GET    /api/deleteUser?id=123
```

### Response Formats

**Use consistent response structure:**
```csharp
✅ CORRECT
// Success response
{
  "success": true,
  "data": {
    "id": 1,
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Doe"
  },
  "message": "User retrieved successfully"
}

// Error response
{
  "success": false,
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User with ID 999 not found",
    "details": {
      "userId": 999
    }
  }
}

// List response with pagination
{
  "success": true,
  "data": [
    { "id": 1, "email": "user1@example.com" },
    { "id": 2, "email": "user2@example.com" }
  ],
  "pagination": {
    "skip": 0,
    "take": 10,
    "total": 2,
    "hasMore": false
  }
}

❌ WRONG
// Inconsistent structure
{
  "user": { ... },
  "status": "ok"
}

{
  "error": "Something went wrong"
}

{
  "items": [...],
  "count": 10
}
```

### API Controller Pattern

**Well-structured API controllers with proper validation:**
```csharp
✅ CORRECT
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;
    private readonly ILogger<UsersController> _logger;

    public UsersController(
        IUserService userService,
        ILogger<UsersController> logger)
    {
        _userService = userService ?? throw new ArgumentNullException(nameof(userService));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }

    [HttpGet("{userId}")]
    [ProducesResponseType(typeof(ApiResponse<UserDto>), StatusCodes.Status200OK)]
    [ProducesResponseType(typeof(ApiResponse<object>), StatusCodes.Status404NotFound)]
    public async Task<IActionResult> GetUser(int userId)
    {
        try
        {
            var user = await _userService.GetUserAsync(userId);

            return Ok(new ApiResponse<UserDto>
            {
                Success = true,
                Data = user,
                Message = "User retrieved successfully"
            });
        }
        catch (UserNotFoundException ex)
        {
            _logger.LogWarning(ex, "User not found: {UserId}", userId);

            return NotFound(new ApiResponse<object>
            {
                Success = false,
                Error = new ErrorDetails
                {
                    Code = "USER_NOT_FOUND",
                    Message = ex.Message
                }
            });
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Unexpected error retrieving user: {UserId}", userId);

            return StatusCode(500, new ApiResponse<object>
            {
                Success = false,
                Error = new ErrorDetails
                {
                    Code = "INTERNAL_SERVER_ERROR",
                    Message = "An unexpected error occurred"
                }
            });
        }
    }

    [HttpPost]
    [ProducesResponseType(typeof(ApiResponse<UserDto>), StatusCodes.Status201Created)]
    [ProducesResponseType(typeof(ApiResponse<object>), StatusCodes.Status400BadRequest)]
    public async Task<IActionResult> CreateUser([FromBody] CreateUserRequest request)
    {
        try
        {
            if (!ModelState.IsValid)
            {
                return BadRequest(new ApiResponse<object>
                {
                    Success = false,
                    Error = new ErrorDetails
                    {
                        Code = "VALIDATION_ERROR",
                        Message = "Invalid request data",
                        Details = ModelState
                    }
                });
            }

            var user = await _userService.CreateUserAsync(request);

            return CreatedAtAction(
                nameof(GetUser),
                new { userId = user.Id },
                new ApiResponse<UserDto>
                {
                    Success = true,
                    Data = user,
                    Message = "User created successfully"
                });
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error creating user");

            return StatusCode(500, new ApiResponse<object>
            {
                Success = false,
                Error = new ErrorDetails
                {
                    Code = "CREATE_FAILED",
                    Message = "Failed to create user"
                }
            });
        }
    }
}

❌ WRONG
[ApiController]
[Route("api/users")]
public class UsersController : ControllerBase
{
    [HttpGet("{id}")]
    public User GetUser(int id)
    {
        try
        {
            return _service.GetUser(id);
        }
        catch
        {
            return null; // No error handling
        }
    }

    [HttpPost]
    public void CreateUser(User user) // No return value, no response structure
    {
        _service.SaveUser(user);
    }
}
```

---

## Testing Standards

### Unit Tests

**Write comprehensive unit tests with proper assertions:**
```csharp
✅ CORRECT
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
        // Arrange
        int invalidUserId = 999;
        _userRepositoryMock
            .Setup(r => r.GetByIdAsync(invalidUserId))
            .ReturnsAsync((User)null);

        // Act
        await _userService.GetUserAsync(invalidUserId);

        // Assert - exception expected
    }

    [TestMethod]
    public async Task GetUserAsync_WithValidId_ReturnsUser()
    {
        // Arrange
        var expectedUser = new User { Id = 1, Email = "test@example.com", Role = UserRoleEnum.User };
        _userRepositoryMock
            .Setup(r => r.GetByIdAsync(1))
            .ReturnsAsync(expectedUser);

        // Act
        var result = await _userService.GetUserAsync(1);

        // Assert
        Assert.IsNotNull(result);
        Assert.AreEqual(expectedUser.Email, result.Email);
        Assert.AreEqual(expectedUser.Role, result.Role);
        _userRepositoryMock.Verify(r => r.GetByIdAsync(1), Times.Once);
    }
}

❌ WRONG
[TestClass]
public class UserServiceTests
{
    [TestMethod]
    public void TestGetUser()
    {
        var service = new UserService();
        var user = service.GetUser(1);
        Assert.IsNotNull(user); // Vague assertion
    }
}
```

### React Component Tests

**Write tests for React components using React Testing Library:**
```typescript
✅ CORRECT
import { render, screen, waitFor, fireEvent } from "@testing-library/react";
import { UserProfile } from "./UserProfile";
import * as userService from "@/services/userService";

jest.mock("@/services/userService");

describe("UserProfile", () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it("should display loading state initially", () => {
    (userService.getUser as jest.Mock).mockImplementation(
      () => new Promise(resolve => setTimeout(resolve, 1000))
    );

    render(<UserProfile userId={1} />);

    expect(screen.getByTestId("loading-spinner")).toBeInTheDocument();
  });

  it("should display user data when loaded", async () => {
    const mockUser = {
      id: 1,
      email: "test@example.com",
      firstName: "John",
      lastName: "Doe"
    };

    (userService.getUser as jest.Mock).mockResolvedValue(mockUser);

    render(<UserProfile userId={1} />);

    await waitFor(() => {
      expect(screen.getByText("John Doe")).toBeInTheDocument();
      expect(screen.getByText("test@example.com")).toBeInTheDocument();
    });
  });

  it("should display error message when fetch fails", async () => {
    (userService.getUser as jest.Mock).mockRejectedValue(
      new Error("Failed to fetch user")
    );

    render(<UserProfile userId={1} />);

    await waitFor(() => {
      expect(screen.getByTestId("error-message")).toBeInTheDocument();
      expect(screen.getByText(/failed to fetch user/i)).toBeInTheDocument();
    });
  });

  it("should call onUpdate when update button clicked", async () => {
    const mockUser = { id: 1, email: "test@example.com", firstName: "John", lastName: "Doe" };
    const onUpdateMock = jest.fn();

    (userService.getUser as jest.Mock).mockResolvedValue(mockUser);

    render(<UserProfile userId={1} onUpdate={onUpdateMock} />);

    await waitFor(() => {
      expect(screen.getByText("John Doe")).toBeInTheDocument();
    });

    fireEvent.click(screen.getByRole("button", { name: /update/i }));

    expect(onUpdateMock).toHaveBeenCalledWith(mockUser);
  });
});

❌ WRONG
describe("UserProfile", () => {
  it("should work", () => {
    const { container } = render(<UserProfile userId={1} />);
    expect(container).toBeTruthy();
  });
});
```

---

## Git & Commit Standards

### Commit Message Format

**Use clear, semantic commit messages:**
```
✅ CORRECT
feat: Add user authentication service
test: Add tests for UserService.GetUserAsync
fix: Handle null reference in payment gateway
refactor: Extract validation logic to helper methods
docs: Update API documentation for user endpoints
style: Format code according to standards
chore: Update dependencies

feat(auth): Implement JWT token generation
fix(orders): Resolve order status update race condition
refactor(database): Optimize user query performance

❌ WRONG
update code
fix bug
wip
asdf
test
changed stuff
final version
final version 2
```

### Branch Naming

**Use descriptive branch names with prefixes:**
```
✅ CORRECT
feature/user-authentication
feature/payment-gateway-integration
bugfix/order-status-update
refactor/database-optimization
docs/api-documentation

❌ WRONG
master
main-fix
test
new-stuff
dev
temp
```

---

## Summary of Key Standards

| Language | Key Standards |
|----------|---|
| **.NET** | PascalCase classes, camelCase fields, enum over strings, async/await, DI, nullable reference types |
| **Python** | snake_case functions, type hints, Django models with choices, proper exception handling |
| **TypeScript** | PascalCase interfaces, strict mode enabled, custom error classes, async/await |
| **React** | Functional components, custom hooks, React Hook Form, Zustand for state, shadcn/ui |
| **Database** | Repository pattern, fluent API configuration, forward-only migrations |
| **API** | RESTful endpoints, consistent response structure, proper status codes, error handling |
| **Testing** | Comprehensive unit tests, mocking, React Testing Library for components |
| **Git** | Semantic commit messages, descriptive branch names |

