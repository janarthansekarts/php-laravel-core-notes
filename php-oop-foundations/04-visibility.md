# Visibility (public, protected, private)

> **Prerequisites:** [03 - Properties and Methods](03-properties-and-methods.md)
> **Next:** [05 - Constructors and Destructors](05-constructors-and-destructors.md)

---

## The Problem / Why This Exists

In [file 03](03-properties-and-methods.md), we kept writing `private` and `public` on properties and methods. We said "private means hidden, public means accessible" — but we never fully explained:

- **Why** do we hide things?
- **When** should something be public vs private?
- What is `protected` and when does it matter?
- What happens if you choose the wrong visibility?

Visibility is how PHP enforces **encapsulation** — the first pillar of OOP from [file 01](01-why-oop-exists.md). Without visibility modifiers, classes would be just as exposed as procedural code — anyone could change anything from anywhere.

---

## What It Introduces

PHP has **three visibility modifiers** (also called **access modifiers**):

| Modifier | Who Can Access | Symbol in Diagrams |
|---|---|---|
| `public` | **Anyone** — code inside the class, child classes, AND outside code | `+` |
| `protected` | **Family only** — code inside the class AND child classes | `#` |
| `private` | **Self only** — ONLY code inside the same class | `-` |

These apply to **both** properties and methods.

```
┌───────────────────────────────────────────────────┐
│                    Class: User                    │
│                                                   │
│  ─ private string $password      ← ONLY this     │
│  # protected string $role         ← This class   │
│  + public string $name            ← Everyone     │
│                                    class can see  │
│                                   + child classes │
│                                    can see        │
│                                                   │
│  ─ private function hashPw()     ← Only this     │
│  # protected function getRole()   ← This + kids  │
│  + public function getName()      ← Everyone     │
└───────────────────────────────────────────────────┘
```

---

## Real-World Analogy

Think of a **Hospital**:

### `public` = The Waiting Room
Anyone can enter — patients, visitors, delivery drivers. It's the area the hospital **exposes** to the outside world. In code: these are the methods and properties you **want** outside code to use.

### `protected` = Staff-Only Areas
Only hospital employees (and trainees learning from them) can access these. A nurse from the **Cardiology Department** (child class) can access tools from the **General Medical Department** (parent class). In code: child classes can access protected members from parent classes.

### `private` = The OR (Operating Room)
Only the specific surgical team performing THIS surgery can enter. Not even other doctors from the same hospital. In code: only the exact class that declared it can use it — not even child classes.

---

## How PHP Implements This

### `public` — Anyone Can Access

```php
class User
{
    public string $name;  // Anyone can read and write

    public function greet(): string
    {
        return "Hello, I'm {$this->name}";
    }
}

$user = new User();
$user->name = "Jegan";           // ✅ Outside code can set it directly
echo $user->name;                // ✅ Outside code can read it directly
echo $user->greet();             // ✅ Outside code can call it
```

**When to use `public`:**
- Methods that form the class's **external API** — what outside code should call
- Properties: **rarely**. Use public only for simple data objects (DTOs) or when you genuinely don't need any control

### `private` — Only This Class

```php
class User
{
    private string $password;  // ONLY User class can access

    public function __construct(string $name, string $password)
    {
        $this->password = $this->hashPassword($password);
    }

    // Private method — only used internally by this class
    private function hashPassword(string $plain): string
    {
        return password_hash($plain, PASSWORD_DEFAULT);
    }

    // Public method — the controlled way to verify password
    public function verifyPassword(string $attempt): bool
    {
        return password_verify($attempt, $this->password);
    }
}

$user = new User("Jegan", "secret123");

// ✅ Outside code uses the public API
$user->verifyPassword("secret123");   // true

// ❌ Outside code CANNOT access private members
echo $user->password;                 // Fatal error: Cannot access private property
$user->hashPassword("something");     // Fatal error: Cannot access private method
```

**Why this matters:** The password is **never exposed**. Outside code can't read it, print it, or modify it. The ONLY way to interact with it is through `verifyPassword()` — the controlled public method.

**When to use `private`:**
- Properties that should **never** be accessed from outside (passwords, internal state, cached values)
- Helper methods that are **implementation details** — if you renamed or removed them, outside code shouldn't break
- **Default choice** — start with `private` and only make something less restrictive when you have a reason

### `protected` — This Class + Child Classes

`protected` only matters when you use **inheritance** (covered in detail in [file 08](08-inheritance.md)). Here's a preview:

```php
class User
{
    protected string $role = 'user';  // Child classes can access this

    public function getRole(): string
    {
        return $this->role;
    }
}

// AdminUser EXTENDS User — it inherits everything from User
class AdminUser extends User
{
    public function promote(): void
    {
        // ✅ Can access $role because it's PROTECTED (family access)
        $this->role = 'admin';
    }
}

class Guest
{
    public function tryAccess(User $user): void
    {
        // ❌ Cannot access $role — Guest is NOT in User's family
        echo $user->role;  // Fatal error: Cannot access protected property
    }
}

$admin = new AdminUser();
$admin->promote();
echo $admin->getRole();   // "admin"

// But from outside:
echo $admin->role;         // ❌ Fatal error — protected!
```

