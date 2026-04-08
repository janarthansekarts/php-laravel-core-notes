# Constructors and Destructors

> **Prerequisites:** [04 - Visibility](04-visibility.md)
> **Next:** [06 - The $this Keyword](06-this-keyword.md)

---

## The Problem / Why This Exists

In [file 04](04-visibility.md), we made properties `private` — so outside code can't set them directly. But that creates a question: **if properties are private, how do we give them values when creating the object?**

Consider this broken code:

```php
class Book
{
    private string $title;
    private string $author;
}

$book = new Book();
// ❌ Can't set private properties from outside!
$book->title = "The Great Gatsby";  // Fatal error: Cannot access private property
```

The object is created, but it's **empty and useless**. It has no title, no author — it's in an **incomplete state**. We need a way to set up the object **at the moment of creation**, before anyone can use it.

This is exactly what the **constructor** solves.

---

## What It Introduces

### Magic Methods

Before explaining constructors, you need to know what **magic methods** are.

PHP has a set of special methods that start with **double underscores** (`__`). They are called **magic methods** because PHP calls them **automatically** in response to certain events — you never call them directly.

| Magic Method | When PHP Calls It |
|---|---|
| `__construct()` | When an object is **created** (`new ClassName()`) |
| `__destruct()` | When an object is **destroyed** (script ends or object is unset) |
| `__toString()` | When you try to use an object as a **string** (`echo $object`) |
| `__get()` / `__set()` | When you access an **undefined or inaccessible** property |
| `__call()` | When you call an **undefined or inaccessible** method |

> **Why the `__` prefix?** It's a naming convention that tells you: "Don't call this yourself — PHP calls it for you." The double underscore signals "this is a magic method." You should **never** name your own methods with `__` prefix — that namespace is reserved for PHP.

We'll focus on `__construct()` and `__destruct()` here. The others will appear in later files.

---

## Real-World Analogy

### Constructor = The Assembly Line

When you order a new car from a factory, the **assembly line** requires you to specify the engine type and color **at the moment of creation**. The car isn't "half-built then finished later" — it comes off the line ready to use.

Similarly, `__construct()` runs the moment you call `new` — it ensures the object is **complete and valid** from birth.

### Destructor = The Cleanup Crew

When a building is demolished, a **cleanup crew** comes in to shut off the water, disconnect electricity, and remove hazardous materials. They don't build anything — they make sure everything is properly **closed and cleaned up**.

Similarly, `__destruct()` runs when an object is being removed from memory — it handles cleanup like closing database connections or file handles.

---

## How PHP Implements This

### `__construct()` — The Constructor

```php
class Book
{
    private string $title;
    private string $author;
    private int $pages;
    private string $createdAt;

    // __construct() runs AUTOMATICALLY when you call new Book(...)
    public function __construct(string $title, string $author, int $pages)
    {
        // Validate at the boundary (from file 01's principle)
        if ($pages <= 0) {
            throw new InvalidArgumentException("Pages must be positive, got: $pages");
        }

        // Set properties — this is the ONLY time private properties get their initial values
        $this->title = $title;
        $this->author = $author;
        $this->pages = $pages;
        $this->createdAt = date('Y-m-d H:i:s');  // Calculated at creation time
    }

    public function getDetails(): string
    {
        return "{$this->title} by {$this->author} ({$this->pages} pages)";
    }
}
```

**Using it:**

```php
// When you write this:
$book = new Book("The Great Gatsby", "F. Scott Fitzgerald", 180);

// PHP does this internally:
// 1. Allocates memory for a new Book object (from file 02)
// 2. Calls __construct("The Great Gatsby", "F. Scott Fitzgerald", 180)
// 3. Constructor sets all properties
// 4. Returns the fully initialized object to $book

echo $book->getDetails();
// "The Great Gatsby by F. Scott Fitzgerald (180 pages)"
```

**Why this matters:** You **cannot** create a Book without a title, author, and pages. The constructor **forces** you to provide them. The object is never incomplete.

