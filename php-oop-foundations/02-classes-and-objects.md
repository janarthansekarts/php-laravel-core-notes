# Classes and Objects (Deep Dive)

> **Prerequisites:** [01 - Why OOP Exists](01-why-oop-exists.md)
> **Next:** [03 - Properties and Methods](03-properties-and-methods.md)

---

## The Problem / Why This Exists

In [01-why-oop-exists](01-why-oop-exists.md), we learned that OOP bundles **data** and **behavior** together into a single unit. But we glossed over the actual mechanics — *how* do you create that unit? *How* does PHP know what data and behavior belong together?

This is where **Classes** and **Objects** come in. They are the two most fundamental building blocks of OOP:

- A **Class** defines the *structure* — what data something will hold and what it can do
- An **Object** is a *real instance* of that structure — actually living in memory with real values

You can't have OOP without understanding this distinction clearly.

---

## What It Introduces

### Class — The Blueprint

A **class** is a blueprint or template. It says:
- "Things of this type will have *these* properties (data)"
- "Things of this type can do *these* methods (behavior)"

But a class is **not** actual data. It's a definition. Just like an architect's blueprint isn't a house — it's a plan for how to build one.

```php
// This is a CLASS — a blueprint. No actual user exists yet.
class User
{
    private string $name;
    private string $email;

    public function __construct(string $name, string $email)
    {
        $this->name = $name;
        $this->email = $email;
    }

    public function getDisplayName(): string
    {
        return $this->name;
    }
}
// At this point, ZERO users exist. We've only described what a User looks like.
```

#### Class Syntax Rules

- The `class` keyword defines a class, followed by the **class name** and curly braces `{}`
- Class names must start with a **letter or underscore**, followed by letters, numbers, or underscores
- By convention, class names use **PascalCase** (each word capitalized): `User`, `ShoppingCart`, `EmailNotification`
- One class per file (by convention and PSR-4 standard — covered in [17-namespaces-and-autoloading](17-namespaces-and-autoloading.md))
- The file name should match the class name: `User` class → `User.php` file
- Everything inside `{ }` belongs to that class — properties, methods, constants

### Object — The Real Thing

An **object** is an **instance** of a class — a real thing created in memory using the blueprint. You create an object using the `new` keyword.

> **What does "instance" mean?** It literally means "one specific occurrence of something." If the `User` class is the concept of a user, then `$user1` is one *instance* (one specific occurrence) of that concept. The terms "object" and "instance" are interchangeable — "creating an instance" and "creating an object" mean the same thing. The verb form is **"instantiate"** — "we instantiate the User class" means "we create a User object."

```php
// NOW real users exist — these are OBJECTS (also called INSTANCES)
$user1 = new User("Jegan", "jegan@example.com");  // Instantiating the User class
$user2 = new User("Arun", "arun@example.com");    // Another instance
```

Each object:
- Has its **own copy** of the properties (its own `$name`, its own `$email`)
- Shares the **same methods** defined by the class
- Is **independent** — changing `$user1` does NOT affect `$user2`

### The `->` Operator (Object Operator)

You'll see `->` constantly in PHP OOP. It's the **object operator** — it lets you access an object's properties and methods.

```php
$user1->getDisplayName();   // Call the getDisplayName() METHOD on $user1
$user1->name;               // Access the name PROPERTY on $user1 (if allowed)
```

**How to read it:** Think of `->` as saying **"'s"** (possessive). `$user1->getDisplayName()` reads as **"user1's getDisplayName()"**.

> **Why `->` and not `.` like other languages?** In JavaScript/Python/Java, you use `$user.name`. PHP chose `->` because the `.` (dot) was already taken — PHP uses `.` for string concatenation (`"Hello" . " World"`). So PHP uses `->` for object access instead. That's it — purely a syntax choice, not a conceptual difference.

### The Relationship

```
┌─────────────────────────────────────────┐
│              Class: User                │
│            (The Blueprint)              │
│                                         │
│  Properties: $name, $email              │
│  Methods: getDisplayName()              │
└──────────┬──────────┬───────────────────┘
           │          │
     new User()   new User()
           │          │
           ▼          ▼
    ┌────────────┐  ┌────────────┐
    │  Object 1  │  │  Object 2  │
    │  $user1    │  │  $user2    │
    │            │  │            │
    │  name:     │  │  name:     │
    │  "Jegan"   │  │  "Arun"    │
    │            │  │            │
    │  email:    │  │  email:    │
    │  "jegan@"  │  │  "arun@"   │
    └────────────┘  └────────────┘
    
    Same blueprint, different data.
    Changing $user1 does NOT affect $user2.
```

