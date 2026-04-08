# The `$this` Keyword

> **Prerequisites:** [05 - Constructors and Destructors](05-constructors-and-destructors.md)
> **Next:** [07 - Static Properties and Methods](07-static-properties-and-methods.md)

---

## The Problem / Why This Exists

We've used `$this` since file 01, but never stopped to explain exactly what it is and why it exists. Consider this problem:

```php
class User
{
    private string $name;

    public function __construct(string $name)
    {
        // Both the parameter and the property are called "name"
        // How does PHP know which one you mean?
        $name = $name;  // ❌ This just assigns the parameter to itself!
    }
}
```

The **parameter** `$name` and the **property** `$name` have the same name. Without a way to say "I mean the **object's** property, not the local variable," PHP can't tell them apart.

**`$this` solves this.** It's a reference to the **current object** — the specific instance that's running the code right now.

```php
$this->name = $name;
//  │     │      │
//  │     │      └── The parameter (local variable)
//  │     └── The property (on this object)
//  └── "This object" — the current instance
```

---

## What It Introduces

`$this` is a **pseudo-variable** — you don't create it, declare it, or assign it. PHP automatically makes it available **inside any non-static method**. It always refers to the object that the method is being called on.

```php
class User
{
    private string $name;

    public function __construct(string $name)
    {
        $this->name = $name;  // "Set THIS object's name to the parameter value"
    }

    public function greet(): string
    {
        return "Hello, I'm {$this->name}";  // "Read THIS object's name"
    }
}

$user1 = new User("Jegan");
$user2 = new User("Arun");

echo $user1->greet();  // "Hello, I'm Jegan"  — $this points to $user1
echo $user2->greet();  // "Hello, I'm Arun"   — $this points to $user2
```

**Key insight:** The same `greet()` method code is shared by all objects, but `$this` **changes** depending on which object calls it. When `$user1->greet()` runs, `$this` is `$user1`. When `$user2->greet()` runs, `$this` is `$user2`.

---

## Real-World Analogy

Think of a **name badge** at a conference.

Every attendee follows the same set of instructions: "When someone asks your name, **look at your own badge** and read it."

The instruction "look at your own badge" is the same for everyone — but the result is different because each person has their own badge with their own name.

`$this` is the equivalent of "your own badge" — it tells each object to look at **itself**.

---

## How `$this` Works in Detail

### Accessing Properties with `$this`

```php
class BankAccount
{
    private string $owner;
    private float $balance;

    public function __construct(string $owner, float $balance = 0)
    {
        $this->owner = $owner;      // Set THIS object's owner
        $this->balance = $balance;   // Set THIS object's balance
    }

    public function deposit(float $amount): void
    {
        $this->balance += $amount;   // Modify THIS object's balance
    }

    public function getBalance(): float
    {
        return $this->balance;       // Read THIS object's balance
    }
}

$account1 = new BankAccount("Jegan", 1000);
$account2 = new BankAccount("Arun", 500);

$account1->deposit(200);  // $this = $account1 → balance becomes 1200
$account2->deposit(100);  // $this = $account2 → balance becomes 600

echo $account1->getBalance();  // 1200 — $account2 is unaffected
echo $account2->getBalance();  // 600  — $account1 is unaffected
```

### Calling Methods with `$this`

An object can call **its own methods** using `$this`:

```php
class User
{
    private string $name;
    private string $email;

    public function __construct(string $name, string $email)
    {
        $this->name = $name;
        $this->setEmail($email);  // Calling another method on THIS object
    }

    public function setEmail(string $email): void
    {
        $sanitized = filter_var(trim(strtolower($email)), FILTER_SANITIZE_EMAIL);

        if (!filter_var($sanitized, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException("Invalid email: $email");
        }

        $this->email = $sanitized;
    }

    public function toArray(): array
    {
        // $this-> gives us access to all properties and methods of this object
        return [
            'name' => $this->name,
            'email' => $this->email,
            'display' => $this->getDisplayName(),  // Calling own method
        ];
    }

    public function getDisplayName(): string
    {
        return $this->name;
    }
}
```

### `$this` Enables Method Chaining

In [file 03](03-properties-and-methods.md), we learned method chaining. Now you can see **why** it works — it's because methods `return $this`:

```php
class QueryBuilder
{
    private string $table = '';
    private array $conditions = [];

    public function from(string $table): self
    {
        $this->table = $table;
        return $this;  // Return THIS object — so the next -> can chain
    }

    public function where(string $condition): self
    {
        $this->conditions[] = $condition;
        return $this;  // Return THIS same object again
    }

    public function toSQL(): string
    {
        $sql = "SELECT * FROM {$this->table}";
        if (!empty($this->conditions)) {
            $sql .= " WHERE " . implode(' AND ', $this->conditions);
        }
        return $sql;
    }
}

// Each method returns $this, so -> calls keep operating on the SAME object:
$sql = (new QueryBuilder())
    ->from('users')              // $this = the QueryBuilder object → returns itself
    ->where('active = 1')        // $this = same object → returns itself
    ->where('role = "admin"')    // $this = same object → returns itself
    ->toSQL();                   // $this = same object → returns the SQL string

// "SELECT * FROM users WHERE active = 1 AND role = "admin""
```

```
Step by step:

new QueryBuilder()     → Creates object A
    ->from('users')    → $this = A, sets table, returns A
    ->where(...)       → $this = A, adds condition, returns A
    ->where(...)       → $this = A, adds condition, returns A
    ->toSQL()          → $this = A, builds and returns SQL string

It's the SAME object (A) the entire time. That's why chaining works.
```

---

## `$this` vs `self::` vs `static::`

This is a common source of confusion. Here's the quick version (we'll cover `self` and `static` deeply in [file 07](07-static-properties-and-methods.md)):

```php
class User
{
    private string $name;
    private const MAX_NAME_LENGTH = 50;                  // Class constant
    private static int $totalUsers = 0;                   // Static property

    public function __construct(string $name)
    {
        // $this-> accesses INSTANCE members (properties/methods on this object)
        $this->name = $name;

        // self:: accesses CLASS-LEVEL members (constants, static properties/methods)
        if (strlen($name) > self::MAX_NAME_LENGTH) {
            throw new InvalidArgumentException("Name too long");
        }

        self::$totalUsers++;
    }
}
```

| Syntax | Accesses | Points To |
|---|---|---|
| `$this->property` | Instance property | The current **object** |
| `$this->method()` | Instance method | The current **object** |
| `self::CONSTANT` | Class constant | The **class** itself |
| `self::$staticProp` | Static property | The **class** itself |
| `self::method()` | Static method | The **class** itself |

> **Simple rule:** `$this->` = "this specific object's stuff." `self::` = "this class's shared stuff." We'll explore `self` and `static` fully in the next file.

---

## Where `$this` Does NOT Exist

`$this` is only available inside **instance methods** — methods called on an object. It does **not** exist in:

### Static Methods

```php
class MathHelper
{
    // Static method — called on the CLASS, not on an object
    public static function add(int $a, int $b): int
    {
        // ❌ $this does NOT exist here — there's no object!
        // echo $this->something;  // Fatal error: Using $this when not in object context

        return $a + $b;
    }
}

// Called on the class, not on an object:
MathHelper::add(5, 3);  // No object involved → no $this
```

> **Why?** `$this` means "the current object." Static methods don't have a current object — they belong to the class itself, not to any instance. Covered in detail in [file 07](07-static-properties-and-methods.md).

### Outside of Classes

```php
// ❌ $this has no meaning outside a class
echo $this->name;  // Fatal error: Using $this when not in object context
```

---

## How Laravel Relies on This Concept

### Models — `$this` Everywhere

```php
class User extends Model
{
    protected $fillable = ['name', 'email'];

    // $this refers to the current User MODEL INSTANCE
    public function getFullNameAttribute(): string
    {
        return $this->first_name . ' ' . $this->last_name;
        //     ^^^^^^                     ^^^^^^
        //     THIS user's first_name     THIS user's last_name
    }

    public function posts()
    {
        return $this->hasMany(Post::class);
        //     ^^^^^^
        //     THIS user's posts (not all posts in the database)
    }

    public function activate(): void
    {
        $this->status = 'active';
        $this->activated_at = now();
        $this->save();  // Save THIS user to the database
    }
}

$jegan = User::find(1);
$arun = User::find(2);

$jegan->activate();  // $this = $jegan → saves Jegan's data
$arun->activate();   // $this = $arun  → saves Arun's data
```

### Controllers — `$this` for Internal Method Calls