**When to use `protected`:**
- Properties and methods that child classes need to access or override
- When you're designing a class that's **meant to be extended**
- **Don't use `protected` by default** — if you're not planning for inheritance, use `private`

---

## The Visibility Spectrum — Restrictive to Permissive

```
Most Restrictive                                    Least Restrictive
      │                                                     │
      ▼                                                     ▼
   private ──────────── protected ──────────── public
   
   Only this class     This class +              Anyone
                       child classes
```

> **Golden rule: Start with `private`. Open up only when needed.** This is called the **Principle of Least Privilege** — expose only what's necessary. You can always make something MORE accessible later, but making a public thing private later might break code that depends on it.

---

## Complete Example: All Three Visibilities

```php
class BankAccount
{
    // ─ PRIVATE: Only this class knows about these
    private string $accountNumber;
    private float $balance;
    private array $transactionLog = [];

    // ─ PRIVATE: Internal helper — no one outside should call this
    private function logTransaction(string $type, float $amount): void
    {
        $this->transactionLog[] = [
            'type' => $type,
            'amount' => $amount,
            'date' => date('Y-m-d H:i:s'),
            'balance_after' => $this->balance,
        ];
    }

    // # PROTECTED: Child classes (like SavingsAccount) can access this
    protected float $interestRate;

    // # PROTECTED: Child classes can customize this behavior
    protected function calculateInterest(): float
    {
        return $this->balance * $this->interestRate;
    }

    // + PUBLIC: The external API — what outside code uses
    public function __construct(string $accountNumber, float $initialDeposit = 0)
    {
        $this->accountNumber = $accountNumber;
        $this->balance = $initialDeposit;
        $this->interestRate = 0.01;  // 1% default

        if ($initialDeposit > 0) {
            $this->logTransaction('deposit', $initialDeposit);
        }
    }

    public function getBalance(): float
    {
        return $this->balance;
    }

    public function deposit(float $amount): void
    {
        if ($amount <= 0) {
            throw new InvalidArgumentException("Deposit must be positive");
        }

        $this->balance += $amount;
        $this->logTransaction('deposit', $amount);
    }

    public function withdraw(float $amount): void
    {
        if ($amount <= 0) {
            throw new InvalidArgumentException("Withdrawal must be positive");
        }
        if ($amount > $this->balance) {
            throw new RuntimeException("Insufficient funds");
        }

        $this->balance -= $amount;
        $this->logTransaction('withdrawal', $amount);
    }
}

// A child class can access protected members:
class SavingsAccount extends BankAccount
{
    public function __construct(string $accountNumber, float $initialDeposit = 0)
    {
        parent::__construct($accountNumber, $initialDeposit);
        $this->interestRate = 0.05;  // ✅ Can access protected property
    }

    public function applyInterest(): void
    {
        $interest = $this->calculateInterest();  // ✅ Can call protected method
        $this->deposit($interest);  // Uses public method to deposit
    }
}
```

```
┌───────────────────────────── BankAccount ──────────────────────────────┐
│                                                                        │
│  PRIVATE (only BankAccount can see):                                  │
│    $accountNumber, $balance, $transactionLog                          │
│    logTransaction()                                                   │
│                                                                        │
│  PROTECTED (BankAccount + child classes can see):                     │
│    $interestRate                                                      │
│    calculateInterest()                                                │
│                                                                        │
│  PUBLIC (everyone can see):                                           │
│    __construct(), getBalance(), deposit(), withdraw()                 │
│                                                                        │
│  ┌──────────────── SavingsAccount (child) ──────────────────┐        │
│  │                                                           │        │
│  │  Can access: ✅ public + ✅ protected                     │        │
│  │  Cannot access: ❌ private ($accountNumber, $balance,     │        │
│  │                     $transactionLog, logTransaction())    │        │
│  │                                                           │        │
│  │  Has its own: applyInterest()                             │        │
│  └───────────────────────────────────────────────────────────┘        │
└────────────────────────────────────────────────────────────────────────┘

Outside code:
  Can access:    ✅ public methods only
  Cannot access: ❌ protected ❌ private
```

---

## How Laravel Relies on This Concept

### Models Use `protected` for Configuration

```php
// Laravel's Eloquent Model uses protected properties as "hooks"
// You override them in your model to configure behavior
class User extends Model
{
    // protected — so Laravel's parent Model class can read them
    // but outside code can't set them directly
    protected $fillable = ['name', 'email', 'password'];
    protected $hidden = ['password', 'remember_token'];
    protected $casts = ['email_verified_at' => 'datetime'];
    protected $table = 'users';

    // If these were private, Laravel's base Model class couldn't read them!
    // If these were public, any code could modify them at runtime — dangerous
    // protected is the perfect middle ground
}
```

