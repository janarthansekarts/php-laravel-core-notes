# Inheritance (is-a vs has-a Relationships)

> **Prerequisites:** [07-static-properties-and-methods.md](07-static-properties-and-methods.md)
> **Next:** [09-method-overriding-and-parent.md](09-method-overriding-and-parent.md)

---

## The Problem / Why This Exists

Imagine you're building an e-commerce site. You have several types of products:

```php
class Book {
    private string $name;
    private float $price;
    private string $sku;
    private string $author;
    private int $pages;

    public function getSummary(): string {
        return "{$this->name} costs \${$this->price}";
    }
}

class Electronics {
    private string $name;     // ❌ Same as Book
    private float $price;     // ❌ Same as Book
    private string $sku;      // ❌ Same as Book
    private int $voltage;
    private string $warranty;

    public function getSummary(): string {     // ❌ Same as Book
        return "{$this->name} costs \${$this->price}";
    }
}

class Clothing {
    private string $name;     // ❌ Same AGAIN
    private float $price;     // ❌ Same AGAIN
    private string $sku;      // ❌ Same AGAIN
    private string $size;
    private string $material;

    public function getSummary(): string {     // ❌ Same AGAIN
        return "{$this->name} costs \${$this->price}";
    }
}
```

**What's wrong here?**
- `$name`, `$price`, `$sku`, and `getSummary()` are **copied identically** across every class
- If you need to change how pricing works, you update it in **3 places** (and forget one — guaranteed)
- If you add a new product type, you copy-paste everything again
- This violates the **DRY principle** — "Don't Repeat Yourself"

**Inheritance** solves this: define the shared stuff once in a **base class**, and let specific classes **inherit** it while adding their own unique features.

---

## What It Introduces

| Term | Meaning |
|------|---------|
| **`extends`** | The keyword that makes one class inherit from another |
| **Parent class** (base class / superclass) | The class being inherited FROM — contains shared code |
| **Child class** (derived class / subclass) | The class that inherits — gets everything from the parent, plus adds its own |
| **Inheritance** | A child class automatically receives all `public` and `protected` members from its parent |
| **is-a relationship** | The child "is a" type of the parent — `Electronics` **is a** `Product` |
| **has-a relationship** | One object contains another — a `Phone` **has a** `Battery` (composition, covered in [14-composition-over-inheritance.md](14-composition-over-inheritance.md)) |
| **Single inheritance** | PHP allows a class to extend only **one** parent class (unlike some languages) |

---

## Real-World Analogy

### The Smartphone Analogy 📱

Think about a **Smartphone** and a **Phone**.

A Smartphone **is a** Phone. It does *everything* a regular phone does:
- Make calls ✅
- Send texts ✅
- Has a phone number ✅

But it **adds** things a regular phone doesn't have:
- Touch screen
- App store
- Camera
- GPS

You didn't have to re-invent "making calls" when designing the smartphone. You **inherited** that capability from the phone concept, then **extended** it with new features.

```
┌────────────────────────┐
│       Phone            │  ← Parent (base) class
│  - phoneNumber         │
│  - makeCall()          │
│  - sendText()          │
└───────────┬────────────┘
            │ extends (inherits all of the above)
            ▼
┌────────────────────────┐
│     Smartphone         │  ← Child (derived) class
│  - phoneNumber    (inherited)
│  - makeCall()     (inherited)
│  - sendText()     (inherited)
│  - camera         (NEW)
│  - installApp()   (NEW)
│  - takePhoto()    (NEW)
└────────────────────────┘
```

Now compare: a Phone **has a** Battery. The battery isn't a *type of phone* — it's a separate object *used inside* the phone. That's **composition** (has-a), not inheritance (is-a). We'll explore that distinction deeply in [file 14](14-composition-over-inheritance.md).

---

## How PHP Implements This

### The `extends` Keyword

```php
class ChildClass extends ParentClass {
    // ChildClass now has everything ParentClass has
    // plus whatever you define here
}
```

The keyword is **`extends`** — very intentional naming. The child doesn't *replace* the parent. It **extends** (adds to) the parent's capabilities.

### Full Example: Product Hierarchy

