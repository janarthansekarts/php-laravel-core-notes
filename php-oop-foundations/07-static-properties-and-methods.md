# Static Properties and Methods

> **Prerequisites:** [06-this-keyword.md](06-this-keyword.md)
> **Next:** [08-inheritance.md](08-inheritance.md)

---

## The Problem / Why This Exists

So far, every property and method we've written belongs to an **instance** — a specific object created with `new`. When you write `$myBook->title`, you're asking *that particular* book for *its* title. Every object carries its own copy of instance properties.

But what if you need data that belongs to the **class itself**, not to any one object?

Think about these real situations:
- **Counting how many objects have been created** — no single object "owns" that number
- **A configuration value** that's the same regardless of which object you ask — like an app version string
- **A utility function** that doesn't need any object state — like converting Celsius to Fahrenheit

You can't store "total cars produced" inside one specific car. That number belongs to the *concept* of "Car" — the **class blueprint**, not any instance.

This is exactly what the **`static`** keyword solves.

---

## What It Introduces

| Term | Meaning |
|------|---------|
| **`static` keyword** | Marks a property or method as belonging to the **class**, not to any instance |
| **`self::`** | Refers to the class where the code is **written** (compile-time binding) |
| **`static::`** | Refers to the class that was **actually called** at runtime (Late Static Binding) |
| **`::`** | The **Scope Resolution Operator** (also called **Paamayim Nekudotayim** — Hebrew for "double colon") |
| **Late Static Binding** | PHP's ability to resolve `static::` to the class that was *actually called*, not where the code was *written* |

### The `::` Operator — A Closer Look

You first saw `::` in [03-properties-and-methods.md](03-properties-and-methods.md) for accessing constants (`User::MAX_LOGIN_ATTEMPTS`). The `::` operator is used whenever you need to access something that belongs to the **class itself** rather than an instance:

```
┌────────────────────────────────────────────────────────┐
│  ->  means "ask THIS OBJECT for its thing"             │
│      $user->name       (instance property)             │
│      $user->getName()  (instance method)               │
│                                                        │
│  ::  means "ask THE CLASS for its thing"               │
│      User::MAX_AGE       (constant)                    │
│      User::$count        (static property)             │
│      User::getCount()    (static method)               │
│      self::$count        (same class, static)          │
│      static::create()    (late static binding)         │
│      parent::__construct() (parent class method)       │
└────────────────────────────────────────────────────────┘
```

> **Key rule:** `->` is for instances (objects). `::` is for classes (blueprints).

---

## Real-World Analogy

### The Car Factory 🏭

Imagine a **Car Factory** that produces cars.

| Concept | Factory Analogy | PHP |
|---------|----------------|-----|
| Instance property | The **color** of a specific car (each car has its own) | `$this->color` |
| Static property | **Total cars produced** — the factory's count, not any one car's | `self::$totalCars` |
| Instance method | `$myCar->startEngine()` — starts *this specific car* | `$object->method()` |
| Static method | `Car::getTotalProduced()` — ask the *factory* how many cars exist | `ClassName::method()` |

The "total cars produced" counter doesn't live inside any car. It lives on the **factory wall** (the class). When a new car rolls off the line, the factory counter goes up. When you ask "how many cars have been made?", you don't ask a specific car — you ask the factory.

---

## How PHP Implements This

### Bad Example: Trying to Count Instances Without `static`

```php
class Car {
    public string $color;
    public int $totalCars = 0;  // ❌ Each car gets its OWN copy of this

    public function __construct(string $color) {
        $this->color = $color;
        $this->totalCars++;  // ❌ Only increments THIS car's counter
    }
}

$car1 = new Car("Red");
$car2 = new Car("Blue");
$car3 = new Car("Black");

echo $car1->totalCars;  // 1 — only knows about itself
echo $car2->totalCars;  // 1 — only knows about itself
echo $car3->totalCars;  // 1 — only knows about itself
// ❌ We wanted 3! But each car has its own separate counter
```

```
┌─────────────┐  ┌─────────────┐  ┌──────────────┐
│   $car1      │  │   $car2      │  │   $car3      │
│ color: Red   │  │ color: Blue  │  │ color: Black │
│ totalCars: 1 │  │ totalCars: 1 │  │ totalCars: 1 │
└─────────────┘  └─────────────┘  └──────────────┘
  Each object has its OWN totalCars — they don't share!
```