```php
// ❌ Missing required arguments — PHP throws an error
$book = new Book();  // ArgumentCountError: 3 arguments required, 0 given

// ❌ Invalid data — constructor validates and rejects
$book = new Book("Title", "Author", -5);  // InvalidArgumentException: Pages must be positive
```

### Bad Example: Without a Constructor

```php
// ❌ No constructor — object is created incomplete
class User
{
    private string $name;
    private string $email;

    // No __construct() — properties are declared but never set!

    public function getDisplayName(): string
    {
        return $this->name;  // ❌ Error! $name was never initialized
    }
}

$user = new User();
echo $user->getDisplayName();
// Fatal error: Typed property User::$name must not be accessed before initialization
```

Without a constructor, typed properties are **declared but uninitialized**. Accessing them causes a fatal error. The constructor ensures every property has a value from the start.

### Constructor with Default Parameters

```php
class User
{
    private string $name;
    private string $email;
    private string $role;
    private bool $isActive;

    public function __construct(
        string $name,
        string $email,
        string $role = 'user',      // Default value
        bool $isActive = true        // Default value
    ) {
        $this->name = $name;
        $this->email = $email;
        $this->role = $role;
        $this->isActive = $isActive;
    }
}

// Various ways to create User objects:
$user1 = new User("Jegan", "jegan@example.com");
// role = 'user' (default), isActive = true (default)

$user2 = new User("Arun", "arun@example.com", "admin");
// role = 'admin', isActive = true (default)

$user3 = new User("Priya", "priya@example.com", "editor", false);
// role = 'editor', isActive = false
```

### Constructor Promotion (Recap from File 03)

In [file 03](03-properties-and-methods.md), we learned constructor promotion. Here's a side-by-side comparison:

```php
// ❌ Traditional — repetitive (declare + assign)
class User
{
    private string $name;
    private string $email;
    private string $role;

    public function __construct(string $name, string $email, string $role = 'user')
    {
        $this->name = $name;
        $this->email = $email;
        $this->role = $role;
    }
}

// ✅ Constructor promotion (PHP 8.0+) — concise
class User
{
    public function __construct(
        private string $name,
        private string $email,
        private string $role = 'user',
    ) {
        // Properties are declared AND assigned automatically
        // You can still add validation here:
        // if (empty($name)) throw new InvalidArgumentException("Name required");
    }
}
```

> **Both versions produce identical results.** Constructor promotion is syntactic sugar — it reduces boilerplate but does the exact same thing. Use it for simple classes. Use the traditional form when you need complex validation logic in the constructor.

### Named Arguments (PHP 8.0+)

When a constructor has many parameters, **named arguments** make code more readable:

```php
class User
{
    public function __construct(
        private string $name,
        private string $email,
        private string $role = 'user',
        private bool $isActive = true,
        private ?string $avatar = null,
    ) {}
}

// WITHOUT named arguments — what does 'true' mean? What does null mean?
$user = new User("Jegan", "jegan@example.com", "admin", true, null);

// WITH named arguments — crystal clear
$user = new User(
    name: "Jegan",
    email: "jegan@example.com",
    role: "admin",
    isActive: true,
    avatar: null,
);

// You can even skip defaults and only set what you need:
$user = new User(
    name: "Jegan",
    email: "jegan@example.com",
    avatar: "/img/jegan.jpg",  // Skipped role and isActive — defaults apply
);
```

---

## `__destruct()` — The Destructor

The destructor runs **automatically** when:
1. The script finishes executing
2. The object is explicitly unset (`unset($object)`)
3. The object goes out of scope (e.g., a function ends and its local objects are cleaned up)
4. There are no more references to the object

```php
class DatabaseConnection
{
    private $connection;

    public function __construct(string $host, string $database)
    {
        echo "Opening connection to $database...\n";
        $this->connection = new PDO("mysql:host=$host;dbname=$database", 'user', 'pass');
    }

    public function query(string $sql): array
    {
        return $this->connection->query($sql)->fetchAll();
    }

    // Destructor — runs automatically when the object is destroyed
    public function __destruct()
    {
        echo "Closing database connection...\n";
        $this->connection = null;  // Close the PDO connection
    }
}

// When this function runs:
function fetchUsers(): array
{
    $db = new DatabaseConnection('localhost', 'myapp');
    // Output: "Opening connection to myapp..."

    $users = $db->query("SELECT * FROM users");
    return $users;

    // Function ends → $db goes out of scope → __destruct() runs automatically
    // Output: "Closing database connection..."
}
```