---

## Real-World Analogy

### Class = Car Design Document

A car manufacturer's design document says: "All cars of this model have a **color**, an **engine type**, and can **drive()** and **brake()**."

That document isn't a car. You can't drive a piece of paper.

### Object = An Actual Car

When the factory uses that design to build a **Blue Tesla** and a **Red Ford** — those are objects. They are real, they exist, you can drive them. Each has its own color, its own engine. Painting one car red doesn't change the other car's color.

### The `new` Keyword = The Factory

Calling `new Car("blue")` is like telling the factory: **"Build me one car using this design, painted blue."** Each `new` call creates a completely independent car.

---

## How PHP Implements This

### Bad Example: Procedural Approach

```php
<?php
// --- Procedural: Managing Users ---

// Users are just loose arrays — no structure enforced
$user1 = [
    'name' => 'Jegan',
    'email' => 'jegan@example.com',
];

$user2 = [
    'name' => 'Arun',
    'email' => 'arun@example.com',
];

// Nothing stops us from adding nonsensical data
$user1['flavor'] = 'chocolate';  // Makes no sense for a user, but PHP allows it
$user2['email'] = 12345;         // Email should be a string, but no one enforces it

// Functions have no formal connection to user data
function getUserDisplayName(array $user): string {
    return $user['name'];
}

// We have to pass the array around everywhere
function updateUserEmail(array &$user, string $newEmail): void {
    $user['email'] = $newEmail;
}

// What if we misspell a key?
echo $user1['naem'];  // No error, just a PHP notice and empty result
                       // Typos become silent bugs
```

**What's wrong here:**
- Arrays have **no enforced structure** — you can add `flavor` to a user and PHP won't complain
- **No type safety** — email can be set to an integer, no one stops it
- **Typos become silent bugs** — `$user['naem']` doesn't throw an error
- Functions are **disconnected** from the data they operate on — you have to pass arrays around manually
- If the user structure changes (e.g., `name` splits into `firstName` and `lastName`), you have to hunt through every function that touches user arrays

### Good Example: OOP Approach

```php
<?php
// --- OOP: Managing Users ---

class User
{
    private string $name;
    private string $email;

    public function __construct(string $name, string $email)
    {
        $this->name = $name;
        $this->setEmail($email);
    }

    public function getDisplayName(): string
    {
        return $this->name;
    }

    public function getEmail(): string
    {
        return $this->email;
    }

    public function setEmail(string $email): void
    {
        $sanitized = filter_var(trim(strtolower($email)), FILTER_SANITIZE_EMAIL);

        if (!filter_var($sanitized, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException("Invalid email: $email");
        }

        $this->email = $sanitized;
    }
}

// --- Creating Objects ---
$user1 = new User("Jegan", "jegan@example.com");
$user2 = new User("Arun", "arun@example.com");

// ✅ Data and behavior are locked together
echo $user1->getDisplayName();  // "Jegan"
echo $user2->getDisplayName();  // "Arun"

// ✅ Can't add random properties
// $user1->flavor = "chocolate"; // ❌ Error! "flavor" is not a property of User

// ✅ Type safety — PHP enforces string type
// $user1->setEmail(12345);      // ❌ TypeError! Must be a string

// ✅ Validation at the boundary
// $user1->setEmail("not-an-email"); // ❌ InvalidArgumentException!

// ✅ Each object is independent
$user1->setEmail("jegan.new@example.com");
echo $user2->getEmail();  // Still "arun@example.com" — unaffected
```

**What's better here:**
- The class **enforces structure** — a User can only have `$name` and `$email`, nothing else
- **Type hints** (`string $name`) ensure the right data types — PHP throws a `TypeError` if violated
- **Validation** happens at the boundary (in `setEmail()`) — bad data is rejected immediately (remember the "validate early" principle from [file 01](01-why-oop-exists.md))
- Data and behavior are **locked together** — `getDisplayName()` lives with the data it uses
- Each object is **independent** — changing one doesn't affect others
- **Typos in property names cause real errors**, not silent bugs

---

## What Happens When You Call `new`

Understanding what `new` does helps demystify objects:

```php
$user = new User("Jegan", "jegan@example.com");
```

Step by step:

1. **PHP allocates memory** for a new User object
2. **PHP copies the class structure** — the object gets its own `$name` and `$email` slots
3. **PHP calls `__construct()`** — the constructor method runs, filling in the property values
4. **PHP returns a reference** to the object — `$user` now points to this object in memory

```
$user ──────→ [ User Object in Memory ]
                  ├── $name = "Jegan"
                  └── $email = "jegan@example.com"
```

> **Important:** `$user` holds a **reference** (a pointer) to the object, not the object itself. This matters when you pass objects to functions — more on this in later files.

### Multiple Objects = Multiple Chunks of Memory

```php
$user1 = new User("Jegan", "jegan@example.com");
$user2 = new User("Arun", "arun@example.com");
$user3 = new User("Priya", "priya@example.com");
```

```
$user1 ──→ [ User Object #1: name="Jegan", email="jegan@..." ]
$user2 ──→ [ User Object #2: name="Arun",  email="arun@..."  ]
$user3 ──→ [ User Object #3: name="Priya", email="priya@..." ]

Three separate objects in memory.
Same class. Different data. Completely independent.
```

---

## Object Identity vs. Equality

A subtle but important concept:

```php
$user1 = new User("Jegan", "jegan@example.com");
$user2 = new User("Jegan", "jegan@example.com");  // Same data!

// == checks if the properties have the same values
var_dump($user1 == $user2);   // true — same data

// === checks if they are the EXACT SAME object in memory
var_dump($user1 === $user2);  // false — two different objects

// To make them the same object:
$user3 = $user1;              // $user3 now points to the SAME object as $user1
var_dump($user1 === $user3);  // true — same object in memory

$user3->setEmail("new@example.com");
echo $user1->getEmail();      // "new@example.com" — because $user3 IS $user1!
```

> **Technical note:** Strictly speaking, PHP passes objects by **"reference to the object handle"** — not exactly the same as pass-by-reference with `&`. The practical effect is the same for assignment: `$user3 = $user1` makes both variables point to the same object. But in function parameters, there's a subtle difference covered in later files. For now, just remember: **assigning an object to another variable does NOT copy it.**

---

## How Laravel Relies on This Concept

In Laravel, **everything** is a class, and you're constantly working with **objects**:

### Models Are Objects

```php
// Laravel doesn't give you an array — it gives you a User OBJECT
$user = User::find(1);

// $user is now an object of the User class (App\Models\User)
// It has properties (name, email, etc.) AND methods:
echo $user->name;              // Property access
$user->email = "new@mail.com"; // Property modification (Eloquent handles this)
$user->save();                 // Method call — saves to database

// This is ONLY possible because $user is an object, not an array.
// An array can't have a save() method.
```

### Controllers Are Classes, Requests Are Objects

```php
// This controller is a CLASS
class UserController extends Controller
{
    // When Laravel calls this method, $request is an OBJECT
    // created from the Request class — it holds all HTTP data
    public function store(Request $request)
    {
        // $request is an object with methods:
        $name = $request->input('name');     // Method call
        $request->validate(['name' => 'required']); // Method call
    }
}
```

### Collections Are Objects

```php
// User::all() returns a Collection OBJECT, not a plain array
$users = User::all();

// Because it's an object, it has powerful methods:
$activeUsers = $users->filter(fn($u) => $u->status === 'active');
$names = $users->pluck('name');
$first = $users->first();

// A plain array can't do ->filter() or ->pluck().
// That's the power of objects — data + behavior together.
```

### Why This Matters

| Scenario | If Laravel Used Arrays | Because Laravel Uses Objects |
|---|---|---|
| Save to DB | You'd write raw SQL every time | `$user->save()` — the object knows how |
| Validate data | Call a standalone function | `$request->validate()` — built into the object |
| Filter a list | Write loops manually | `$users->filter()` — the collection object has the method |
| Access related data | Write join queries | `$user->posts` — the model object handles the relationship |

---

## Common Mistakes

### 1. The "God Class"
Beginners often try to put everything into one massive class (e.g., a `Website` class that handles users, products, emails, and payments). A class should represent **one thing** with **one responsibility**. We'll formalize this as the **Single Responsibility Principle** in [16-solid-principles](16-solid-principles.md).

### 2. Confusing Class vs. Object
Trying to call a method on the class blueprint instead of a created object:

```php
// ❌ WRONG — User is the class (blueprint), not an object
User->getDisplayName();

// ✅ RIGHT — $user is the object (instance)
$user = new User("Jegan", "jegan@example.com");
$user->getDisplayName();

// The exception: static methods (called on the class directly) — covered in file 07
User::find(1);  // This is a static call — we'll explain why it's different
```

### 3. Forgetting `new`
```php
// ❌ WRONG — this doesn't create an object
$user = User("Jegan", "jegan@example.com");  // PHP thinks you're calling a function!

// ✅ RIGHT — `new` tells PHP to create an object from the class
$user = new User("Jegan", "jegan@example.com");
```

### 4. Thinking Objects Are Copies When Assigned
```php
$user1 = new User("Jegan", "jegan@example.com");
$user2 = $user1;  // ⚠️ This does NOT create a copy!

$user2->setEmail("changed@example.com");
echo $user1->getEmail();  // "changed@example.com" — $user1 was affected!

// To make an actual copy, use the `clone` keyword:
$user2 = clone $user1;   // Now they're independent

$user2->setEmail("different@example.com");
echo $user1->getEmail();  // Still "changed@example.com" — unaffected!
```

> **About `clone`:** The `clone` keyword creates a **shallow copy** — it copies all property values into a new object. For simple types (strings, integers), this works perfectly. But if a property holds another *object*, `clone` only copies the reference to that inner object, not the inner object itself. This is called the "shallow copy problem" — you can handle it by defining a `__clone()` magic method in your class, but that's an advanced topic for later.

### 5. Using Arrays When You Should Use Classes
If you find yourself creating associative arrays with consistent keys (like `['name' => ..., 'email' => ...]`), that's a sign you should be using a class. Arrays have no structure enforcement, no methods, and no type safety.

---

## Useful Tools: Checking an Object's Class

When debugging or writing conditional logic, you often need to know **what class** an object belongs to:

### `instanceof` — Check If an Object Is of a Specific Class

```php
$user = new User("Jegan", "jegan@example.com");

if ($user instanceof User) {
    echo "Yes, this is a User object";  // ✅ This runs
}

if ($user instanceof Product) {
    echo "This is a Product";           // ❌ This does NOT run
}
```

`instanceof` is a **keyword** (not a function) — it returns `true` or `false`. You'll use this heavily in Laravel when checking if an object is a specific model, exception type, etc.

### `get_class()` — Get the Class Name as a String

```php
$user = new User("Jegan", "jegan@example.com");
echo get_class($user);  // "User" (or "App\Models\User" in Laravel with namespaces)
```

Useful for debugging — "what type of object is this variable holding?"

### `::class` — Get the Fully Qualified Class Name

```php
echo User::class;  // "User" (or "App\Models\User" with namespaces)
```

This is a compile-time constant — useful for referencing class names without creating an object. You'll see this everywhere in Laravel:

```php
// Laravel uses ::class constantly:
Route::get('/users', [UserController::class, 'index']);
```

---

## Key Takeaways

- A **class** is a blueprint — it defines properties (data) and methods (behavior) but doesn't hold real data yet
- An **object** (also called an **instance**) is created from a class with `new` — it lives in memory with actual values
- **Instantiate** = create an object from a class. "Instance" and "object" mean the same thing
- `new ClassName()` allocates memory, copies the class structure, runs `__construct()`, and returns a reference
- The **`->` operator** accesses an object's properties and methods — read it as **"'s"** (`$user->name` = "user's name")
- Class names use **PascalCase** (`ShoppingCart`), one class per file, file name matches class name
- Each object is **independent** — same class, different data, changing one doesn't affect others
- `$b = $a` makes both point to the **same** object (not a copy) — use `clone` for an actual copy
- `clone` creates a **shallow copy** — simple property values are copied, but nested objects still share references
- `==` checks if two objects have the same property values; `===` checks if they're the exact same object in memory
- Use `instanceof` to check an object's class, `get_class()` to get the class name, `::class` for the class name constant
- In **Laravel**, everything is objects: Models, Requests, Collections, Controllers — that's why they have methods like `->save()`, `->validate()`, `->filter()`
- If you're using arrays with consistent keys, consider using a class instead — you get structure, type safety, and validation for free

---

> **Next up:** [03 - Properties and Methods](03-properties-and-methods.md) — We'll dive into properties and methods in detail — the data and behavior that live inside classes.