**Why it fails:** `$this->totalCars` creates a **separate counter inside each object**. Objects don't share instance properties — that's the whole point of OOP. So each car thinks only 1 car was ever made (itself).

### Good Example: Using `static` for Shared Data

```php
class Car {
    // Static property — belongs to the CLASS, shared by all instances
    private static int $totalCars = 0;

    // Instance property — each object gets its own
    private string $color;

    public function __construct(string $color) {
        $this->color = $color;

        // self:: refers to the class itself, not $this (the instance)
        self::$totalCars++;
    }

    // Static method — can be called without creating an object
    public static function getTotalCars(): int {
        return self::$totalCars;
    }

    // Instance method — needs an object to call
    public function getColor(): string {
        return $this->color;
    }
}

$car1 = new Car("Red");
$car2 = new Car("Blue");
$car3 = new Car("Black");

// Call static method on the CLASS — no object needed
echo Car::getTotalCars();  // 3 ✅ — the class knows the total

// Call instance method on a specific OBJECT
echo $car1->getColor();   // "Red" — this specific car's color
echo $car2->getColor();   // "Blue"
```

```
┌──────────────────────────────────────────────────┐
│              Car (CLASS)                          │
│         static $totalCars = 3                    │
│              ┌──────┐                            │
│              │  3   │  ← shared by all instances │
│              └──────┘                            │
└──────────────┬──────────┬──────────┬─────────────┘
               │          │          │
        ┌──────▼──┐ ┌─────▼───┐ ┌───▼────────┐
        │  $car1  │ │  $car2  │ │  $car3     │
        │  Red    │ │  Blue   │ │  Black     │
        └─────────┘ └─────────┘ └────────────┘
         Each has own color, but totalCars lives on the CLASS
```

### Breaking Down the Syntax

```php
// DECLARING static members:
private static int $totalCars = 0;  // static PROPERTY
public static function getTotalCars(): int { ... }  // static METHOD

// ACCESSING from inside the class:
self::$totalCars      // ← use self:: (not $this->)

// ACCESSING from outside the class:
Car::getTotalCars()   // ← use ClassName:: (not $object->)
Car::$totalCars       // ← only if public (avoid this — use a method)
```

> **Note the `$` sign:** When accessing static properties, the `$` stays: `self::$totalCars`, `Car::$totalCars`. This is different from constants (`self::MAX_SPEED` — no `$`). The `$` tells PHP it's a variable (can change), not a constant (cannot change).

### A Simple Utility Class — The AppConfig Example

Sometimes a class is *entirely* static — it's just a container for utility data and functions with no instances needed:

```php
class AppConfig {
    // These values belong to the app, not to any specific object
    private static string $appName = "My Awesome App";
    private static string $version = "1.0.0";

    public static function getAppName(): string {
        return self::$appName;
    }

    public static function getVersion(): string {
        return self::$version;
    }

    // Utility method — no object state needed
    public static function isVersion(string $check): bool {
        return self::$version === $check;
    }
}

// No "new" keyword needed — these belong to the CLASS
echo AppConfig::getAppName();   // "My Awesome App"
echo AppConfig::getVersion();   // "1.0.0"
echo AppConfig::isVersion("2.0.0");  // false
```

> **Why not just use constants?** Use `const` when the value will **never** change (like `PI = 3.14159`). Use `static` when the value might change during the program (like a counter that increments, or a config that gets loaded from a file at startup). See the comparison table below.

### `static` vs `const` — When to Use Which

| Feature | `static` property | `const` |
|---------|-------------------|---------|
| **Can change at runtime?** | ✅ Yes (`self::$count++`) | ❌ No (immutable) |
| **Needs `$` in name?** | ✅ Yes (`$totalCars`) | ❌ No (`MAX_AGE`) |
| **Has visibility?** | ✅ `private static`, `public static` | ✅ `private const`, `public const` (PHP 7.1+) |
| **Accessed with** | `self::$prop` or `ClassName::$prop` | `self::CONST` or `ClassName::CONST` |
| **Good for** | Counters, caches, state that changes | Fixed configuration, limits, enum-like values |

