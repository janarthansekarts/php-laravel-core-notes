# Properties and Methods (Deep Dive)

> **Prerequisites:** [02 - Classes and Objects](02-classes-and-objects.md)
> **Next:** [04 - Visibility](04-visibility.md)

---

## The Problem / Why This Exists

In [file 02](02-classes-and-objects.md), we created classes with properties and methods — but we used them without fully understanding all the rules and options. We wrote `private string $name` without explaining what `string` does there, or why we write `private`. We used `: void` and `: string` on methods without explaining what those do.

This file covers **everything** about properties and methods — the syntax, rules, types, defaults, and patterns. Think of it as the **detailed manual** for the two things that live inside every class: **data (properties)** and **behavior (methods)**.

---

## What It Introduces

### Quick Refresher (from files 01-02)

Remember from [file 01](01-why-oop-exists.md):
- A **variable** inside a class is called a **property**
- A **function** inside a class is called a **method**
- Same concepts, different names based on where they live

This file goes deeper into:
- **Properties:** How to declare them, types, defaults, constants, readonly
- **Methods:** Parameters, return types, getters/setters, method chaining

---

## Real-World Analogy

Think of a **Bank Account**:

**Properties** = the data written on your account statement:
- Account number, balance, owner name, account type
- This data **describes the state** of the account at any moment

**Methods** = the actions you can perform at the bank:
- Deposit, withdraw, check balance, transfer
- These actions **read or change** the state (properties)

The properties and methods are **inseparable** — a `withdraw()` method makes no sense without a `$balance` property to modify. That's why they live together in one class.

---

## Part 1: Properties (Data Inside a Class)

### Basic Property Declaration

```php
class User
{
    // Declaring properties — these are the "data slots" every User object will have
    public string $name;
    public string $email;
    public int $age;
}
```

Each property declaration has up to 4 parts:

```
 visibility   type    name       default value
     │          │       │              │
     ▼          ▼       ▼              ▼
  private    string   $name    =    "Unknown";
```

Let's break each part down.

### Property Types (PHP 7.4+)

**Type declarations** (also called **type hints**) tell PHP what kind of data a property can hold. PHP will throw a `TypeError` if you try to store the wrong type.

```php
class Product
{
    public string $name;        // Can only hold text
    public int $quantity;       // Can only hold whole numbers
    public float $price;        // Can only hold decimal numbers
    public bool $isActive;      // Can only hold true or false
    public array $tags;         // Can only hold arrays
}
```

| Type | What It Accepts | Example Values |
|---|---|---|
| `string` | Text | `"Jegan"`, `"hello@mail.com"` |
| `int` | Whole numbers | `1`, `42`, `-5` |
| `float` | Decimal numbers | `3.14`, `99.99` |
| `bool` | True/false | `true`, `false` |
| `array` | Arrays | `['a', 'b']`, `[1, 2, 3]` |
| `object` | Any object | `new User()`, `new DateTime()` |
| `ClassName` | Object of that specific class | `User`, `DateTime`, `Request` |

> **Why use types?** Without types, PHP silently accepts any value — you could put a string in a price field and discover the bug 3 days later. Types catch mistakes **immediately** at runtime, just like `setStatus()` caught `"banana"` in [file 01](01-why-oop-exists.md).

### Nullable Types (`?`)

Sometimes a property **might** have no value. The `?` prefix makes a type **nullable** — it can hold its declared type OR `null`.

```php
class User
{
    public string $name;          // MUST have a string — null is NOT allowed
    public ?string $middleName;   // Can be a string OR null
    public ?int $age;             // Can be an int OR null
}

$user = new User();
$user->name = null;        // ❌ TypeError! string cannot be null
$user->middleName = null;  // ✅ This is fine — it's nullable
$user->middleName = "Kumar"; // ✅ Also fine — it's a string
```

> **When to use `?`:** Use nullable when a value is genuinely optional — e.g., middle name, phone number, deleted_at timestamp. Don't make everything nullable "just in case" — that defeats the purpose of type safety.

### Default Values

Properties can have **default values** — these are used when you create an object without explicitly setting that property.

```php
class User
{
    public string $name;
    public string $role = 'user';       // Default: every new user is a 'user'
    public bool $isActive = true;       // Default: every new user starts active
    public int $loginCount = 0;         // Default: starts at zero
    public array $permissions = [];     // Default: empty array
}

$user = new User();
echo $user->role;        // "user" — the default kicked in
echo $user->loginCount;  // 0
```