```php
class Product {
    // Protected — accessible in this class AND child classes
    // (NOT accessible from outside — see file 04)
    protected string $name;
    protected float $price;
    protected string $sku;

    public function __construct(string $name, float $price, string $sku) {
        $this->name = $name;
        $this->price = $price;

        // Validate at the boundary
        if ($price < 0) {
            throw new \InvalidArgumentException("Price cannot be negative");
        }

        $this->sku = $sku;
    }

    public function getSummary(): string {
        return "{$this->name} (SKU: {$this->sku}) — \${$this->price}";
    }

    public function getName(): string {
        return $this->name;
    }

    public function getPrice(): float {
        return $this->price;
    }

    public function getSku(): string {
        return $this->sku;
    }
}
```

Now the child classes **inherit** all of that and **add** their own properties:

```php
class Electronics extends Product {
    // New property — only Electronics has this
    private int $voltage;
    private string $warranty;

    public function __construct(
        string $name,
        float $price,
        string $sku,
        int $voltage,
        string $warranty
    ) {
        // Call the PARENT constructor to handle name, price, sku
        // (We'll explain parent:: in detail in file 09)
        parent::__construct($name, $price, $sku);

        // Then handle our own properties
        $this->voltage = $voltage;
        $this->warranty = $warranty;
    }

    public function getVoltage(): int {
        return $this->voltage;
    }

    public function getWarranty(): string {
        return $this->warranty;
    }
}

class Book extends Product {
    private string $author;
    private int $pages;

    public function __construct(
        string $name,
        float $price,
        string $sku,
        string $author,
        int $pages
    ) {
        parent::__construct($name, $price, $sku);
        $this->author = $author;
        $this->pages = $pages;
    }

    public function getAuthor(): string {
        return $this->author;
    }

    public function getPages(): int {
        return $this->pages;
    }
}

class Clothing extends Product {
    private string $size;
    private string $material;

    public function __construct(
        string $name,
        float $price,
        string $sku,
        string $size,
        string $material
    ) {
        parent::__construct($name, $price, $sku);
        $this->size = $size;
        $this->material = $material;
    }

    public function getSize(): string {
        return $this->size;
    }

    public function getMaterial(): string {
        return $this->material;
    }
}
```

### Using It

```php
$laptop = new Electronics("MacBook Pro", 2499.99, "ELEC-001", 220, "2 years");
$novel  = new Book("Clean Code", 34.99, "BOOK-042", "Robert C. Martin", 464);
$shirt  = new Clothing("PHP Developer Tee", 29.99, "CLO-100", "L", "Cotton");

// All three have getSummary() — inherited from Product
echo $laptop->getSummary();
// "MacBook Pro (SKU: ELEC-001) — $2499.99"

echo $novel->getSummary();
// "Clean Code (SKU: BOOK-042) — $34.99"

echo $shirt->getSummary();
// "PHP Developer Tee (SKU: CLO-100) — $29.99"

// But each has its own unique methods too
echo $laptop->getVoltage();   // 220
echo $novel->getAuthor();     // "Robert C. Martin"
echo $shirt->getMaterial();   // "Cotton"

// Type checking works with inheritance
var_dump($laptop instanceof Electronics);  // true
var_dump($laptop instanceof Product);      // true ← Electronics IS-A Product
var_dump($laptop instanceof Book);         // false ← Electronics is NOT a Book
```

> **`instanceof` with inheritance:** An `Electronics` object is `instanceof` both `Electronics` AND `Product`, because Electronics **is a** Product. This is how PHP enforces the is-a relationship.

### What EXACTLY Gets Inherited?

```
┌─────────────────────────────────────────────────┐
│  What the child class GETS from the parent:     │
│                                                 │
│  ✅ public properties and methods               │
│  ✅ protected properties and methods            │
│  ❌ private properties and methods (NOT inherited│
│     — they exist in the parent but the child    │
│     CANNOT access them directly)                │
│  ✅ constants                                   │
└─────────────────────────────────────────────────┘
```

This is why we used **`protected`** (not `private`) for `$name`, `$price`, `$sku` in `Product`. If they were `private`, the child classes couldn't access `$this->name` directly — they'd have to use the getters.