```php
class OrderController extends Controller
{
    public function __construct(
        private OrderService $orderService,
    ) {}

    public function store(Request $request): JsonResponse
    {
        $data = $request->validated();
        $order = $this->orderService->create($data);
        //       ^^^^^^
        //       THIS controller's orderService (injected via constructor)

        $this->notifyAdmin($order);
        //   ^^^^^^
        //   Call THIS controller's private method

        return response()->json($order, 201);
    }

    private function notifyAdmin(Order $order): void
    {
        // internal helper method
    }
}
```

### Method Chaining in Eloquent — Powered by `$this`

```php
// Every method in the chain returns $this (the query builder object):
$users = User::query()
    ->where('active', true)     // returns $this
    ->where('age', '>', 18)     // returns $this
    ->orderBy('name')           // returns $this
    ->limit(10)                 // returns $this
    ->get();                    // returns the Collection (end of chain)
```

---

## Common Mistakes

### 1. The `$this->$property` Typo

```php
class User
{
    private string $name;

    public function getName(): string
    {
        // ❌ WRONG — $this->$name (with $ before name)
        return $this->$name;
        // PHP interprets this as: "access the property whose name is stored in $name"
        // This is called a VARIABLE VARIABLE — almost never what you want

        // ✅ RIGHT — $this->name (no $ before property name)
        return $this->name;
    }
}
```

> **Why does `$this->$name` even work?** It's a PHP feature called **variable variables** — `$this->$name` means "access the property whose name is the VALUE of `$name`." If `$name = "email"`, then `$this->$name` accesses `$this->email`. Useful in rare dynamic cases, but confusing as a typo.

### 2. Forgetting `$this->` and Using the Property Name Directly

```php
class User
{
    private string $name;

    public function __construct(string $name)
    {
        // ❌ WRONG — local variable assigned to itself
        $name = $name;  // Does nothing useful!

        // ✅ RIGHT — assign parameter to the object's property
        $this->name = $name;
    }

    public function greet(): string
    {
        // ❌ WRONG — $name doesn't exist as a local variable here
        return "Hello, $name";  // PHP Warning: Undefined variable $name

        // ✅ RIGHT — use $this-> to access the object's property
        return "Hello, {$this->name}";
    }
}
```

### 3. Using `$this` in a Static Method

```php
class Counter
{
    private static int $count = 0;

    public static function increment(): void
    {
        // ❌ WRONG — $this doesn't exist in static context
        $this->count++;  // Fatal error!

        // ✅ RIGHT — use self:: for static properties
        self::$count++;
    }
}
```

### 4. Confusing `$this` Between Objects

```php
class User
{
    private string $name;

    public function __construct(string $name)
    {
        $this->name = $name;
    }

    // $this is NOT shared between objects — each object has its own
    public function isSameAs(User $other): bool
    {
        // $this->name = THIS object's name
        // $other->name = the OTHER object's name
        // Even though both are User objects, $this is different for each
        return $this->name === $other->name;
    }
}

$user1 = new User("Jegan");
$user2 = new User("Arun");

$user1->isSameAs($user2);
// Inside this call: $this = $user1, $other = $user2
// So: $this->name = "Jegan", $other->name = "Arun"
// Result: false
```

> **Note:** Inside `isSameAs()`, `$this` can access `$other->name` even though it's `private`. This is because they're both `User` objects — PHP allows objects of the **same class** to access each other's private members. This is by design — it enables comparison methods like this.

---

## Key Takeaways

- **`$this`** is a pseudo-variable that refers to the **current object instance** — the specific object running the code
- It's how an object accesses **its own** properties (`$this->name`) and methods (`$this->method()`)
- `$this` is **not shared** between objects — when `$user1->greet()` runs, `$this` = `$user1`; when `$user2->greet()` runs, `$this` = `$user2`
- **Method chaining** works because methods `return $this` — each `->` in the chain operates on the same object
- **`$this->name`** (no `$` before property name) is correct; **`$this->$name`** (with `$`) is a variable variable — different thing entirely
- **`$this` does not exist** in static methods or outside of classes — it only lives inside instance methods
- **`$this->` vs `self::`**: Use `$this->` for instance members (properties/methods per object), use `self::` for class-level members (constants, static properties)
- Objects of the **same class** can access each other's private members — enabling comparison methods
- In **Laravel**, `$this` is used everywhere: `$this->hasMany()` in models, `$this->orderService` in controllers, `$this->validate()` in form requests

---

> **Next up:** [07 - Static Properties and Methods](07-static-properties-and-methods.md) — What happens when something belongs to the **class itself** rather than to any specific object? That's where `static`, `self::`, and `::` come in.

## Key Takeaways

Bullet-point summary of what to remember.