> **Rule:** You can only use **simple, constant values** as defaults (strings, numbers, booleans, arrays, `null`). You CANNOT use function calls or `new` as defaults in property declarations: `public DateTime $createdAt = new DateTime();` is a **syntax error**. Use the constructor for that.

### Untyped Properties (Legacy)

Before PHP 7.4, properties couldn't have type declarations:

```php
// Old PHP (before 7.4) — no types
class User
{
    public $name;   // Can hold ANY value — string, int, array, null, anything
    public $email;
}

$user = new User();
$user->name = "Jegan";   // Fine
$user->name = 42;        // Also fine — no type enforcement!
$user->name = ['array'];  // Also fine — PHP doesn't care
```

> **Don't do this in new code.** Always use typed properties. Untyped properties are only seen in legacy codebases.

### Class Constants (`const`)

Sometimes a value belongs to the class but **never changes**. Use `const` instead of a property:

```php
class User
{
    // Constants — NEVER change, shared across ALL objects
    public const MAX_LOGIN_ATTEMPTS = 5;
    public const ALLOWED_ROLES = ['admin', 'editor', 'user'];

    // Properties — CAN change, each object has its own copy
    private string $name;
    private int $loginAttempts = 0;
}

// Access constants with :: (not ->)
echo User::MAX_LOGIN_ATTEMPTS;  // 5

// You CANNOT change a constant
// User::MAX_LOGIN_ATTEMPTS = 10; // ❌ Fatal error!
```

| Feature | Property | Constant |
|---|---|---|
| **Can change?** | Yes, per object | No — fixed forever |
| **Each object gets its own?** | Yes | No — shared by all objects |
| **Accessed with** | `$object->property` (via `->`) | `ClassName::CONSTANT` (via `::`) |
| **Naming convention** | `$camelCase` | `UPPER_SNAKE_CASE` |
| **Use when** | Value varies per object | Value is fixed and universal |

> **The `::` operator:** You saw `->` in [file 02](02-classes-and-objects.md) for accessing object members. `::` is the **scope resolution operator** (also called **Paamayim Nekudotayim** — Hebrew for "double colon"). It accesses things that belong to the **class itself**, not to a specific object. We'll cover this deeply in [07-static-properties-and-methods](07-static-properties-and-methods.md).

### Readonly Properties (PHP 8.1+)

Sometimes a property should be set **once** (in the constructor) and never changed again:

```php
class User
{
    public readonly string $name;
    public readonly string $email;

    public function __construct(string $name, string $email)
    {
        $this->name = $name;      // ✅ First assignment — allowed
        $this->email = $email;
    }
}

$user = new User("Jegan", "jegan@example.com");
echo $user->name;           // ✅ "Jegan" — reading is fine
$user->name = "Changed";    // ❌ Error: Cannot modify readonly property!
```

> **When to use `readonly`:** Use it for properties that should be **immutable** after creation — like an ID, creation date, or user's email in some systems. It's a signal to other developers: "this should never change after the object is created."

---

## Part 2: Methods (Behavior Inside a Class)

### Basic Method Declaration

```php
class User
{
    private string $name;

    //  visibility  return type
    //      │           │
    //      ▼           ▼
    public function getDisplayName(): string
    //         │              │
    //      keyword      method name
    {
        return $this->name;
    }
}
```

A method declaration has:
1. **Visibility** (`public`, `private`, `protected`) — who can call it (covered in [file 04](04-visibility.md))
2. **`function` keyword** — tells PHP this is a method
3. **Name** — by convention uses `camelCase`: `getDisplayName`, `setEmail`, `calculateTotal`
4. **Parameters** — input values (in parentheses)
5. **Return type** — what the method gives back (after the `:`)
6. **Body** — the actual code (in curly braces)

### Parameters (Method Input)

Parameters let you pass data INTO a method:

```php
class Calculator
{
    // Simple parameters with type hints
    public function add(int $a, int $b): int
    {
        return $a + $b;
    }

    // Parameter with a default value
    public function multiply(int $a, int $b = 1): int
    {
        return $a * $b;
    }

    // Nullable parameter — can receive int OR null
    public function divide(int $a, ?int $b): float
    {
        if ($b === null || $b === 0) {
            throw new InvalidArgumentException("Cannot divide by zero or null");
        }
        return $a / $b;
    }
}

$calc = new Calculator();
echo $calc->add(5, 3);          // 8
echo $calc->multiply(5);        // 5 (default $b = 1)
echo $calc->multiply(5, 3);     // 15
echo $calc->divide(10, null);   // ❌ Exception!
```