```php
class MathHelper {
    public const PI = 3.14159;          // ← never changes, use const
    private static int $calculations = 0;  // ← changes over time, use static

    public static function circleArea(float $radius): float {
        self::$calculations++;
        return self::PI * ($radius ** 2);
        // ** is the exponentiation operator: $radius ** 2 means $radius²
    }

    public static function getCalculationCount(): int {
        return self::$calculations;
    }
}

echo MathHelper::PI;                    // 3.14159
echo MathHelper::circleArea(5);         // 78.53975
echo MathHelper::circleArea(10);        // 314.159
echo MathHelper::getCalculationCount(); // 2
```

---

## The Critical Piece: `self::` vs `static::` (Late Static Binding)

This is the concept you caught Gemini skipping, and it's **essential** for understanding how Laravel works internally.

### The Problem

```php
class ParentClass {
    protected static string $type = "parent";

    public static function getType(): string {
        return self::$type;  // ← self:: locks to THIS class (ParentClass)
    }
}

class ChildClass extends ParentClass {
    protected static string $type = "child";  // Override the value
}

echo ChildClass::getType();  // "parent" 😱 — NOT "child"!
```

**Why?** `self::` is **"greedy"** — it locks onto the class where the code is **physically written**. Since `getType()` is written inside `ParentClass`, `self::$type` always resolves to `ParentClass::$type`, even when called as `ChildClass::getType()`.

### The Fix: `static::`

```php
class ParentClass {
    protected static string $type = "parent";

    public static function getType(): string {
        return static::$type;  // ← static:: waits to see WHO called
    }
}

class ChildClass extends ParentClass {
    protected static string $type = "child";
}

echo ParentClass::getType();  // "parent" ✅
echo ChildClass::getType();   // "child"  ✅ — static:: resolved to ChildClass
```

**`static::` is "social"** — it waits to see which class is **actually called at runtime**, then resolves to that class.

### Visual: How Late Static Binding Works

```
    ChildClass::getType()                     CALL
         │
         ▼
    getType() is defined in ParentClass       LOOKUP
         │
         ▼
    self::$type  → "Who wrote this code?"     RESOLVE
    → ParentClass → "parent"                  ❌ WRONG
    
    static::$type → "Who CALLED this method?"
    → ChildClass  → "child"                   ✅ CORRECT
```

### `self::` vs `static::` Comparison

| | `self::` | `static::` |
|---|---------|-----------|
| **Resolves to** | The class where code is **written** | The class that was **called** |
| **Binding time** | Compile-time (early) | Runtime (late) |
| **Affected by inheritance?** | ❌ No — always the defining class | ✅ Yes — follows the call chain |
| **Analogy** | "Greedy" — claims it for itself | "Social" — looks at who's actually calling |
| **Use when** | You want to force the parent class | You want child classes to override behavior |

---

## Why This Matters for Laravel 🔥

### Example 1: Model::find() and Model::all()

```php
// You write this in your controller:
$user = User::find(1);
$posts = Post::all();
```

How does Laravel know to search the `users` table for `User::find()` and the `posts` table for `Post::all()`? Because internally, the base `Model` class uses `static::` — not `self::`:

```php
// Simplified version of what happens inside Laravel's Model class:
class Model {
    public static function find(int $id): static {
        // static:: resolves to the class that CALLED this method
        $instance = new static();  // If User::find(), creates a new User
                                   // If Post::find(), creates a new Post
        
        // $instance->getTable() returns "users" or "posts"
        // based on the actual class
        return $instance->where('id', $id)->first();
    }
}

class User extends Model {
    // User::find(1) → new static() creates a User → searches "users" table
}

class Post extends Model {
    // Post::find(1) → new static() creates a Post → searches "posts" table
}
```

> **If Laravel used `self::` instead of `static::`**, every model would create a generic `Model` object and try to search a nonexistent "models" table. Late Static Binding is what makes Eloquent work.

### Example 2: Facades — Static-Looking Syntax for Non-Static Code

```php
// This LOOKS static:
Cache::put('key', 'value', 600);
Auth::user();
Route::get('/home', [HomeController::class, 'index']);

// But it's NOT really static! Laravel's Facade pattern:
// 1. You call Cache::put(...)
// 2. PHP hits the __callStatic() magic method on the Facade class
// 3. The Facade resolves the real object from the Service Container
// 4. Calls put() on that real object instance
```