### When to Use Destructors

| Use Case | Example |
|---|---|
| Close database connections | `$this->connection = null` |
| Close file handles | `fclose($this->fileHandle)` |
| Release locks | `$this->lock->release()` |
| Flush buffered data | `$this->logger->flush()` |
| Temporary file cleanup | `unlink($this->tempFile)` |

### Another Example: File Logger

The database example above uses PDO. Here's a more visual example using file handles — you can actually run this and see the destructor trigger:

```php
class FileLogger
{
    private $handle;  // The file resource (an open file pointer)

    public function __construct(string $filename)
    {
        // fopen() opens a file — 'a' means "append" (add to the end, don't overwrite)
        $this->handle = fopen($filename, 'a');

        if ($this->handle === false) {
            throw new RuntimeException("Could not open file: $filename");
        }

        echo "File opened!\n";
    }

    public function log(string $message): void
    {
        // fwrite() writes text to the open file
        $timestamp = date('Y-m-d H:i:s');
        fwrite($this->handle, "[$timestamp] $message\n");
    }

    // Destructor — PHP calls this automatically when the object is destroyed
    public function __destruct()
    {
        // fclose() releases the file handle so other processes can access the file
        if ($this->handle) {
            fclose($this->handle);
            echo "File closed safely.\n";
        }
    }
}

$logger = new FileLogger('app.log');
$logger->log("User logged in.");
$logger->log("User viewed dashboard.");
// Script ends here → __destruct() triggers automatically
// Output: "File closed safely."

// If we DIDN'T close the file, other processes might not be able to read/write it
// The destructor guarantees cleanup happens even if we forget
```

> **Why this matters:** Without the destructor, you'd have to remember to call `fclose()` manually every time you're done with the logger. If an exception happens mid-script, or if you forget, the file handle stays open. The destructor acts as a **safety net** — it ensures cleanup happens no matter how the object's life ends.

> **In practice, destructors are rarely used** in modern PHP — especially in Laravel. PHP automatically closes connections and frees memory when a script ends. Laravel's Service Container manages object lifecycles for you. But understanding destructors helps you grasp the full object lifecycle.

---

## How Laravel Relies on This Concept

### Dependency Injection via Constructor

This is the **#1 use of constructors in Laravel** — and one of the most important patterns you'll learn:

```php
class UserController extends Controller
{
    // Laravel's Service Container reads the constructor,
    // sees the type hints, and automatically creates and injects the dependencies
    public function __construct(
        private UserService $userService,
        private MailService $mailer,
    ) {}

    public function store(Request $request): JsonResponse
    {
        $user = $this->userService->create($request->validated());
        $this->mailer->sendWelcome($user);

        return response()->json($user, 201);
    }
}
```

> **How does this work?** Laravel sees `UserService` in the constructor type hint. It checks its Service Container, finds (or creates) a `UserService` instance, and passes it in — all automatically. You never call `new UserController(...)` yourself. We'll cover this deeply in [15-dependency-injection-pure-php](15-dependency-injection-pure-php.md) and Section 2.

### Middleware Constructor

```php
class RateLimiter
{
    public function __construct(
        private CacheInterface $cache,  // Injected by Laravel
    ) {}

    public function handle(Request $request, Closure $next)
    {
        $key = $request->ip();
        $attempts = $this->cache->get($key, 0);

        if ($attempts >= 60) {
            abort(429, 'Too many requests');
        }

        $this->cache->put($key, $attempts + 1, 60);

        return $next($request);
    }
}
```

### Model Constructor (Rarely Overridden)