```php
class Product {
    private string $name;  // ← If private...
}

class Electronics extends Product {
    public function showName(): void {
        echo $this->name;  // ❌ Fatal Error! Cannot access private property
        echo $this->getName();  // ✅ Works — uses the public getter
    }
}
```

> **Design choice:** Use `protected` when child classes need **direct** access to the property. Use `private` + public getters when you want to **control** how the property is accessed (even by children). Laravel's base `Model` uses `protected` for `$fillable`, `$guarded`, `$table`, etc. because child models need to set these directly.

### PHP's Single Inheritance Rule

Unlike some languages (C++, Python), PHP allows **only one parent class**:

```php
class Electronics extends Product {       // ✅ One parent — allowed
    // ...
}

class SmartTV extends Electronics, Display {  // ❌ Two parents — NOT allowed
    // PHP Fatal error: Class SmartTV cannot extend two classes
}
```

**Why?** Multiple inheritance creates the **"Diamond Problem"** — if both parent classes have a method with the same name, which one does the child use? PHP avoids this ambiguity entirely. If you need behavior from multiple sources, PHP provides **interfaces** ([file 10](10-interfaces.md)) and **traits** ([file 12](12-traits.md)) as alternatives.

```
        ┌─────────┐
        │ Product  │
        └────┬────┘
             │
     ┌───────┼────────┐
     ▼       ▼        ▼
   Book  Electronics  Clothing     ← Each extends ONE parent
```

---

## is-a vs has-a — Know the Difference

This is one of the most important design decisions in OOP: **when to inherit and when to compose**.

| Relationship | Meaning | Keyword | Example |
|-------------|---------|---------|---------|
| **is-a** (Inheritance) | The child IS a type of the parent | `extends` | `Electronics` **is a** `Product` |
| **has-a** (Composition) | One class USES another class inside it | property | `Phone` **has a** `Battery` |

### How to Decide

Ask yourself: **"Is [Child] truly a [Parent]?"**

- "Is an Electronics truly a Product?" → **Yes** → Use inheritance (`extends`)
- "Is a Phone truly a Battery?" → **No** → Use composition (has-a)

```php
// ✅ is-a: Electronics IS a Product
class Electronics extends Product { }

// ✅ has-a: Phone HAS a Battery (Battery is a property, not a parent)
class Phone {
    private Battery $battery;  // composition — Phone CONTAINS a battery

    public function __construct(Battery $battery) {
        $this->battery = $battery;
    }
}
```

> **Common beginner mistake:** Using inheritance just to "reuse code." If the is-a relationship doesn't make logical sense, don't use `extends`. A `User` and a `Logger` might share some methods, but a User is NOT a Logger. Use composition instead. We go deep on this in [file 14](14-composition-over-inheritance.md).

---

## How Laravel Relies on This Concept

### Example 1: Every Model Extends `Model`

```php
// Your model:
class User extends Model {
    protected $fillable = ['name', 'email', 'password'];
}

// Laravel's base Model class gives you (simplified):
// - find(), all(), create(), update(), delete()
// - Relationships: hasMany(), belongsTo()
// - Query scopes, events, serialization
// - 4000+ lines of inherited functionality

// You write 3 lines. You inherit hundreds of methods.
$user = User::find(1);        // Inherited from Model
$user->posts();               // Relationship you defined
$user->delete();              // Inherited from Model
```

### Example 2: Every Controller Extends `Controller`

```php
// app/Http/Controllers/Controller.php (Laravel's base)
class Controller {
    use AuthorizesRequests, ValidatesRequests;
    // Provides authorize(), validate(), and other helpers
}

// Your controller inherits those helpers:
class UserController extends Controller {
    public function store(Request $request): RedirectResponse {
        // validate() comes from the parent Controller
        $validated = $this->validate($request, [
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users',
        ]);

        User::create($validated);
        return redirect('/users');
    }
}
```

### Example 3: Form Requests Extend `FormRequest`

```php
// Laravel's base FormRequest extends Request extends SymfonyRequest
// That's a 3-level inheritance chain!

class StoreUserRequest extends FormRequest {
    // You only define what's UNIQUE to this request
    public function authorize(): bool {
        return true;
    }

    public function rules(): array {
        return [
            'name' => 'required|string|max:255',
            'email' => 'required|email',
        ];
    }
    // Everything else (input handling, header parsing, session access)
    // is inherited from FormRequest → Request → SymfonyRequest
}
```