### Middleware Uses `public` Methods as the API

```php
class EnsureUserIsAdmin
{
    // public — Laravel must be able to call this from outside
    public function handle(Request $request, Closure $next)
    {
        if (!$request->user()?->isAdmin()) {
            abort(403);
        }

        return $next($request);
    }
}
```

### Services Use `private` for Internal Logic

```php
class PaymentService
{
    // PUBLIC — what controllers call
    public function processPayment(Order $order, string $cardToken): PaymentResult
    {
        $this->validateOrder($order);
        $charge = $this->chargeCard($cardToken, $order->total);
        $this->sendReceipt($order, $charge);

        return $charge;
    }

    // PRIVATE — internal steps no one else should call directly
    private function validateOrder(Order $order): void
    {
        if ($order->items->isEmpty()) {
            throw new InvalidArgumentException("Cannot process empty order");
        }
    }

    private function chargeCard(string $token, float $amount): PaymentResult
    {
        // Talk to Stripe/PayPal API...
    }

    private function sendReceipt(Order $order, PaymentResult $charge): void
    {
        // Send email...
    }
}
```

> **Notice the pattern:** `processPayment()` is the **public API** — it's the only thing a controller should call. The three `private` methods are **implementation details** — if you refactored them, renamed them, or split them differently, outside code wouldn't break because no one depends on them directly.

---

## Deciding Visibility: A Quick Flowchart

```
Should outside code call/access this?
│
├── YES → public
│
└── NO → Will child classes need this?
         │
         ├── YES → protected
         │
         └── NO → private  ← DEFAULT CHOICE
```

---

## Common Mistakes

### 1. Making Everything Public (The "Lazy" Approach)

```php
// ❌ Everything public — no encapsulation at all
class User
{
    public string $name;
    public string $password;
    public string $role;
    public float $salary;
}

$user = new User();
$user->salary = -50000;     // Negative salary? No validation.
$user->role = "supreme_god"; // Invalid role? No one stops it.
echo $user->password;        // Password leaked!
```

> **Rule:** If you make everything public, you haven't gained anything over a plain array. The whole point of classes is **controlled access**.

### 2. Using `protected` When `private` Is Enough

```php
// ❌ Overly permissive — protected "just in case someone extends this"
class UserService
{
    protected UserRepository $repo;  // Why protected? Is anyone extending this?
}

// ✅ Be intentional — private until you KNOW inheritance is needed
class UserService
{
    private UserRepository $repo;
}
```

> **Don't design for hypothetical inheritance.** If no one is extending your class right now, use `private`. You can always change it to `protected` later if needed (following YAGNI — "You Aren't Gonna Need It").

### 3. Trying to Access Private Members from Outside

```php
$user = new User("Jegan", "secret");

// ❌ This causes a Fatal Error — not an Exception you can catch
echo $user->password;  // Fatal error: Cannot access private property

// This isn't a "nice" error with a message — it KILLS the script.
// This is PHP enforcing the access rule at the language level.
```

### 4. Confusing `private` and `protected` in Inheritance

```php
class Animal
{
    private string $dna = "ACGT";       // Private to Animal
    protected string $species;           // Accessible to child classes
}

class Dog extends Animal
{
    public function describe(): string
    {
        echo $this->species;  // ✅ Works — protected, family access
        echo $this->dna;      // ❌ Fatal error — private to Animal only!
    }
}
```

> **Private means private to the EXACT class.** Not even children can access it. If a child class needs it, it should be `protected`.

---

## Key Takeaways

- **`public`** = anyone can access (inside class, child classes, outside code) — use for the class's external API
- **`protected`** = only this class + child classes — use when designing for inheritance
- **`private`** = only this exact class — **default choice** for properties and internal methods
- **Start with `private`, open up only when needed** — Principle of Least Privilege
- **Properties should almost always be `private`** (or `protected` in parent classes meant to be extended) — use getters/setters for controlled access
- **Public methods** form the class's "contract" with the outside world — think of them as the API
- **Private methods** are implementation details — you can refactor them freely without breaking outside code
- **Laravel uses `protected`** for model configuration (`$fillable`, `$hidden`, `$casts`) — so the parent `Model` class can read them but outside code can't
- If everything in your class is `public`, you've essentially written an array with extra steps — you've lost the benefits of encapsulation
- Visibility errors are **fatal errors** in PHP — they kill the script, not throw catchable exceptions

---

> **Next up:** [05 - Constructors and Destructors](05-constructors-and-destructors.md) — We'll dive deep into `__construct()` and `__destruct()` — the methods that run when an object is born and when it dies.