> **Default parameters must come LAST.** `function add(int $a = 0, int $b)` is technically valid but practically useless — you can't skip `$a` and only pass `$b`. Always put required parameters first, defaults last.

### Return Types

The return type (after `:`) declares what a method gives back:

```php
class User
{
    private string $name;
    private ?string $bio = null;

    // Returns a string — MUST return a string
    public function getName(): string
    {
        return $this->name;
    }

    // Returns void — MUST NOT return anything
    public function setName(string $name): void
    {
        $this->name = $name;
        // No return statement (or just `return;` with no value)
    }

    // Returns nullable string — can return string OR null
    public function getBio(): ?string
    {
        return $this->bio;  // Might be null
    }

    // Returns self — returns the object itself (for method chaining)
    public function updateName(string $name): self
    {
        $this->name = $name;
        return $this;  // Returns the same object
    }

    // Returns bool
    public function hasBio(): bool
    {
        return $this->bio !== null;
    }
}
```

| Return Type | Meaning | When to Use |
|---|---|---|
| `: string` | Must return a string | Getters that return text |
| `: int` | Must return an integer | Counts, calculations |
| `: float` | Must return a decimal | Prices, percentages |
| `: bool` | Must return true/false | Checks: `isActive()`, `hasPermission()` |
| `: void` | Must NOT return a value | Setters, actions with no output |
| `: array` | Must return an array | Lists, collections |
| `: self` | Returns the same object | Method chaining (below) |
| `: ?string` | String or null | Optional data that might not exist |
| `: ClassName` | Object of that class | Factory methods, builders |

> **`void` explained:** When a method's job is to **do something** (set a value, send an email, log a message) rather than **return something**, its return type is `void`. Trying to `return $value` in a void method causes a fatal error. You can use `return;` (with no value) to exit early.

### The Getter/Setter Pattern

In files 01-02, we used getters (`getName()`) and setters (`setEmail()`). Here's **why this pattern exists**:

```php
// ❌ WITHOUT getters/setters — direct property access
class User
{
    public string $email;  // Anyone can set this to anything
}

$user = new User();
$user->email = "not-an-email";  // No validation! Bad data gets in.
$user->email = "";              // Empty email? No one stops it.
```

```php
// ✅ WITH getters/setters — controlled access
class User
{
    private string $email;  // Hidden — outside code can't touch it directly

    // GETTER — controlled way to READ the email
    public function getEmail(): string
    {
        return $this->email;
    }

    // SETTER — controlled way to WRITE the email (with validation)
    public function setEmail(string $email): void
    {
        $sanitized = filter_var(trim(strtolower($email)), FILTER_SANITIZE_EMAIL);

        if (!filter_var($sanitized, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException("Invalid email: $email");
        }

        $this->email = $sanitized;
    }
}
```

| Pattern | Purpose | Naming Convention |
|---|---|---|
| **Getter** | Read a property's value | `getPropertyName()` — e.g., `getName()`, `getEmail()` |
| **Setter** | Write/modify a property's value (with validation) | `setPropertyName()` — e.g., `setName()`, `setEmail()` |
| **Boolean getter** | Check a true/false condition | `isPropertyName()` or `hasPropertyName()` — e.g., `isActive()`, `hasPermission()` |

> **Why not just make properties public?** Because public properties have NO control — any code can set them to any value. Private properties + getters/setters give you a **controlled API**: you decide what values are valid, how data is stored, and what format it's returned in. This is **encapsulation** in practice.

### Constructor Promotion (PHP 8.0+)

Writing properties, then assigning them in the constructor, is repetitive:

```php
// ❌ VERBOSE — PHP 7 style (still works, but repetitive)
class User
{
    private string $name;
    private string $email;

    public function __construct(string $name, string $email)
    {
        $this->name = $name;
        $this->email = $email;
    }
}
```

PHP 8.0 introduced **constructor promotion** — declare and assign properties in one line:

```php
// ✅ CONCISE — PHP 8.0+ constructor promotion
class User
{
    public function __construct(
        private string $name,
        private string $email,
        private string $role = 'user',  // With default value
    ) {
        // Properties are automatically declared AND assigned
        // No need for $this->name = $name; — PHP does it for you
    }

    public function getName(): string
    {
        return $this->name;
    }
}

$user = new User("Jegan", "jegan@example.com");
echo $user->getName();  // "Jegan"
```

The visibility keyword (`private`, `public`, `protected`) in the constructor parameters tells PHP: **"this parameter is also a property — declare it and assign it automatically."**

> **You'll see this everywhere in modern Laravel** — Service classes, DTOs, Form Requests all use constructor promotion. It reduces boilerplate significantly.

### Method Chaining (Fluent Interface)

When a setter returns `$this` (the object itself), you can **chain** multiple method calls on one line:

```php
class QueryBuilder
{
    private string $table;
    private string $where = '';
    private string $orderBy = '';
    private int $limit = 0;

    public function table(string $table): self
    {
        $this->table = $table;
        return $this;  // Returns the same object — enables chaining
    }

    public function where(string $condition): self
    {
        $this->where = $condition;
        return $this;
    }

    public function orderBy(string $column): self
    {
        $this->orderBy = $column;
        return $this;
    }

    public function limit(int $limit): self
    {
        $this->limit = $limit;
        return $this;
    }

    public function toSQL(): string
    {
        $sql = "SELECT * FROM {$this->table}";
        if ($this->where) $sql .= " WHERE {$this->where}";
        if ($this->orderBy) $sql .= " ORDER BY {$this->orderBy}";
        if ($this->limit) $sql .= " LIMIT {$this->limit}";
        return $sql;
    }
}

// WITHOUT chaining — verbose
$query = new QueryBuilder();
$query->table('users');
$query->where('active = 1');
$query->orderBy('name');
$query->limit(10);
echo $query->toSQL();

// WITH chaining — clean and readable
$sql = (new QueryBuilder())
    ->table('users')
    ->where('active = 1')
    ->orderBy('name')
    ->limit(10)
    ->toSQL();
```

**How it works step by step:**

```
(new QueryBuilder())        → Returns a QueryBuilder object
    ->table('users')        → Sets table, returns $this (same object)
    ->where('active = 1')   → Sets where, returns $this (same object)
    ->orderBy('name')       → Sets orderBy, returns $this (same object)
    ->limit(10)             → Sets limit, returns $this (same object)
    ->toSQL();              → Returns the SQL string
```

> **The key:** Each method returns `$this` (return type `: self`), so the next `->` call operates on the same object. If any method returned `void`, the chain would break.

---

## How Laravel Relies on This Concept

### Eloquent Models — Properties as Database Columns

```php
// Laravel's Model class uses "magic" properties — you don't declare them
// but they map to database columns automatically
$user = new User();
$user->name = "Jegan";         // 'name' column in the users table
$user->email = "j@example.com"; // 'email' column
$user->save();                  // Method — saves to database

// Behind the scenes, Eloquent stores these in a protected $attributes array
// and uses __get() and __set() magic methods (covered in a later file)
```

### Query Builder — Method Chaining Everywhere

```php
// This is the EXACT method chaining pattern from our QueryBuilder example:
$users = DB::table('users')
    ->where('active', true)
    ->orderBy('name')
    ->limit(10)
    ->get();

// Eloquent also uses it:
$users = User::where('active', true)
    ->where('role', 'admin')
    ->orderBy('created_at', 'desc')
    ->get();
```

### Controller Methods — Parameters Come from Laravel

```php
class UserController extends Controller
{
    // Laravel automatically injects the Request object as a parameter
    // The type hint (Request $request) tells Laravel WHAT to inject
    public function update(Request $request, int $id): JsonResponse
    //                     ^^^^^^^^^^^^^^^^  ^^^^^^^^  ^^^^^^^^^^^^
    //                     parameter          param    return type
    {
        $user = User::findOrFail($id);
        $user->update($request->validated());

        return response()->json($user);  // Returns a JsonResponse object
    }
}
```

### Constructor Promotion in Laravel Services

```php
// Modern Laravel services use constructor promotion heavily:
class UserService
{
    public function __construct(
        private UserRepository $repository,
        private MailService $mailer,
        private LoggerInterface $logger,
    ) {}

    public function createUser(array $data): User
    {
        $user = $this->repository->create($data);
        $this->mailer->sendWelcome($user);
        $this->logger->info("User created: {$user->id}");

        return $user;
    }
}
// Laravel's Service Container automatically injects the dependencies
// because of the type hints — we'll cover this in file 15
```