Facades are **syntactic sugar** — they give you the convenience of `ClassName::method()` syntax while actually using proper instances under the hood. You'll learn the full Facade pattern in [Section 3](../03-laravel-core-architecture/).

### Example 3: Factory Pattern in Eloquent

```php
// Creating models with static factory methods:
$user = User::create([
    'name' => 'Jane',
    'email' => 'jane@example.com',
    'password' => bcrypt('secret'),
]);

// This works because Model::create() uses static::
// to know it should create a User, not a generic Model
```

---

## When NOT to Use Static

Static isn't always better. Overusing it creates problems:

### Problem 1: Hidden Dependencies (Hard to Test)

```php
// ❌ Hard to test — OrderProcessor secretly depends on Database
class OrderProcessor {
    public function process(int $orderId): void {
        $order = Database::find($orderId);  // Hidden static call
        // How do you test this without a real database?
    }
}

// ✅ Better — dependency is visible and replaceable
class OrderProcessor {
    public function __construct(
        private DatabaseInterface $db  // Injected, can be mocked in tests
    ) {}

    public function process(int $orderId): void {
        $order = $this->db->find($orderId);
    }
}
```

### Problem 2: Shared Mutable State

```php
// ❌ Dangerous — any code anywhere can change this
class Config {
    public static string $dbHost = "localhost";
}

// Somewhere deep in your code:
Config::$dbHost = "hacked-server.com";  // 😱 Global state changed silently
```

> **Rule of thumb:** Use static for **read-only utilities**, **factory methods**, and **counters**. If you need mutable state or testable dependencies, use regular instances with dependency injection (which Laravel is built around — see [Section 2](../02-how-laravel-thinks/)).

---

## Common Mistakes

### Mistake 1: Using `$this` Inside a Static Method

```php
class Counter {
    private static int $count = 0;
    private string $name;

    public static function increment(): void {
        $this->name;  // ❌ Fatal Error: Using $this when not in object context
        // Static methods belong to the CLASS — there is no "this" object
    }
}
```

**Why it fails:** A static method can be called without creating an object (`Counter::increment()`). If there's no object, `$this` has nothing to point to. `$this` only exists inside instance methods.

**Fix:** Use `self::` for static members, or rethink whether the method should be static at all.

### Mistake 2: Forgetting `$` When Accessing Static Properties

```php
echo Car::totalCars;     // ❌ PHP thinks you want a CONSTANT named totalCars
echo Car::$totalCars;    // ✅ The $ tells PHP it's a static PROPERTY
```

### Mistake 3: Using `self::` When You Mean `static::`

```php
class BaseModel {
    public static function create(array $data): self {
        return new self();   // ❌ Always creates a BaseModel, even for children
    }
}

class User extends BaseModel {}

$user = User::create(['name' => 'Jane']);
// $user is a BaseModel, NOT a User! 😱
```

**Fix:**
```php
public static function create(array $data): static {
    return new static();  // ✅ Creates whatever class was actually called
}
```

> **Interview question:** "Can you use `$this` inside a static method?" **No.** A static method belongs to the class blueprint, not a specific object instance. `$this` (which refers to the current instance) doesn't exist in that context.

---

## Key Takeaways

1. **`static` properties/methods belong to the class**, not to any instance — they're shared across all objects
2. **`::` is the Scope Resolution Operator** — used for class-level access (static, constants, parent)
3. **`self::` resolves to the class where code is written** — "greedy", locked at compile time
4. **`static::` resolves to the class that was called** — "social", resolved at runtime (Late Static Binding)
5. **Late Static Binding is how Laravel works** — `User::find()`, `Post::create()`, model factories — all rely on `static::` to know which child class was actually called
6. **`static` vs `const`:** Use `const` for values that never change, `static` for values that change at runtime
7. **Static properties keep `$`** in access: `self::$count`, `Car::$totalCars` — no `$` means PHP looks for a constant
8. **`$this` does not exist in static methods** — there's no instance to point to
9. **Don't overuse static** — it hides dependencies and makes testing harder. Prefer dependency injection for mutable state
10. **Facades look static but aren't** — Laravel uses magic methods to route `Cache::put()` calls to real object instances

---

> **Next up:** [08-inheritance.md](08-inheritance.md) — How classes can inherit from other classes, `extends`, and the is-a vs has-a distinction

## Key Takeaways

Bullet-point summary of what to remember.