```php
// Laravel's base Model has its own __construct() that sets up the model
// If you override it, you MUST call parent::__construct()
class User extends Model
{
    public function __construct(array $attributes = [])
    {
        parent::__construct($attributes);  // ⚠️ ALWAYS call parent!
        // Your custom setup here...
    }
}

// parent:: is covered in file 09, but the key point:
// If a parent class has a constructor, the child MUST call it
// or the parent's setup never happens
```

---

## Common Mistakes

### 1. Forgetting to Call `parent::__construct()` in Child Classes

```php
class Animal
{
    private string $type;

    public function __construct(string $type)
    {
        $this->type = $type;
    }
}

class Dog extends Animal
{
    private string $breed;

    // ❌ WRONG — parent's constructor never runs, $type is never set
    public function __construct(string $breed)
    {
        $this->breed = $breed;
    }

    // ✅ RIGHT — call parent's constructor first
    public function __construct(string $breed)
    {
        parent::__construct('dog');  // Set up the parent
        $this->breed = $breed;       // Then set up the child
    }
}
```

### 2. Doing Too Much in the Constructor

```php
// ❌ Constructor does HTTP requests, file writes, emails — too much!
class Report
{
    public function __construct(private int $userId)
    {
        $data = file_get_contents("https://api.example.com/users/$userId");
        $parsed = json_decode($data, true);
        file_put_contents("/reports/user_{$userId}.csv", $this->generateCSV($parsed));
        mail("admin@example.com", "Report generated", "...");
    }
}

// ✅ Constructor should only INITIALIZE — do work in methods
class Report
{
    public function __construct(
        private int $userId,
        private ApiClient $api,
    ) {}

    public function generate(): string
    {
        $data = $this->api->getUser($this->userId);
        return $this->generateCSV($data);
    }

    public function save(string $path): void { /* ... */ }
    public function notify(string $email): void { /* ... */ }
}
```

> **Rule of thumb:** A constructor should **initialize** (set properties, validate inputs). It should NOT **execute** (make API calls, write files, send emails). Initialization is instant and predictable. Execution has side effects and can fail.

### 3. Trying to Return a Value from the Constructor

```php
// ❌ WRONG — constructors CANNOT return values
class User
{
    public function __construct(string $name)
    {
        return "User created!";  // This does NOTHING. PHP ignores it.
    }
}

$result = new User("Jegan");
// $result is a User object, NOT "User created!"
// new ALWAYS returns the object, regardless of what __construct returns
```

### 4. Calling `__construct()` Directly

```php
// ❌ WRONG — never call __construct() yourself
$user = new User("Jegan");
$user->__construct("Arun");  // Technically runs, but WRONG

// This re-runs the constructor on an existing object — confusing and error-prone
// If you need to "reset" an object, create a new one instead:
$user = new User("Arun");
```

---

## Key Takeaways

- **`__construct()`** is a magic method that PHP calls **automatically** when you use `new` — never call it yourself
- **Magic methods** start with `__` (double underscore) — PHP reserves this prefix for special automatic behavior
- The constructor's job is to **initialize** the object — set properties, validate inputs, ensure the object is complete
- **Objects should never be in an incomplete state** — force required data through constructor parameters
- Use **default parameter values** for optional properties (put them last in the parameter list)
- **Constructor promotion** (PHP 8.0+) declares and assigns properties in one line — use for simple classes
- **Named arguments** (PHP 8.0+) make constructors with many parameters readable
- **`__destruct()`** runs automatically when the object is destroyed — used for cleanup (closing connections, files)
- **Destructors are rare** in modern Laravel — the framework manages object lifecycles for you
- **Don't do heavy work in constructors** — initialize only, execute in methods
- **Always call `parent::__construct()`** when overriding a constructor in a child class
- In **Laravel**, constructors are used primarily for **Dependency Injection** — the Service Container reads type hints and auto-injects dependencies

---

> **Next up:** [06 - The $this Keyword](06-this-keyword.md) — We've been using `$this` since file 01 without fully explaining it. Let's fix that.

## Key Takeaways

Bullet-point summary of what to remember.