---

## Practice Exercise

> Design a `Book` class for a Library System:
> - What are 2-3 **properties** it should hold? (Think: what data describes a book?)
> - What are 1-2 **methods** it should have? (Think: what actions can you perform on/with a book?)
> - Which properties should have default values?
> - Which should be `readonly`?
>
> Try writing it yourself before looking at the example below.

<details>
<summary>Click to see one possible solution</summary>

```php
class Book
{
    private bool $isCheckedOut = false;  // Default: books start on the shelf

    public function __construct(
        public readonly string $isbn,     // Never changes — readonly
        private string $title,
        private string $author,
        private int $pageCount,
    ) {}

    public function getTitle(): string
    {
        return $this->title;
    }

    public function isAvailable(): bool
    {
        return !$this->isCheckedOut;
    }

    public function checkout(): void
    {
        if ($this->isCheckedOut) {
            throw new RuntimeException("{$this->title} is already checked out");
        }
        $this->isCheckedOut = true;
    }

    public function returnBook(): void
    {
        $this->isCheckedOut = false;
    }
}
```

</details>

---

## Common Mistakes

### 1. Not Using Type Hints

```php
// ❌ No types — anything goes
class User
{
    public $name;
    public $age;
}

$user = new User();
$user->age = "twenty";  // PHP won't complain, but this is a bug

// ✅ Always declare types
class User
{
    public string $name;
    public int $age;
}

$user = new User();
$user->age = "twenty";  // ❌ TypeError! — caught immediately
```

### 2. Making All Properties Public

```php
// ❌ Public properties = no control
class BankAccount
{
    public float $balance = 0;
}

$account = new BankAccount();
$account->balance = -999999;  // Negative balance? No one stopped it.

// ✅ Private property + method with validation
class BankAccount
{
    private float $balance = 0;

    public function withdraw(float $amount): void
    {
        if ($amount <= 0) {
            throw new InvalidArgumentException("Amount must be positive");
        }
        if ($amount > $this->balance) {
            throw new RuntimeException("Insufficient funds");
        }
        $this->balance -= $amount;
    }
}
```

### 3. Forgetting Return Types

```php
// ❌ No return type — what does this give back? No one knows.
class User
{
    public function findByEmail($email)
    {
        // Returns... User? null? array? string? 🤷
    }
}

// ✅ Clear return type — everyone knows what to expect
class User
{
    public function findByEmail(string $email): ?self
    {
        // Returns a User object, or null if not found
    }
}
```

### 4. Using Constants Where Properties Should Be Used (and Vice Versa)

```php
// ❌ WRONG — this changes per object, should be a property
class User
{
    public const NAME = "Jegan";  // Every user named Jegan? No.
}

// ❌ WRONG — this never changes, should be a constant
class User
{
    public string $maxLoginAttempts = 5;  // Should this vary per user? No.
}

// ✅ RIGHT
class User
{
    public const MAX_LOGIN_ATTEMPTS = 5;  // Universal, never changes
    private string $name;                 // Different per user
}
```

---

## Key Takeaways

- **Properties** are variables inside a class — they hold the object's **state** (data)
- **Methods** are functions inside a class — they define the object's **behavior** (actions)
- Always use **type declarations** on properties (`string`, `int`, `bool`, `?string`, etc.) — they catch type errors immediately
- Use **nullable types** (`?string`) only when a value is genuinely optional
- **Default values** must be simple constants — use the constructor for complex defaults
- **`const`** is for values that never change and are shared across all objects; **properties** are for values that vary per object
- **`readonly`** (PHP 8.1+) = set once in the constructor, never changed again — use for IDs, dates, immutable data
- **Constructor promotion** (PHP 8.0+) = declare + assign properties directly in the constructor parameters — reduces boilerplate
- **Getters/setters** give you controlled access — private property + public getter/setter = encapsulation in practice
- **Method chaining** works by returning `$this` (return type `: self`) — used everywhere in Laravel's Query Builder and Eloquent
- Always declare **return types** on methods — `: string`, `: void`, `: bool`, `: ?self`, etc.
- **Parameters with defaults** must come last in the parameter list

---

> **Next up:** [04 - Visibility (public, protected, private)](04-visibility.md) — We've been using `private` and `public` without fully explaining them. Next, we'll cover all three visibility modifiers and when to use each one.
