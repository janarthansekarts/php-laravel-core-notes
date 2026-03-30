# Why Object-Oriented Programming Exists

> **Prerequisites:** Basic PHP syntax (variables, functions, arrays, loops)
> **Next:** [02 - Classes and Objects](02-classes-and-objects.md)

---

## The Problem / Why This Exists

In your first couple of years as a developer, you likely wrote **Procedural (function‑oriented) PHP** — a linear, top-down approach where you have **data** (variables) and **functions** that act on that data separately.

This works fine for small scripts. But the moment your application grows — more pages, more features, more developers — procedural code turns into a maintenance nightmare. Here's why:

### 1. Data and Logic Are Separated

In procedural PHP, variables(data) live on their own, and functions live on their own. There's no formal connection between them. Any function can read or modify any variable if it has access to it.

### 2. Global State Becomes Dangerous

If you have a `$user_status` variable used by 50 different functions, and one function accidentally changes it to an invalid value, your **entire app breaks** — and debugging it means tracing through every function that ever touches that variable.

### 3. Code Duplication Everywhere

Without a way to bundle reusable logic, you end up **copy-pasting** the same blocks of code across files. When a bug is found, you have to fix it in 10 different places.

### 4. No Clear Boundaries

As the codebase grows, there's no clear ownership. Who handles user data? Who validates the email? Who sends the notification? In procedural code, the answer is often "a function somewhere in some file" — and you have to hunt for it.

---

## What OOP Introduces

OOP is not about using more classes — it’s about creating boundaries where change is dangerous.

OOP wasn’t introduced to make code “cleaner” — it was introduced to manage growing complexity in long‑lived systems, where change is inevitable.

OOP solves these problems by introducing one core idea:

> **Group data and the behavior that operates on that data into a single unit called a Class.**

This concept is called **Encapsulation** — the most fundamental idea in OOP.

Instead of having scattered variables and scattered functions, you create a **Class** that:
- **Holds its own data** (called **Properties** — same concept as variables, but living inside a class)
- **Defines its own behavior** (called **Methods** — same concept as functions, but living inside a class)
- **Controls who can access what** (called **Visibility** — public, private, protected)

> **Terminology note:** A **function** and a **method** are the same thing — the only difference is *where they live*. A function that lives inside a class is called a **method**. Similarly, a variable inside a class is called a **property**. Throughout this series, "method" always means "a function inside a class" and "property" always means "a variable inside a class."

The data is **protected** inside the class. Outside code can't randomly change it. It has to go through the methods the class exposes — like a controlled API.

> **Key principle: Validate early, reject bad data at the boundary.** When data enters an object (through the constructor or a setter method), validate it immediately. Once data is inside the object, it should *always* be in a valid state. You'll see this pattern throughout the OOP examples in this series.

### The Four Pillars of OOP (Preview)

OOP is built on four core concepts. You don't need to understand them deeply yet — we'll cover each one in its own file — but here's the map:

| Pillar | What It Means | Covered In |
|---|---|---|
| **Encapsulation** | Bundling data + behavior together, hiding internals | This file + [04-visibility](04-visibility.md) |
| **Inheritance** | Creating new classes based on existing ones | [08-inheritance](08-inheritance.md) |
| **Polymorphism** | Different classes responding to the same method call differently | [12-interfaces](12-interfaces.md) |
| **Abstraction** | Hiding complex implementation behind simple interfaces | [11-abstract-classes](11-abstract-classes.md) |

---

## Real-World Analogy

### Procedural = A Messy Workshop

Tools (functions) and materials (data) are scattered everywhere. If you need to build a chair, you grab a saw from one corner and wood from another. As the workshop gets bigger, you spend more time **looking for tools** than actually building. Anyone can walk in and move your tools or use your materials without asking.

### OOP = A Specialized Factory

The **"Chair Department"** has its own tools, its own wood, and its own workers. If you want a chair, you talk to the department. You don't care *how* they saw the wood — you just want the result. Each department manages its own resources, and outsiders can't mess with their internal process.

---

## How PHP Implements This

### Bad Example: Procedural Approach

```php
<?php
// --- Procedural: User Management ---

// Data is just floating around
$user_name = "Jegan";
$user_email = "jegan@example.com";
$user_status = "active";

// Functions that act on user data — but have no formal connection to it
function get_user_display_name($name, $status) {
    if ($status === "active") {
        return $name;
    }
    return $name . " (inactive)";
}

function deactivate_user(&$status) {
    $status = "inactive";
}

function send_user_email($email, $message) {
    // Imagine this sends an email
    echo "Sending '$message' to $email\n";
}

// --- The Problem ---
// Any part of the code can break things:
$user_status = "banana"; // Oops! No one stopped this invalid value.

echo get_user_display_name($user_name, $user_status);
// Output: Jegan (inactive)... wait, it's "banana", not "inactive"!
// The function doesn't know what valid statuses are.
```

**What's wrong here:**
- `$user_status` can be set to anything — no validation, no protection
- Functions need to receive every piece of data as parameters — messy when you have 10+ fields
- If you add another user, you need another set of variables (`$user2_name`, `$user2_email`...) — doesn't scale
- No clear ownership: who is responsible for user data?
- No validation at the boundary — bad data enters the system silently and breaks things later

### Good Example: OOP Approach