```
┌──────────────────────┐
│  SymfonyRequest      │  ← Handles raw HTTP (headers, body, method)
└──────────┬───────────┘
           │ extends
┌──────────▼───────────┐
│  Illuminate\Request  │  ← Adds Laravel helpers (input(), user(), session())
└──────────┬───────────┘
           │ extends
┌──────────▼───────────┐
│  FormRequest         │  ← Adds validation (rules(), authorize())
└──────────┬───────────┘
           │ extends
┌──────────▼───────────┐
│  StoreUserRequest    │  ← YOUR class — defines rules for this specific form
└──────────────────────┘
```

---

## Common Mistakes

### Mistake 1: Inheriting When the is-a Relationship Doesn't Exist

```php
// ❌ A Stack is NOT an Array — it just USES array-like storage
class Stack extends Array {
    // Now Stack has push(), pop(), but also sort(), slice(), etc.
    // A stack shouldn't support random access or sorting!
}

// ✅ Stack HAS internal array storage (composition)
class Stack {
    private array $items = [];

    public function push(mixed $item): void {
        $this->items[] = $item;
    }

    public function pop(): mixed {
        return array_pop($this->items);
    }
}
```

### Mistake 2: Forgetting to Call `parent::__construct()`

```php
class Electronics extends Product {
    private int $voltage;

    public function __construct(string $name, float $price, string $sku, int $voltage) {
        // ❌ Forgot parent::__construct() — $name, $price, $sku are NEVER set
        $this->voltage = $voltage;
    }
}

$tv = new Electronics("TV", 599.99, "ELEC-002", 120);
echo $tv->getName();  // "" or null — parent properties never initialized! 😱
```

**Fix:** Always call `parent::__construct()` when the parent has a constructor. We'll cover `parent::` in full detail in [file 09](09-method-overriding-and-parent.md).

### Mistake 3: Making Everything `public` Just Because a Child Needs It

```php
// ❌ Don't do this — now ANY code can change the price
class Product {
    public float $price;  // Made public just so Electronics can access it
}

// ✅ Use protected — accessible to children, hidden from outside
class Product {
    protected float $price;  // Children can access, outside code cannot
}
```

### Mistake 4: Deep Inheritance Chains

```php
// ❌ Too deep — fragile and hard to understand
class BaseEntity extends DatabaseRecord 
    extends Serializable 
        extends Cacheable { }
// Every change to a parent risks breaking everything below it
```

> **Rule of thumb:** Keep inheritance chains to **2-3 levels max**. Beyond that, prefer composition and interfaces. Laravel's own inheritance is rarely more than 3 levels deep.

---

## Key Takeaways

1. **`extends` makes a child class inherit** all `public` and `protected` members from a parent class
2. **is-a relationship** — only use inheritance when the child truly IS a type of the parent (`Electronics` is a `Product`)
3. **has-a relationship** — use composition (object as a property) when one class USES another (`Phone` has a `Battery`)
4. **`private` members are NOT accessible** in child classes — use `protected` for properties children need, or `private` + getters for controlled access
5. **PHP enforces single inheritance** — one class can only extend one parent (no Diamond Problem)
6. **Always call `parent::__construct()`** if the parent has a constructor (detailed in [file 09](09-method-overriding-and-parent.md))
7. **`instanceof` checks inheritance** — `$laptop instanceof Product` is `true` because Electronics extends Product
8. **Laravel is built on inheritance** — Models extend `Model`, Controllers extend `Controller`, Requests extend `FormRequest`
9. **Don't inherit just to reuse code** — if the is-a relationship doesn't make sense, use composition instead
10. **Keep inheritance shallow** — 2-3 levels max, prefer composition for deeper hierarchies

---

> **Next up:** [09-method-overriding-and-parent.md](09-method-overriding-and-parent.md) — How child classes customize inherited behavior with method overriding and the `parent::` keyword

## Key Takeaways

Bullet-point summary of what to remember.