```php
<?php
// --- OOP: User Management ---

class User
{
    // Properties (data) — bundled INSIDE the class
    private string $name;
    private string $email;
    private string $status;

    // The allowed statuses — defined in ONE place
    private const VALID_STATUSES = ['active', 'inactive', 'suspended'];

    // Constructor — runs when you create a new User object
    public function __construct(string $name, string $email, string $status = 'active')
    {
        $this->name = $name;
        $this->setEmail($email);       // Validated + sanitized!
        $this->setStatus($status); // Validated!
    }

    // Public method — the controlled way to get the display name
    public function getDisplayName(): string
    {
        if ($this->status === 'active') {
            return $this->name;
        }
        return $this->name . " ({$this->status})";
    }

    // Controlled way to set email — validate THEN store
    public function setEmail(string $email): void
    {
        $sanitized = filter_var(trim(strtolower($email)), FILTER_SANITIZE_EMAIL);

        if (!filter_var($sanitized, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException("Invalid email: $email");
        }

        $this->email = $sanitized;
    }

    // Public method — the controlled way to change status
    public function setStatus(string $status): void
    {
        if (!in_array($status, self::VALID_STATUSES)) {
            throw new InvalidArgumentException("Invalid status: $status");
        }
        $this->status = $status;
    }

    // Public method — sending email
    public function sendEmail(string $message): void
    {
        echo "Sending '$message' to {$this->email}\n";
    }
}

// --- Usage ---
$user = new User("Jegan", "jegan@example.com");
echo $user->getDisplayName(); // "Jegan"

$user->setStatus("inactive");
echo $user->getDisplayName(); // "Jegan (inactive)"

$user->setStatus("banana"); // 💥 Throws InvalidArgumentException!
// The class PROTECTS itself from invalid data.

// Creating multiple users is clean:
$user2 = new User("Arun", "arun@example.com");
$user3 = new User("Priya", "priya@example.com", "suspended");
```

**What's better here:**
- Data (`$name`, `$email`, `$status`) is **private** — outside code can't directly touch it
- Status is **validated** inside `setStatus()` — the class protects itself
- Each user is a self-contained **object** — no variable name conflicts
- Everything about a User is in **one place** — easy to find, easy to change

---

## Comparison Table

| Feature | Procedural PHP | Object-Oriented PHP |
|---|---|---|
| **Organization** | Functions and variables scattered across files | Classes bundle related data + behavior together |
| **Data Safety** | Data is "naked" — any code can read/modify it | Data is **encapsulated** — access is controlled via methods |
| **Reusability** | Copy-pasting functions between files | **Inherit** and **extend** classes; use **interfaces** and **traits** |
| **Scalability** | Gets chaotic as the app grows | Stays organized — each class has a clear responsibility |
| **Debugging** | Hard to trace which function changed what | Easy — data can only change through defined methods |
| **Multiple Instances** | Need separate variables for each entity | Create as many objects as you want from one class |
| **Teamwork** | Developers step on each other's code | Clear boundaries — each class is a unit of ownership |

---

## How Laravel Relies on This Concept

Laravel is built **entirely** on OOP. You cannot use Laravel without classes and objects. Here's a glimpse of how the concepts from this file show up:

| OOP Concept | Laravel Example |
|---|---|
| **Classes** | Every Controller, Model, Middleware, Job, Event is a class |
| **Encapsulation** | Model properties are protected; you use methods like `$user->save()` |
| **Objects** | When you write `$user = User::find(1)`, you get back an **object** — an instance of the `User` class |
| **Controlled access** | Form Requests validate data before it ever reaches your controller |

```php
// This is OOP in action — even if it looks simple
$user = User::find(1);         // Object creation (behind the scenes)
$user->name = "Jegan";         // Property access (controlled by Eloquent)
$user->save();                 // Method call (encapsulated DB logic)
```

You'll see this pattern **everywhere** in Laravel. Understanding OOP is not optional — it's the foundation.

---

## Common Mistakes

### 1. "I can do everything with functions, why bother with classes?"
You can — for small scripts. But the moment you have 50+ files, multiple developers, and complex business logic, procedural code becomes untraceable. OOP gives you **structure and boundaries**.

### 2. Thinking OOP means "just use classes"
Writing a class doesn't automatically make your code OOP. If you put all your logic into one massive class with public properties, you've just moved the mess into a class. **Encapsulation** (controlling access) is the key, not just the `class` keyword.

### 3. Over-engineering with OOP too early
Don't create 15 classes for a 20-line script. Use OOP when the problem demands structure. Start simple, refactor to OOP when complexity grows.

### 4. Confusing classes and objects
A **class** is the blueprint (like a recipe). An **object** is the actual thing created from it (like the dish). You can create multiple objects from one class. This is covered in depth in the [next file](02-classes-and-objects.md).

---

## Key Takeaways

- **Procedural PHP** separates data (set in variables) and functions — this leads to global state bugs, code duplication, and scaling problems
- **OOP** bundles data (properties) and behavior (methods) into **classes**, creating a self-contained unit
- A **function** inside a class is called a **method**. A **variable** inside a class is called a **property**. Same things, different names based on where they live
- **Encapsulation** is the core idea: protect data inside the class, expose only what's necessary through methods
- **Validate early:** always validate data when it enters the object (constructor/setter) — once inside, data should always be valid
- The **four pillars** of OOP are: Encapsulation, Inheritance, Polymorphism, Abstraction
- **Laravel is 100% OOP** — Controllers, Models, Middleware, everything is a class
- Don't just "use classes" — understand **why** they exist: to create boundaries, protect data, and make code maintainable
- OOP isn't about writing more code — it's about writing **organized, scalable, debuggable** code

---

> **Next up:** [02 - Classes and Objects](02-classes-and-objects.md) — We'll dive deep into what classes and objects actually are, how they work in memory, and how PHP creates them.
