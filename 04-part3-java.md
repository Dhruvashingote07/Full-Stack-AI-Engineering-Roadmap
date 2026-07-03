# Part 3: Java — From Zero to Expert

---

## Chapter 1: Java Fundamentals

### Introduction

Java is a class-based, object-oriented programming language designed for portability, performance, and security. Created by James Gosling at Sun Microsystems in 1995, Java follows the philosophy of **"Write Once, Run Anywhere" (WORA)** through its bytecode execution model. It powers over 3 billion devices worldwide—from Android phones to enterprise servers, from embedded systems to cloud-native microservices.

This chapter lays the absolute foundation: how Java works under the hood, its type system, control flow, arrays, strings, and the complete lifecycle of a Java program.

### Why It Matters

Understanding Java fundamentals is non-negotiable. Every framework (Spring, Hibernate, Quarkus), every design pattern, every optimization technique builds on these basics. A weak foundation leads to subtle bugs—autoboxing overhead, integer overflow, string concatenation in loops, misunderstanding of reference types. Mastery of fundamentals separates a "code writer" from an "engineer."

### Real World Analogy

Think of Java as a **multilingual play**: you write the script (`.java` source), it gets translated into a universal stage language (`.class` bytecode), and any theater with the right machinery (JVM) can perform it identically. The playwright doesn't need to know the specifics of each theater's acoustics, lighting, or stage size—that's the JVM's job.

### Theory

#### History Timeline

| Year | Event |
|------|-------|
| 1991 | The Green Project (Oak language) begins |
| 1995 | Java 1.0a2 publicly released |
| 1998 | Java 2 (J2SE 1.2) with Collections Framework |
| 2004 | Java 5: Generics, Enums, Annotations, Autoboxing |
| 2011 | Oracle acquires Sun; Java 7 released |
| 2014 | Java 8: Lambdas, Streams, Optional, new Date/Time API |
| 2017 | Java 9: Modules (JPMS), JShell |
| 2018 | Java 11 (LTS): HTTP Client, var for lambdas |
| 2021 | Java 17 (LTS): Sealed classes, Records, Pattern Matching |
| 2024+ | Java 21 (LTS): Virtual Threads, Record Patterns, Pattern Matching for switch |

#### JDK vs JRE vs JVM

```
+---------------------------+
|         JDK (Kit)         |
|  +---------------------+  |
|  |   JRE (Runtime Env) |  |
|  |  +---------------+  |  |
|  |  |     JVM       |  |  |
|  |  +---------------+  |  |
|  |  + Core Libraries |  |  |
|  +---------------------+  |
|  + javac, jar, javadoc   |
+---------------------------+
```

- **JDK (Java Development Kit)**: Full kit for developers — compiler (`javac`), debugger (`jdb`), archiver (`jar`), documentation tool (`javadoc`), and the JRE.
- **JRE (Java Runtime Environment)**: JVM + core libraries + deployment tools. Needed to *run* Java programs.
- **JVM (Java Virtual Machine)**: The engine that executes bytecode. Platform-specific, memory-managed, security-enforced.

#### Platform Independence

```
Source (.java) --javac--> Bytecode (.class) --JVM--> Native Machine Code
```

The JVM abstracts away the underlying OS and hardware. The same `.class` file runs on Windows, Linux, macOS, or any system with a JVM implementation.

#### Variables and Data Types

**Primitive Types** (stored directly in stack memory):

| Type | Size | Range | Default |
|------|------|-------|---------|
| `byte` | 8 bits | -128 to 127 | 0 |
| `short` | 16 bits | -32,768 to 32,767 | 0 |
| `int` | 32 bits | -2^31 to 2^31-1 | 0 |
| `long` | 64 bits | -2^63 to 2^63-1 | 0L |
| `float` | 32 bits | +-3.4E-38 to +-3.4E+38 | 0.0f |
| `double` | 64 bits | +-1.7E-308 to +-1.7E+308 | 0.0d |
| `char` | 16 bits | 0 to 65,535 (Unicode) | '\\u0000' |
| `boolean` | JVM-dependent | true or false | false |

**Reference Types** (stored in heap, reference on stack):
- Classes (including String)
- Interfaces
- Arrays
- Enums

#### Operators

| Category | Operators |
|----------|-----------|
| Arithmetic | `+`, `-`, `*`, `/`, `%` |
| Relational | `==`, `!=`, `<`, `>`, `<=`, `>=` |
| Logical | `&&`, `||`, `!` |
| Bitwise | `&`, `|`, `^`, `~`, `<<`, `>>`, `>>>` |
| Assignment | `=`, `+=`, `-=`, `*=`, `/=`, `%=` |
| Unary | `++`, `--`, `+`, `-`, `!` |
| Ternary | `? :` |
| instanceof | `obj instanceof Type` |

#### Control Statements

```java
// if-else
if (score >= 90) {
    grade = 'A';
} else if (score >= 80) {
    grade = 'B';
} else {
    grade = 'F';
}

// switch (Java 14+ arrow syntax)
switch (day) {
    case MONDAY, TUESDAY -> System.out.println("Work day");
    case SATURDAY, SUNDAY -> System.out.println("Weekend");
    default -> System.out.println("Midweek");
}

// switch (expression, Java 14+)
String result = switch (day) {
    case MONDAY, TUESDAY -> "Work";
    case SATURDAY, SUNDAY -> "Weekend";
    default -> "Midweek";
};

// for loop
for (int i = 0; i < 10; i++) {
    System.out.println(i);
}

// enhanced for-each
for (String item : items) {
    System.out.println(item);
}

// while
while (condition) {
    // body
}

// do-while
do {
    // executes at least once
} while (condition);
```

#### Methods

```java
// Method structure
[access_modifier] [static] [final] [synchronized] returnType methodName(ParameterType param1, ...) [throws ExceptionType] {
    // body
    return value;
}

// Variable arity (varargs)
public void printNumbers(int... numbers) {
    for (int n : numbers) {
        System.out.println(n);
    }
}
```

#### Arrays

```java
// Declaration and initialization
int[] numbers = new int[5];
int[] primes = {2, 3, 5, 7, 11};
String[][] matrix = new String[3][4];

// Multi-dimensional (jagged arrays)
int[][] jagged = new int[3][];
jagged[0] = new int[5];
jagged[1] = new int[2];
jagged[2] = new int[7];
```

#### Strings

```java
// String pool
String s1 = "hello";                    // goes to string pool
String s2 = "hello";                    // same reference as s1
String s3 = new String("hello");       // heap object (avoid this)
String s4 = s3.intern();                // returns pooled reference

// String methods
s.length();            // 5
s.charAt(0);           // 'h'
s.substring(1, 4);     // "ell"
s.indexOf('l');        // 2
s.replace('l', 'x');   // "hexxo"
s.toUpperCase();       // "HELLO"
s.equals("hello");     // true (content comparison)

// StringBuilder (mutable, for concatenation in loops)
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 100; i++) {
    sb.append(i).append(", ");
}
String result = sb.toString();
```

> **WARNING**: Never use `+` or `+=` for string concatenation inside loops. Each operation creates a new `StringBuilder` and `String`, yielding O(n^2) complexity. Use `StringBuilder` or `StringBuffer` instead.

#### Type Casting

```java
// Widening (implicit) -- safe
int i = 100;
long l = i;        // int to long
double d = i;      // int to double

// Narrowing (explicit) -- potential data loss
double d = 100.99;
int i = (int) d;   // 100 (truncated)

// Autoboxing / Unboxing (Java 5+)
Integer wrapper = 42;          // autoboxing: int to Integer
int primitive = wrapper;       // unboxing: Integer to int

// Pitfall: null unboxing
Integer n = null;
int x = n;  // NullPointerException at runtime!
```

> ⚠️ **Warning:** Autoboxing and unboxing in hot loops creates hidden object allocations that trigger GC pressure. For performance-critical code, prefer primitive collections (e.g., `IntArrayList` from Eclipse Collections) over `ArrayList<Integer>`.

#### Java Program Lifecycle

1. **Edit**: Write source code in `.java` files
2. **Compile**: `javac` translates source to bytecode (`.class` files)
3. **Load**: ClassLoader loads `.class` files into the JVM
4. **Verify**: Bytecode Verifier checks for security violations
5. **Execute**: Interpreter executes; hot methods get JIT-compiled to native code
6. **Native**: JIT-compiled code runs directly on the CPU

### Internal Working

**JVM Architecture (simplified):**

```
+-------------------------------------------+
|         Runtime Data Areas                |
|  +----------+ +----------+ +-----------+  |
|  |  Heap    | |  Stack   | | Metaspace |  |
|  | (Objects)| | (Frames) | | (Classes) |  |
|  +----------+ +----------+ +-----------+  |
|  +----------+ +----------+                |
|  |PC Regstr | |Native Stk|                |
|  +----------+ +----------+                |
+-------------------+-----------------------+
                    |
+-------------------v-----------------------+
|         Execution Engine                  |
|  Interpreter | JIT Compiler | GC          |
+-------------------------------------------+
```

### Step-by-Step: Your First Java Program

```java
// Step 1: Write source (HelloWorld.java)
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, Java!");
    }
}
```

```
Step 2: Compile
> javac HelloWorld.java
Produces HelloWorld.class

Step 3: Run
> java HelloWorld
Output: Hello, Java!
```

### Common Mistakes

| Mistake | Wrong | Correct |
|---------|-------|---------|
| String equality with `==` | `s1 == s2` | `s1.equals(s2)` |
| Integer overflow | `int x = 2000000000 * 3;` | Use `long` or `Math.multiplyExact()` |
| Float comparison | `0.1 + 0.2 == 0.3` | Use epsilon: `Math.abs(a - b) < 0.0001` |
| Array index out of bounds | `arr[arr.length]` | `arr[arr.length - 1]` |
| Null pointer on array | `int[] arr = null; arr.length` | Check `arr != null` |

### Best Practices

- Use descriptive variable names (`customerCount` not `cc`)
- Declare variables at the point of first use, not at method start
- Prefer `StringBuilder` over `StringBuffer` (no synchronization overhead)
- Use `final` for constants and method parameters that shouldn't be reassigned
- Use `var` (Java 10+) for obvious types: `var list = new ArrayList<String>();`
- Never rely on default values—initialize explicitly
- Use `_` (underscore) for large numeric literals: `1_000_000`

### Interview Questions

1. **Why is Java platform-independent?** — The JVM abstracts the OS. Bytecode runs on any JVM implementation.
2. **What's the difference between `==` and `.equals()`?** — `==` compares references; `.equals()` compares content (overridable).
3. **Is String a primitive or reference type?** — Reference type. But it's treated specially with string pooling and immutability.
4. **What happens if `main` is not declared `static`?** — JVM can't call it without an instance; throws `NoSuchMethodError`.
5. **Explain autoboxing and its pitfalls in loops.** — Auto-wrapping in loops creates many unnecessary `Integer` objects.

### Practical Exercises

1. Write a program that reverses a string without using `StringBuilder.reverse()`.
2. Implement a basic calculator using a switch expression (Java 14+).
3. Create an array of 10 random integers, sort it, and find the median.
4. Write a method that uses varargs to sum any number of integers.

### Mini Project: Number Guessing Game

```java
import java.util.Scanner;
import java.util.concurrent.ThreadLocalRandom;

public class NumberGuessingGame {
    public static void main(String[] args) {
        int secret = ThreadLocalRandom.current().nextInt(1, 101);
        Scanner scanner = new Scanner(System.in);
        int attempts = 0;
        boolean won = false;

        System.out.println("Guess a number between 1 and 100:");

        while (attempts < 10) {
            System.out.print("Attempt " + (attempts + 1) + ": ");
            int guess = Integer.parseInt(scanner.nextLine());
            attempts++;

            if (guess == secret) {
                won = true;
                System.out.println("Correct! You won in " + attempts + " attempts.");
                break;
            }
            System.out.println(guess < secret ? "Too low!" : "Too high!");
        }

        if (!won) {
            System.out.println("Game over! The number was " + secret);
        }
        scanner.close();
    }
}
```

### Revision Notes

- Java: compiled to bytecode, interpreted/JIT-compiled by JVM, WORA
- 8 primitive types; everything else is a reference
- String pool: interning saves memory for literal strings
- Arrays are objects; `length` is a field, not a method
- Autoboxing has performance cost — beware in loops
- Java program lifecycle: Edit, Compile, Load, Verify, Execute

---
## Chapter 2: Object-Oriented Programming

### Introduction

Java is fundamentally object-oriented. Everything except primitives is an object. The four pillars—**Encapsulation, Inheritance, Polymorphism, Abstraction**—are enforced by language constructs like `private`, `extends`, `interface`, and method dispatch mechanisms.

### Why It Matters

Without mastering OOP, you cannot design maintainable systems. OOP enables:
- **Modularity**: Each class is a self-contained unit
- **Reusability**: Inheritance and composition reduce duplication
- **Testability**: Polymorphism enables mocking in tests
- **Extensibility**: Open/Closed Principle through abstraction

### Real World Analogy

Think of a **car manufacturing plant**:
- **Class**: The blueprint for a car model
- **Object**: An actual car built from that blueprint
- **Encapsulation**: The engine is hidden under the hood; you use the steering wheel, not the pistons
- **Inheritance**: SUV "is-a" Car (extends base car with off-road features)
- **Polymorphism**: Different cars implement `drive()` differently (automatic vs manual)
- **Abstraction**: You interact with the `interface` of pedals and steering, not the internal combustion process

### Theory

#### Classes and Objects

```java
// Class definition
public class BankAccount {
    // Fields (state)
    private String accountNumber;
    private double balance;
    private static final double MIN_BALANCE = 100.0;
    private static int totalAccounts = 0;

    // Constructor
    public BankAccount(String accountNumber, double initialDeposit) {
        this.accountNumber = accountNumber;
        this.balance = initialDeposit;
        totalAccounts++;
    }

    // Methods (behavior)
    public void deposit(double amount) {
        if (amount > 0) {
            this.balance += amount;
        }
    }

    public boolean withdraw(double amount) {
        if (amount > 0 && this.balance - amount >= MIN_BALANCE) {
            this.balance -= amount;
            return true;
        }
        return false;
    }

    public double getBalance() {
        return this.balance;
    }

    // Static method
    public static int getTotalAccounts() {
        return totalAccounts;
    }
}

// Usage
BankAccount acc = new BankAccount("123456", 1000.0);
acc.deposit(500);
acc.withdraw(200);
System.out.println(BankAccount.getTotalAccounts()); // 1
```

#### The `this` Keyword

```java
public class Employee {
    private String name;
    private int id;

    public Employee(String name, int id) {
        this.name = name;
        this.id = id;
    }

    // Constructor chaining
    public Employee(String name) {
        this(name, 0);
    }
}
```

#### The `static` Keyword

```java
public class Configuration {
    private static String appName = "MyApp";
    private static int instanceCount = 0;

    static {
        System.out.println("Loading configuration...");
        appName = System.getProperty("app.name", "DefaultApp");
    }

    {
        instanceCount++;
    }

    public static String getAppName() {
        return appName;
    }
}
```

#### Encapsulation

Access modifiers control visibility:

| Modifier | Class | Package | Subclass | World |
|----------|-------|---------|----------|-------|
| `private` | Yes | No | No | No |
| default | Yes | Yes | No | No |
| `protected` | Yes | Yes | Yes | No |
| `public` | Yes | Yes | Yes | Yes |

```java
public class User {
    private String password;

    public void setPassword(String password) {
        if (password.length() < 8) {
            throw new IllegalArgumentException("Password must be 8+ characters");
        }
        this.password = hashPassword(password);
    }

    public boolean authenticate(String input) {
        return this.password.equals(hashPassword(input));
    }

    private String hashPassword(String plain) {
        return BCrypt.withDefaults().hashToString(12, plain.toCharArray());
    }
}
```

#### Inheritance

```java
// Base class
public class Vehicle {
    protected String brand;
    protected int year;

    public Vehicle(String brand, int year) {
        this.brand = brand;
        this.year = year;
    }

    public void start() {
        System.out.println("Vehicle starting...");
    }
}

// Derived class
public class Car extends Vehicle {
    private int doors;

    public Car(String brand, int year, int doors) {
        super(brand, year);
        this.doors = doors;
    }

    @Override
    public void start() {
        super.start();
        System.out.println("Car engine running. Doors: " + doors);
    }

    public void honk() {
        System.out.println("Beep beep!");
    }
}
```

**Key rules:**
- Java supports **single class inheritance** (one parent class)
- `super()` must be the first statement in a constructor
- `final` classes cannot be extended
- `final` methods cannot be overridden

> 💡 **Pro Tip:** Favor composition over inheritance. Inheritance creates tight coupling — changes to the base class can break all subclasses. Use interfaces for contracts and composition (has-a) for code reuse. The GoF Design Patterns book consistently prefers composition.

#### Polymorphism

**Compile-time (Method Overloading):**
```java
public class Calculator {
    public int add(int a, int b) { return a + b; }
    public double add(double a, double b) { return a + b; }
    public int add(int a, int b, int c) { return a + b + c; }
}
```

**Runtime (Method Overriding):**
```java
public class PaymentProcessor {
    public void processPayment(double amount) {
        System.out.println("Processing payment of $" + amount);
    }
}

public class CreditCardProcessor extends PaymentProcessor {
    @Override
    public void processPayment(double amount) {
        System.out.println("Charging credit card: $" + amount);
    }
}

public class PayPalProcessor extends PaymentProcessor {
    @Override
    public void processPayment(double amount) {
        System.out.println("Redirecting to PayPal: $" + amount);
    }
}

// Polymorphic usage
PaymentProcessor processor = getPaymentMethod();
processor.processPayment(100.0);
```

#### Abstraction

**Abstract Classes:**
```java
public abstract class Database {
    protected String connectionString;

    public Database(String connectionString) {
        this.connectionString = connectionString;
    }

    public abstract Connection connect();

    public void disconnect() {
        System.out.println("Closing connection");
    }

    public final DataSet executeQuery(String sql) {
        Connection conn = connect();
        DataSet result = doExecute(conn, sql);
        conn.close();
        return result;
    }

    protected abstract DataSet doExecute(Connection conn, String sql);
}
```

**Interfaces:**
```java
public interface Flyable {
    void fly();

    default void takeOff() {
        System.out.println("Taking off...");
        fly();
    }

    static boolean isAirborne(int altitude) {
        return altitude > 0;
    }
}

// Multiple interface implementation
public class Airplane implements Flyable, Landable {
    @Override
    public void fly() {
        System.out.println("Airplane flying");
    }

    @Override
    public void land() {
        System.out.println("Airplane landing");
    }
}
```

#### Relationships

**Association** (uses-a):
```java
public class Driver {
    public void drive(Car car) {
        car.start();
        car.accelerate();
    }
}
```

**Aggregation** (has-a, weak):
```java
public class Department {
    private List<Employee> employees;

    public Department(List<Employee> employees) {
        this.employees = employees;
    }
}
```

**Composition** (has-a, strong):
```java
public class House {
    private final List<Room> rooms;

    public House() {
        this.rooms = new ArrayList<>();
        this.rooms.add(new Kitchen());
        this.rooms.add(new Bedroom());
    }
}
```

### Internal Working

**How JVM resolves method calls:**

1. **Static binding** (compile-time): `private`, `static`, `final` methods — resolved at compile time
2. **Dynamic binding** (runtime): instance methods — JVM uses vtable (virtual method table) to dispatch

For each class, the JVM builds a vtable with pointers to method implementations. When `obj.method()` is called, the JVM looks up the vtable for the actual runtime type.

### Step-by-Step: Building an Inheritance Hierarchy

```java
// Step 1: Define base class
public class Animal {
    protected String name;
    public Animal(String name) { this.name = name; }
    public void speak() { System.out.println("..."); }
}

// Step 2: Extend with Dog
public class Dog extends Animal {
    public Dog(String name) { super(name); }
    @Override
    public void speak() { System.out.println("Woof!"); }
}

// Step 3: Extend with Cat
public class Cat extends Animal {
    public Cat(String name) { super(name); }
    @Override
    public void speak() { System.out.println("Meow!"); }
}

// Step 4: Polymorphic usage
Animal[] animals = {new Dog("Rex"), new Cat("Whiskers")};
for (Animal a : animals) {
    a.speak();  // Woof! Meow!
}
```

### Common Mistakes

| Mistake | Wrong | Correct |
|---------|-------|---------|
| Using inheritance for "has-a" | `class Car extends Engine` | `class Car { private Engine engine; }` |
| Forgetting `super()` in constructor | Implicit super() fails | Explicitly call `super(args)` |
| Overriding without `@Override` | Typo makes it overload | Always use `@Override` |
| Public fields | `public double balance;` | `private double balance;` with getters/setters |
| Calling overridden method from constructor | Overridable method called in constructor | Avoid calling overridable methods in constructors |

### Best Practices

- **Favor composition over inheritance**
- Keep class hierarchy shallow (max 3-4 levels)
- Make classes `final` by default
- Use interfaces for contracts, abstract classes for partial implementations
- Follow the **Liskov Substitution Principle**
- Use `@Override` on every overriding method
- Minimize mutability — prefer `final` fields

### Interview Questions

1. **What is the diamond problem in Java?** — Java solves the diamond problem for interfaces via explicit disambiguation with `Interface.super.method()`.
2. **Can you override a `private` method?** — No. Private methods are not inherited.
3. **What's the difference between `abstract class` and `interface`?** — Abstract classes can have state and constructors. Interfaces (pre-Java 8) had only abstract methods. Since Java 8, interfaces can have default/static methods.
4. **What is a covariant return type?** — The return type of an overriding method can be a subtype of the overridden method's return type.
5. **Can a `static` method be overridden?** — No, it's hidden, not overridden.
6. **Explain the template method pattern.** — Abstract class defines the skeleton with a final template method and lets subclasses override specific steps.

### Practical Exercises

1. Design a `Shape` hierarchy with `Circle`, `Rectangle`, `Triangle` implementing `area()` and `perimeter()`.
2. Create an `EmailService` interface with `send(Email)` and an `SmtpEmailService` implementation.
3. Implement the Builder pattern for a `Pizza` class (size, crust, toppings).
4. Write a program demonstrating method overloading.

### Mini Project: Employee Management System

```java
import java.util.*;
import java.math.BigDecimal;

public abstract class Employee {
    private final String id;
    private final String name;
    protected BigDecimal baseSalary;

    public Employee(String id, String name, BigDecimal baseSalary) {
        this.id = id;
        this.name = name;
        this.baseSalary = baseSalary;
    }

    public abstract BigDecimal calculateSalary();
    public abstract String getRole();

    @Override
    public String toString() {
        return getRole() + ": " + name + " ($" + calculateSalary() + ")";
    }
}

public class FullTimeEmployee extends Employee {
    private final BigDecimal bonus;

    public FullTimeEmployee(String id, String name, BigDecimal baseSalary, BigDecimal bonus) {
        super(id, name, baseSalary);
        this.bonus = bonus;
    }

    @Override
    public BigDecimal calculateSalary() {
        return baseSalary.add(bonus);
    }

    @Override
    public String getRole() { return "Full-Time"; }
}

public class Contractor extends Employee {
    private final int hoursWorked;
    private final BigDecimal hourlyRate;

    public Contractor(String id, String name, BigDecimal hourlyRate, int hoursWorked) {
        super(id, name, BigDecimal.ZERO);
        this.hourlyRate = hourlyRate;
        this.hoursWorked = hoursWorked;
    }

    @Override
    public BigDecimal calculateSalary() {
        return hourlyRate.multiply(BigDecimal.valueOf(hoursWorked));
    }

    @Override
    public String getRole() { return "Contractor"; }
}

public class PayrollSystem {
    private final List<Employee> employees = new ArrayList<>();

    public void addEmployee(Employee e) { employees.add(e); }

    public void processPayroll() {
        for (Employee e : employees) {
            System.out.println("Paying " + e.getName() + ": $" + e.calculateSalary());
        }
    }

    public static void main(String[] args) {
        PayrollSystem payroll = new PayrollSystem();
        payroll.addEmployee(new FullTimeEmployee("E1", "Alice", new BigDecimal("5000"), new BigDecimal("1000")));
        payroll.addEmployee(new Contractor("E2", "Bob", new BigDecimal("50"), 80));
        payroll.processPayroll();
    }
}
```

### Revision Notes

- **4 pillars**: Encapsulation (data hiding), Inheritance (IS-A), Polymorphism (many forms), Abstraction (hide complexity)
- **`this`**: reference to current instance; used for field disambiguation and constructor chaining
- **`static`**: belongs to class, not instance; no access to `this`
- **`super()`**: calls parent constructor; must be first in constructor
- **Overloading**: same name, different params (compile-time)
- **Overriding**: same signature, different impl (runtime)
- **Abstract class**: can have state and constructors; `abstract` methods must be overridden
- **Interface**: contract; multiple inheritance of behavior; default/static methods since Java 8
- **Composition > Inheritance**: prefer has-a over is-a unless genuine subtype relationship

---
## Chapter 3: Collections Framework

### Introduction

The Java Collections Framework (JCF) is a unified architecture for storing, retrieving, and manipulating groups of objects. It provides interfaces (`List`, `Set`, `Queue`, `Map`) with multiple implementations optimized for different use cases.

### Why It Matters

Choosing the wrong collection causes:
- **Performance degradation**: O(n) instead of O(1) lookups
- **Concurrent modification exceptions**: Using non-thread-safe collections in multi-threaded code
- **Memory bloat**: Inefficient data structures
- **Ordering bugs**: Assuming insertion order with `HashSet`

### Real World Analogy

- **List** (ArrayList): A numbered parking lot where each spot has an index
- **Set** (HashSet): A library's ISBN catalog -- no duplicates, instant lookup
- **Queue** (ArrayDeque): A ticket counter line -- first come, first served
- **Map** (HashMap): A dictionary -- look up a word (key) to get its definition (value)

### Theory

#### Complete Hierarchy

```
                          Iterable
                              |
                         Collection
                        /    |                       List    Set    Queue
                  /  \    /  \     |
           ArrayList  LinkedList  PriorityQueue
           LinkedList              Deque
                                 /                                 ArrayDeque  LinkedList

                         Map (not a Collection)
                        /    |                      HashMap   TreeMap  LinkedHashMap

             Utility: Collections, Arrays
```

#### List Interface

| Feature | ArrayList | LinkedList |
|---------|-----------|------------|
| Internal | Resizable array | Doubly-linked list |
| get(index) | O(1) | O(n) |
| add(E) | O(1) amortized | O(1) |
| add(index, E) | O(n) -- shift elements | O(n) -- traverse to index |
| remove(index) | O(n) | O(n) |
| Memory | Contiguous, less overhead | Per-element node overhead |
| Use case | Random access, iterate | Frequent add/remove at ends |

```java
// ArrayList -- most common List
List<String> list = new ArrayList<>();
list.add("Apple");
list.add("Banana");
list.add(0, "Apricot");     // Shift right: O(n)
String fruit = list.get(1); // O(1)

// LinkedList -- Deque operations
LinkedList<String> deque = new LinkedList<>();
deque.addFirst("First");
deque.addLast("Last");
String first = deque.removeFirst();
```

#### ArrayList Internal Working

```java
// Simplified internal structure
public class ArrayList<E> {
    private static final int DEFAULT_CAPACITY = 10;
    private Object[] elementData;
    private int size;

    public ArrayList() {
        this.elementData = new Object[DEFAULT_CAPACITY];
    }

    public boolean add(E e) {
        if (size == elementData.length) {
            grow();  // 1.5x growth
        }
        elementData[size++] = e;
        return true;
    }

    private void grow() {
        int newCapacity = elementData.length + (elementData.length >> 1);
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    public E get(int index) {
        Objects.checkIndex(index, size);
        return (E) elementData[index];
    }
}
```

> **WARNING**: Setting initial capacity matters. `new ArrayList<>(1_000_000)` avoids repeated resizing for large lists.

#### HashMap Internal Working (CRITICAL)

```java
// Simplified internal structure
public class HashMap<K, V> {
    static final int DEFAULT_INITIAL_CAPACITY = 16;
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    static final int TREEIFY_THRESHOLD = 8;
    static final int UNTREEIFY_THRESHOLD = 6;
    static final int MIN_TREEIFY_CAPACITY = 64;

    transient Node<K,V>[] table;
    int threshold;
}
```

**How `put(key, value)` works:**

1. Compute hash: `hash = key.hashCode() ^ (key.hashCode() >>> 16)`
2. Calculate bucket index: `(n - 1) & hash` (where n = table.length, power of 2)
3. If bucket is empty: create Node and place it
4. If bucket has nodes:
   - Check if same key (by hash and equals) -- replace value
   - If TreeNode -- insert into tree (O(log n))
   - If regular Node chain -- traverse; if found replace else add at end
   - After adding: if chain length >= 8 AND table length >= 64 -- convert to tree
5. After insertion: if `size > threshold` -- resize (double capacity + rehash)

**HashMap performance factors:**

| Factor | Impact |
|--------|--------|
| Initial capacity | Too low -> frequent resize (O(n) each) |
| Load factor | Lower -> more memory, fewer collisions; Higher -> less memory, more collisions |
| hashCode quality | Bad hashCode -> collisions -> chaining -> performance degrades |
| Key immutability | Mutable keys -> lookup fails after mutation |

#### Set Implementations

```java
// HashSet -- backed by HashMap
Set<String> hashSet = new HashSet<>();
hashSet.add("apple");    // O(1)
hashSet.contains("apple"); // O(1)

// LinkedHashSet -- maintains insertion order
Set<String> linkedHashSet = new LinkedHashSet<>();
linkedHashSet.add("banana");
linkedHashSet.add("apple");
// Iteration: banana, apple (insertion order)

// TreeSet -- sorted, backed by TreeMap (Red-Black tree)
Set<String> treeSet = new TreeSet<>();
treeSet.add("banana");
treeSet.add("apple");
treeSet.add("cherry");
// Iteration: apple, banana, cherry (natural order)
```

#### Queue and Deque

```java
// PriorityQueue -- min-heap by default
Queue<Task> taskQueue = new PriorityQueue<>(Comparator.comparing(Task::getPriority));
taskQueue.offer(new Task("Fix bug", 1));
taskQueue.offer(new Task("Write docs", 3));
Task next = taskQueue.poll();  // gets highest priority (lowest number)

// ArrayDeque -- resizable array, faster than LinkedList
Deque<String> stack = new ArrayDeque<>();
stack.push("first");
stack.push("second");
String top = stack.pop();  // "second"

Deque<String> queue = new ArrayDeque<>();
queue.offer("first");
queue.offer("second");
String head = queue.poll();  // "first"
```

#### Collections Utility Class

```java
List<String> list = new ArrayList<>(Arrays.asList("c", "a", "b"));

Collections.sort(list);                              // natural sort
Collections.sort(list, Comparator.reverseOrder());   // reverse sort
Collections.shuffle(list);                           // random shuffle
Collections.reverse(list);                           // reverse order
Collections.binarySearch(list, "b");                 // binary search
Collections.frequency(list, "a");                    // count occurrences
Collections.min(list);                               // minimum
Collections.max(list);                               // maximum
Collections.fill(list, "x");                         // replace all
Collections.unmodifiableList(list);                  // unmodifiable view
Collections.synchronizedList(list);                  // thread-safe wrapper
```

#### Comparable vs Comparator

```java
// Comparable -- natural ordering
public class User implements Comparable<User> {
    private String name;
    private int age;

    @Override
    public int compareTo(User other) {
        return this.name.compareTo(other.name);
    }
}

// Comparator -- custom ordering
Comparator<User> byAge = Comparator.comparingInt(User::getAge);
Comparator<User> byAgeDesc = Comparator.comparingInt(User::getAge).reversed();
Comparator<User> byNameThenAge = Comparator.comparing(User::getName)
    .thenComparingInt(User::getAge);

users.sort(byAge);
```

#### Fail-Fast vs Fail-Safe Iterators

| Feature | Fail-Fast | Fail-Safe |
|---------|-----------|-----------|
| Behavior | Throws ConcurrentModificationException | Works on a snapshot |
| Collections | ArrayList, HashMap, HashSet | ConcurrentHashMap, CopyOnWriteArrayList |
| Mechanism | Tracks modCount | Iterates over a clone |
| Thread-safety | Not safe | Safe for concurrent modification |

```java
// Fail-fast: ConcurrentModificationException
List<String> list = new ArrayList<>(List.of("a", "b", "c"));
for (String s : list) {
    if (s.equals("b")) {
        list.remove(s);  // throws ConcurrentModificationException!
    }
}

// Correct way: use iterator.remove()
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    if (it.next().equals("b")) {
        it.remove();  // safe
    }
}
```

### Internal Working

**HashMap collision resolution (Java 8+):**

```
Bucket 0: null
Bucket 1: Node("John", 25) -> Node("Jane", 30) -> Node("Jack", 28)  // chain
   (treeify if length >= 8 and table.length >= 64)
Bucket 2: TreeNode("Alice", 22) <-> TreeNode("Alex", 35)  // Red-Black Tree
```

### Common Mistakes

| Mistake | Wrong | Correct |
|---------|-------|---------|
| Mutable HashMap keys | Modify key after insertion | Use immutable keys |
| Assuming insertion order | `new HashMap<>()` | `new LinkedHashMap<>()` |
| Forgetting equals/hashCode | Custom class in HashSet | Always override both |
| Using raw types | `List list = new ArrayList()` | `List<String> list = new ArrayList<>()` |
| Concurrent modification | Modifying list during for-each | Use `Iterator.remove()` or `removeIf()` |
| Returning internal list | Getter returns internal list | Return `Collections.unmodifiableList()` |

### Best Practices

- Program to interfaces: `List<String> list = new ArrayList<>()`
- Set initial capacity when you know the size
- Use `EnumMap` for enum keys (fastest Map)
- Use `LinkedHashMap` for LRU cache (`removeEldestEntry`)
- Never modify a collection while iterating
- Use `List.of()`, `Set.of()`, `Map.of()` for small fixed collections (Java 9+)
- Override `equals()` and `hashCode()` together
- Use `computeIfAbsent` for atomic lazy initialization

### Interview Questions

1. **How does HashMap work internally?** -- Array of buckets; hash -> index; collisions handled by linked list or tree (Java 8+); resize at 75% load.
2. **What's the difference between HashMap and ConcurrentHashMap?** -- ConcurrentHashMap uses fine-grained locking; allows concurrent reads.
3. **When does HashMap treeify?** -- When chain length >= 8 AND table length >= 64.
4. **Why is 0.75 the default load factor?** -- Classic trade-off between time and space.
5. **What's the difference between ArrayList and LinkedList?** -- Array vs linked nodes; O(1) get vs O(n); O(n) insert in middle.
6. **Explain WeakHashMap.** -- Entries removed when key is no longer referenced. Uses WeakReference.

### Practical Exercises

1. Count word frequencies in a text file using HashMap.
2. Implement an LRU cache using LinkedHashMap.
3. Use TreeSet to store students sorted by GPA.
4. Demonstrate ConcurrentModificationException and fix it.

### Mini Project: In-Memory Product Catalog

```java
public class ProductCatalog {
    private final Map<String, Product> products = new LinkedHashMap<>();
    private final NavigableMap<String, Product> sortedByName = new TreeMap<>();
    private final NavigableMap<BigDecimal, Set<Product>> byPrice = new TreeMap<>();

    public void addProduct(Product p) {
        products.put(p.sku(), p);
        sortedByName.put(p.name(), p);
        byPrice.computeIfAbsent(p.price(), k -> new HashSet<>()).add(p);
    }

    public Product findBySku(String sku) { return products.get(sku); }

    public List<Product> findByNamePrefix(String prefix) {
        return sortedByName.subMap(prefix, prefix + Character.MAX_VALUE)
            .values().stream().toList();
    }

    public List<Product> findByPriceRange(BigDecimal min, BigDecimal max) {
        return byPrice.subMap(min, true, max, true).values().stream()
            .flatMap(Collection::stream).toList();
    }

    public void removeProduct(String sku) {
        Product p = products.remove(sku);
        if (p != null) {
            sortedByName.remove(p.name());
            byPrice.getOrDefault(p.price(), Collections.emptySet()).remove(p);
        }
    }
}
```

### Revision Notes

- **List**: Ordered, allows duplicates. ArrayList (fast random access), LinkedList (fast add/remove)
- **Set**: No duplicates. HashSet (O(1)), LinkedHashSet (insertion order), TreeSet (sorted, O(log n))
- **Queue**: FIFO. PriorityQueue (heap-based priority), ArrayDeque (fast)
- **Map**: Key-value. HashMap (O(1)), LinkedHashMap (order), TreeMap (sorted), ConcurrentHashMap (thread-safe)
- **HashMap internals**: hash, bucket index, chaining, treeify at 8, resize at 75%, power of 2 capacity
- **Comparable**: natural ordering (`compareTo`)
- **Comparator**: custom ordering (`compare`)
- **Fail-fast**: throws CME on concurrent modification during iteration

---
## Chapter 4: Streams & Lambda

### Introduction

Java 8 introduced **functional programming** constructs into the language. Lambdas and the Stream API enable declarative data processing--express *what* to do, not *how* to do it. Combined, they reduce boilerplate, improve readability, and enable efficient parallel processing.

### Why It Matters

Before Java 8, data processing meant nested for-loops, mutable accumulators, and complex conditional logic. With Streams: declarative, lazy evaluation, built-in parallelism, and immutable operations. 90% of modern Java code uses lambdas and streams.

### Real World Analogy

Think of a **factory assembly line**:
- **Source**: Raw materials arrive (collection)
- **Intermediate operations**: Each station transforms the product (filter, map, sort)
- **Terminal operation**: The product is packaged and shipped (collect, reduce)

The line only runs when the final packaging station is ready (lazy evaluation).

### Theory

#### Functional Interfaces

A functional interface has exactly one abstract method. They can be implemented with lambdas.

| Interface | Signature | Purpose |
|-----------|-----------|---------|
| `Predicate<T>` | T -> boolean | Test a condition |
| `Function<T,R>` | T -> R | Transform input to output |
| `Consumer<T>` | T -> void | Perform an action |
| `Supplier<T>` | () -> T | Provide a value |
| `UnaryOperator<T>` | T -> T | Same type transform |
| `BinaryOperator<T>` | (T,T) -> T | Combine two values |

```java
// Without lambda (anonymous class)
Predicate<String> isEmpty = new Predicate<String>() {
    @Override
    public boolean test(String s) { return s.isEmpty(); }
};

// With lambda
Predicate<String> isEmpty = s -> s.isEmpty();
Predicate<String> isEmpty2 = String::isEmpty;  // method reference

// Composing predicates
Predicate<String> nonEmpty = isEmpty.negate();
Predicate<String> validLength = s -> s.length() >= 3 && s.length() <= 20;
Predicate<String> isValid = nonEmpty.and(validLength);
```

#### Lambda Syntax

```java
// Full syntax: (parameters) -> { body }
(int x, int y) -> { return x + y; }

// Single parameter with type inference
x -> x * 2

// No parameters
() -> System.currentTimeMillis()

// Multiple statements
(String name, int age) -> {
    String formatted = name.toUpperCase() + " (" + age + ")";
    return formatted;
}

// No braces for single expression
(a, b) -> a + b
```

#### Method References

```java
// Static method
Stream.of("a", "b").map(String::toUpperCase)

// Instance method of specific object
stream.forEach(System.out::println)

// Instance method of arbitrary object
stream.map(String::length)

// Constructor
Stream.of("Alice", "Bob").map(User::new)
```

#### Stream Pipeline

```
Source -> 0..N Intermediate Operations (lazy) -> Terminal Operation (eager) -> Result
```

```java
Stream<String> stream = names.stream()
    .filter(s -> s.startsWith("A"))
    .map(s -> s.toUpperCase());
// Nothing printed yet -- pipeline not executed

List<String> result = stream.toList();
// Now operations execute lazily
```

#### Common Stream Operations

```java
// Creating streams
Stream<String> fromList = list.stream();
Stream<String> fromArray = Arrays.stream(array);
Stream<String> ofValues = Stream.of("a", "b", "c");
Stream<String> fromFile = Files.lines(Paths.get("file.txt"));

// filter -- retains elements matching predicate
List<Product> cheap = products.stream()
    .filter(p -> p.price() < 50.0)
    .toList();

// map -- transforms each element
List<String> names = users.stream()
    .map(User::getName)
    .toList();

// flatMap -- flattens nested structures
List<String> allWords = sentences.stream()
    .flatMap(sentence -> Arrays.stream(sentence.split(" ")))
    .toList();

// distinct -- removes duplicates
List<Integer> unique = numbers.stream().distinct().toList();

// sorted
List<User> byAge = users.stream()
    .sorted(Comparator.comparingInt(User::getAge))
    .toList();

// limit & skip -- pagination
List<Product> page = products.stream()
    .skip(page * size)
    .limit(size)
    .toList();

// anyMatch / allMatch / noneMatch
boolean hasAdults = users.stream().anyMatch(u -> u.getAge() >= 18);

// findFirst
Optional<User> firstMatch = users.stream()
    .filter(u -> u.getCity().equals("NYC"))
    .findFirst();

// reduce
Optional<Integer> sum = numbers.stream().reduce(Integer::sum);
int product = numbers.stream().reduce(1, (a, b) -> a * b);
```

#### Collectors

```java
// toList, toSet, toMap
List<String> list = stream.collect(Collectors.toList());
Set<String> set = stream.collect(Collectors.toSet());
Map<Integer, User> map = users.stream()
    .collect(Collectors.toMap(User::getId, Function.identity()));

// groupingBy -- like SQL GROUP BY
Map<String, List<User>> byCity = users.stream()
    .collect(Collectors.groupingBy(User::getCity));

Map<String, Long> countByCity = users.stream()
    .collect(Collectors.groupingBy(User::getCity, Collectors.counting()));

Map<String, Optional<User>> oldestByCity = users.stream()
    .collect(Collectors.groupingBy(
        User::getCity,
        Collectors.maxBy(Comparator.comparingInt(User::getAge))
    ));

// partitioningBy -- splits into two groups
Map<Boolean, List<User>> partitioned = users.stream()
    .collect(Collectors.partitioningBy(u -> u.getAge() >= 18));

// joining
String csv = users.stream()
    .map(User::getName)
    .collect(Collectors.joining(", "));
```

#### Parallel Streams

```java
// Parallel stream -- uses ForkJoinPool.commonPool()
long count = items.parallelStream()
    .filter(ExpensiveFilter::check)
    .count();

// Controlling parallelism
System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "8");
```

> **WARNING**: Parallel streams are not a magic speedup. Only beneficial for large datasets (10k+ elements) with computationally expensive operations.

#### Primitive Streams

```java
// IntStream, LongStream, DoubleStream
IntStream.range(0, 100)
    .filter(n -> n % 2 == 0)
    .map(n -> n * n)
    .average()
    .orElse(0.0);

// Statistical
IntSummaryStatistics stats = IntStream.range(1, 100).summaryStatistics();
stats.getSum();
stats.getAverage();
stats.getMin();
stats.getMax();
stats.getCount();
```

### Internal Working

**Stream pipeline execution:**
1. **Build**: `stream()` creates a ReferencePipeline (head)
2. **Chain**: Each intermediate op wraps previous stage
3. **Evaluate**: Terminal op triggers evaluate which uses Spliterator
4. **Spliterator**: Splits source into partitions for parallel processing

**Short-circuit optimization:**
```java
stream.filter(expensivePredicate).findFirst()
// Stops evaluating after first match
```

### Common Mistakes

| Mistake | Wrong | Correct |
|---------|-------|---------|
| Reusing a stream | `stream.forEach(...); stream.count();` | Create new stream per terminal op |
| Mutable state in parallel | `stream.parallel().forEach(itemList::add)` | Use `toList()` or concurrent collections |
| Forgetting terminal op | `stream.filter(x -> x > 5);` -- no op | Add `collect()` or `forEach()` |
| Using `parallelStream()` on small data | 10 elements, overhead > benefit | Only for large, expensive operations |
| Side effects in `map()` | `map(x -> { log(x); return x; })` | Use `peek()` for side effects |

### Best Practices

- Favor streams over loops for collection processing
- Keep lambdas short and focused
- Use method references where possible
- Avoid side effects in stream operations
- Use `toList()` (Java 16+) over `collect(Collectors.toList())`
- Use primitive streams for numeric operations
- Use `Optional` correctly -- don't call `get()` without checking `isPresent()`

### Interview Questions

1. **What is a functional interface?** -- Interface with exactly one abstract method.
2. **What's the difference between intermediate and terminal operations?** -- Intermediate ops are lazy and return a stream; terminal ops are eager.
3. **How does `flatMap` differ from `map`?** -- `map` transforms 1:1; `flatMap` transforms each element into a stream and flattens.
4. **What is a stateful intermediate operation?** -- Operations like `sorted()`, `distinct()`, `limit()`.
5. **Can you reuse a stream after terminal operation?** -- No. Stream is consumed.
6. **Explain lazy evaluation in streams.** -- Pipeline built but not executed until terminal op.

### Practical Exercises

1. Given a list of transactions, compute total per type using `groupingBy`.
2. Use `flatMap` to extract all unique words from multiple sentences.
3. Implement a parallel word count.
4. Use `reduce` to find the product with the highest rating.

### Mini Project: Sales Analytics Engine

```java
public record Sale(String product, String region, double amount, LocalDate date) {}

public class SalesAnalytics {
    private final List<Sale> sales;

    public SalesAnalytics(List<Sale> sales) { this.sales = sales; }

    public Map<String, Double> revenueByRegion() {
        return sales.stream()
            .collect(Collectors.groupingBy(
                Sale::region,
                Collectors.summingDouble(Sale::amount)
            ));
    }

    public List<String> topProducts(int n) {
        return sales.stream()
            .collect(Collectors.groupingBy(
                Sale::product,
                Collectors.summingDouble(Sale::amount)
            ))
            .entrySet().stream()
            .sorted(Map.Entry.<String, Double>comparingByValue().reversed())
            .limit(n)
            .map(Map.Entry::getKey)
            .toList();
    }

    public Map<YearMonth, Double> monthlyRevenue() {
        return sales.stream()
            .collect(Collectors.groupingBy(
                sale -> YearMonth.from(sale.date()),
                TreeMap::new,
                Collectors.summingDouble(Sale::amount)
            ));
    }
}
```

### Revision Notes

- **Functional interfaces**: Predicate, Function, Consumer, Supplier
- **Lambda syntax**: `(params) -> expression` or `(params) -> { statements }`
- **Method references**: `String::length`, `System.out::println`, `User::new`
- **Stream pipeline**: Source -> 0+ intermediate ops (lazy) -> terminal op (eager)
- **Key ops**: filter, map, flatMap, distinct, sorted, peek, limit, skip
- **Collectors**: toList, toSet, toMap, groupingBy, partitioningBy, joining
- **Parallel streams**: ForkJoinPool; only for large data with expensive ops
- **Lazy evaluation**: Pipeline built upfront, executed on terminal op only
- **No reuse**: Stream is single-use

---
## Chapter 5: Exception Handling

### Introduction

Exception handling is Java's mechanism for managing runtime anomalies. It separates normal flow from error flow, provides meaningful diagnostics, and prevents resource leaks.

### Why It Matters

Poor exception handling leads to silent failures, lost stack traces, resource leaks, and security vulnerabilities. Good exception handling is the difference between "the system crashed" and "the payment failed because the third-party service returned HTTP 503 at 14:32:05 UTC."

### Real World Analogy

Exception handling is like **safety systems in a building**:
- **Try block**: The normal operation floor
- **Catch block**: Fire extinguishers for specific types of fires
- **Finally block**: The emergency exit--always available
- **Throw**: Pulling the fire alarm

### Theory

#### Throwable Hierarchy

```
                    Object
                       |
                   Throwable
                    /                   Exception    Error
             /    |           |
  Checked    |  RuntimeEx.  OutOfMemoryError
 (IOException)|(NullPtrEx.) | StackOverflowError
 SQLException |  ...        | ...
```

| Category | Checked? | Examples | Handling |
|----------|----------|----------|----------|
| **Error** | No (unchecked) | OutOfMemoryError, StackOverflowError | Cannot recover |
| **RuntimeException** | No (unchecked) | NullPointerException, IllegalArgumentException | Bug - should not happen |
| **Checked Exception** | Yes | IOException, SQLException | Must handle (catch or declare) |

#### Checked vs Unchecked

```java
// Checked -- must handle or declare
public void readFile(String path) throws IOException {
    Files.readString(Path.of(path));
}

// Client must either:
void caller() throws IOException {   // propagate
    readFile("data.txt");
}

void caller2() {
    try {                             // handle
        readFile("data.txt");
    } catch (IOException e) {
        log.error("Failed to read file", e);
        throw new RuntimeException(e);
    }
}

// Unchecked -- no declaration needed
public void setName(String name) {
    if (name == null) {
        throw new IllegalArgumentException("Name cannot be null");
    }
    this.name = name;
}
```

#### try-catch-finally

```java
try {
    connection = pool.getConnection();
    result = connection.executeQuery(sql);
    process(result);
} catch (SQLException e) {
    log.error("Database error: {}", e.getMessage());
    throw new DataAccessException("Failed to query data", e);
} catch (IOException e) {
    log.error("IO error: {}", e.getMessage());
} finally {
    if (connection != null) {
        try { connection.close(); }
        catch (SQLException e) { log.warn("Failed to close connection", e); }
    }
}
```

#### try-with-resources (Java 7+)

```java
// Resource must implement AutoCloseable
public static List<String> readLines(String path) throws IOException {
    try (BufferedReader reader = Files.newBufferedReader(Path.of(path));
         FileInputStream fis = new FileInputStream(path)) {
        return reader.lines().toList();
    }
}

// Multiple resources
try (Connection conn = dataSource.getConnection();
     PreparedStatement stmt = conn.prepareStatement(sql);
     ResultSet rs = stmt.executeQuery()) {
    while (rs.next()) { /* process */ }
}
```

> **Note**: `try-with-resources` adds suppressed exceptions. Call `e.getSuppressed()` to see all.

#### Custom Exceptions

```java
// Custom checked exception
public class InsufficientFundsException extends Exception {
    private final String accountId;
    private final BigDecimal requested;
    private final BigDecimal available;

    public InsufficientFundsException(String accountId, BigDecimal requested, BigDecimal available) {
        super(String.format("Account %s: requested %s but only %s available",
            accountId, requested, available));
        this.accountId = accountId;
        this.requested = requested;
        this.available = available;
    }

    public BigDecimal getShortfall() {
        return requested.subtract(available);
    }
}

// Custom unchecked exception
public class PaymentProcessingException extends RuntimeException {
    public PaymentProcessingException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

#### Best Practices for Exception Handling

**DO:**
- Catch specific exceptions, not `Exception` or `Throwable`
- Log exceptions with full stack trace (`log.error("msg", e)`)
- Use custom exceptions for domain-specific errors
- Include context in exception messages
- Use `try-with-resources` for all `AutoCloseable` resources
- Preserve the original exception when wrapping

**DON'T:**
- Catch and swallow (empty catch blocks)
- Use exceptions for control flow
- Log and rethrow the same exception
- Throw `Exception` in method signatures
- Catch `Throwable` (includes Error)

**Exception handling patterns:**

```java
// Pattern 1: try-with-resources for resources
public void processFile(String path) throws IOException {
    try (var reader = Files.newBufferedReader(Path.of(path))) { }
}

// Pattern 2: Exception translation
public class UserRepository {
    public User findById(long id) {
        try {
            return jdbcTemplate.queryForObject("SELECT * FROM users WHERE id=?", User.class, id);
        } catch (DataAccessException e) {
            throw new UserNotFoundException("User " + id + " not found", e);
        }
    }
}

// Pattern 3: Retry with exponential backoff
public <T> T retry(Supplier<T> operation, int maxRetries) {
    Exception lastException = null;
    for (int attempt = 1; attempt <= maxRetries; attempt++) {
        try { return operation.get(); }
        catch (Exception e) {
            lastException = e;
            if (attempt < maxRetries) {
                long wait = (long) Math.pow(2, attempt) * 100;
                Thread.sleep(wait);
            }
        }
    }
    throw new RuntimeException("Failed after " + maxRetries + " attempts", lastException);
}
```

### Internal Working

**How `finally` always executes:**

```java
public static int test() {
    try { return 1; }
    finally { System.out.println("Finally runs!"); }
}

public static int test2() {
    try { return 1; }
    finally { return 2; }  // OVERRIDES return!
}
// test2() returns 2 -- never do this
```

### Common Mistakes

| Mistake | Wrong | Correct |
|---------|-------|---------|
| Empty catch | `catch (Exception e) {}` | Log and handle, or don't catch |
| Catching generic exception | `catch (Exception e)` | Catch specific types |
| Log and rethrow | `log.error(e); throw e;` | Either log OR rethrow |
| Swallowing in finally | `finally { return -1; }` | Don't return in finally |
| Not preserving cause | `new RuntimeException(e.getMessage())` | `new RuntimeException(e)` |

> 💡 **Pro Tip:** Use checked exceptions for recoverable conditions where the caller can take meaningful action (e.g., retry a network call). Use unchecked exceptions for programming bugs (null checks, invalid args). Overusing checked exceptions leads to `throws Exception` declarations that provide no information.

### Interview Questions

1. **Difference between checked and unchecked exceptions?** -- Checked must be caught/declared.
2. **Can `finally` block execute without `try`?** -- No.
3. **What happens if `finally` throws an exception?** -- It replaces the original exception.
4. **What is exception chaining?** -- Wrapping one exception inside another.
5. **When should you create a custom exception?** -- For domain-specific semantics.

### Practical Exercises

1. Write a `BankAccount` class that throws custom `InsufficientFundsException`.
2. Implement `retry()` for an IOException-prone operation.
3. Refactor try-finally to try-with-resources.
4. Demonstrate suppressed exceptions.

### Mini Project: Robust File Processor

```java
public class RobustFileProcessor {
    private static final Logger log = LoggerFactory.getLogger(RobustFileProcessor.class);

    public record ProcessingResult(int successCount, int failureCount, List<String> errors) {}

    public ProcessingResult processFiles(List<String> filePaths) {
        int success = 0, failure = 0;
        List<String> errors = new ArrayList<>();

        for (String path : filePaths) {
            try {
                processSingleFile(path);
                success++;
            } catch (FileNotFoundException e) {
                failure++; errors.add("File not found: " + path);
                log.warn("Skipping missing file: {}", path);
            } catch (IOException e) {
                failure++; errors.add("IO error: " + path + ": " + e.getMessage());
                log.error("Failed to process {}", path, e);
            } catch (DataFormatException e) {
                failure++; errors.add("Invalid format: " + path);
                quarantineFile(path);
            }
        }
        return new ProcessingResult(success, failure, errors);
    }

    private void processSingleFile(String path) throws IOException, DataFormatException {
        Path filePath = Path.of(path);
        if (!Files.exists(filePath)) throw new FileNotFoundException(path);
        try (BufferedReader reader = Files.newBufferedReader(filePath)) {
            String line; int lineNum = 0;
            while ((line = reader.readLine()) != null) {
                lineNum++;
                try { processLine(line); }
                catch (IllegalArgumentException e) {
                    throw new DataFormatException("Error at line " + lineNum + ": " + line, e);
                }
            }
        }
    }

    private void processLine(String line) {
        if (line.isBlank() || line.startsWith("#")) return;
    }

    private void quarantineFile(String path) {
        try { Files.move(Path.of(path), Path.of(path + ".quarantined")); }
        catch (IOException e) { log.error("Failed to quarantine {}", path, e); }
    }
}
```

### Revision Notes

- **Throwable hierarchy**: Error (JVM) -> Exception -> RuntimeException (bug) or Checked (must handle)
- **try-catch-finally**: Finally always executes
- **try-with-resources**: Auto-closes AutoCloseable; handles suppressed exceptions
- **Custom exceptions**: Extend Exception (checked) or RuntimeException (unchecked)
- **Best practices**: Specific catches, preserve cause, log with context, never swallow

---
## Chapter 6: File I/O & Serialization

### Introduction

Java provides three generations of I/O APIs:
1. **Java 1.0-1.1**: Stream-based (byte) and Reader/Writer (character)
2. **Java 1.4**: NIO (New I/O) -- buffers, channels, selectors
3. **Java 7+**: NIO.2 -- `Path`, `Files`, `FileSystem` -- modern, feature-rich

### Why It Matters

Every real application deals with I/O: reading configuration, writing logs, processing data files, network communication, serialization. I/O is **the bottleneck** in most systems. Incorrect I/O handling causes resource leaks, poor performance, and data corruption.

### Real World Analogy

Think of I/O as **plumbing**:
- **Byte Streams**: Raw water pipes -- only move bytes
- **Character Streams**: Water purifiers -- handle encoding
- **Buffered Streams**: Water storage tanks -- batches for efficiency
- **NIO Channels**: High-pressure pipes -- direct, fast
- **Serialization**: Canning food -- object to bytes to object

### Theory

#### NIO.2 (Path, Files) -- Modern API

```java
// Creating paths
Path path = Path.of("C:", "data", "config.properties");
Path absolute = path.toAbsolutePath().normalize();

// Files utility class
List<String> lines = Files.readAllLines(path, StandardCharsets.UTF_8);  // small files
String content = Files.readString(path);                                // Java 11+
byte[] bytes = Files.readAllBytes(path);                                // binary

// Writing
Files.writeString(path, "Hello", StandardCharsets.UTF_8);               // Java 11+
Files.write(path, lines, StandardCharsets.UTF_8);

// Streaming (large files -- lazy)
try (Stream<String> lines = Files.lines(path)) {
    lines.filter(l -> l.startsWith("ERROR")).forEach(System.out::println);
}

// File operations
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
Files.move(source, target, StandardCopyOption.ATOMIC_MOVE);
Files.deleteIfExists(path);
Files.createDirectories(dir);    // creates all missing parents

// File attributes
Files.exists(path);
Files.isRegularFile(path);
Files.isDirectory(path);
Files.size(path);
Files.getLastModifiedTime(path);

// Walking directory tree
try (Stream<Path> walk = Files.walk(startDir)) {
    walk.filter(Files::isRegularFile)
        .filter(p -> p.toString().endsWith(".java"))
        .forEach(System.out::println);
}

// Watching a directory for changes
WatchService watcher = FileSystems.getDefault().newWatchService();
path.register(watcher, StandardWatchEventKinds.ENTRY_CREATE,
    StandardWatchEventKinds.ENTRY_MODIFY,
    StandardWatchEventKinds.ENTRY_DELETE);
```

#### Channels and Buffers (NIO)

```java
// FileChannel -- fast, position-based I/O
try (FileChannel channel = FileChannel.open(Path.of("large.bin"),
        StandardOpenOption.READ, StandardOpenOption.WRITE)) {

    ByteBuffer buffer = ByteBuffer.allocate(4096);
    int bytesRead = channel.read(buffer);
    buffer.flip();                         // switch to read mode
    while (buffer.hasRemaining()) {
        byte b = buffer.get();
    }
    buffer.clear();                        // switch to write mode

    // Position-based operations
    channel.position(1024);
    MappedByteBuffer mapped = channel.map(
        FileChannel.MapMode.READ_WRITE, 0, channel.size());
    // Direct memory-mapped file access -- zero-copy
}
```

#### Serialization

```java
// Serializable interface -- marker (no methods)
public class Employee implements Serializable {
    private static final long serialVersionUID = 1L;  // version control

    private String name;
    private int id;
    private transient String password;     // not serialized

    @Serial
    private void writeObject(ObjectOutputStream out) throws IOException {
        out.defaultWriteObject();
        out.writeObject(encrypt(password));
    }

    @Serial
    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
        in.defaultReadObject();
        this.password = decrypt((String) in.readObject());
    }
}

// Serialization
try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("employee.ser"))) {
    oos.writeObject(employee);
}

// Deserialization
try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("employee.ser"))) {
    Employee emp = (Employee) ois.readObject();
}
```

> **WARNING**: Java serialization is slow, insecure, and verbose. Prefer JSON (Jackson/Gson), Protocol Buffers, or Avro.

### Internal Working

**How BufferedInputStream works:**
1. Wraps another InputStream
2. On first read(), fills internal buffer (8KB) via one system call
3. Subsequent reads return from buffer -- no system call
4. When buffer exhausted, reads next chunk

**Memory-Mapped File (MappedByteBuffer):**
1. `mmap()` system call maps file into virtual memory
2. Pages loaded on demand (page fault)
3. Changes written back by the OS
4. Zero-copy: JVM to OS cache to disk, bypassing JVM heap

### Common Mistakes

| Mistake | Wrong | Correct |
|---------|-------|---------|
| Not closing resources | new FileInputStream("f") without close | try-with-resources |
| Ignoring encoding | new FileReader("f") | Specify Charset explicitly |
| Byte-by-byte read | while(in.read() != -1) | Use BufferedInputStream or read(byte[]) |
| Serialization without UID | Default UID changes on recompilation | Declare serialVersionUID |
| Reading entire file for large files | readAllBytes() on 10GB file | Use Files.lines() or channels |

### Best Practices

- Always use `try-with-resources` for I/O
- Specify character encoding explicitly
- Use NIO.2 (`Path` + `Files`) over legacy `File`
- Buffer I/O streams
- For large files, use streaming APIs like `Files.lines()`
- Prefer JSON/Protocol Buffers over Java serialization

### Interview Questions

1. **Difference between InputStream and Reader?** -- InputStream reads bytes; Reader reads characters.
2. **What is try-with-resources?** -- Auto-closes AutoCloseable resources.
3. **How does NIO differ from standard I/O?** -- NIO is buffer-oriented, non-blocking, channel-based.
4. **What is serialVersionUID?** -- Version ID for serialization.
5. **What does transient mean?** -- Field is not serialized.

### Practical Exercises

1. Read a log file, extract all ERROR lines, write to a separate file.
2. Implement recursive directory search for files matching a glob.
3. Serialize a Person object, modify the class, demonstrate InvalidClassException.
4. Calculate total size of all .java files in a directory tree using Files.walk().

### Mini Project: Config File Loader

```java
public class ConfigLoader {
    private final Path configPath;
    private final Properties properties = new Properties();

    public ConfigLoader(String configPath) {
        this.configPath = Path.of(configPath);
    }

    public void load() throws IOException {
        if (!Files.exists(configPath)) {
            log.warn("Config file not found, using defaults", configPath);
            return;
        }
        try (var reader = Files.newBufferedReader(configPath, StandardCharsets.UTF_8)) {
            properties.load(reader);
        }
        validateRequiredKeys();
    }

    private void validateRequiredKeys() {
        List<String> missing = List.of("db.url", "db.user", "db.password").stream()
            .filter(k -> !properties.containsKey(k)).toList();
        if (!missing.isEmpty())
            throw new ConfigurationException("Missing: " + missing);
    }

    public String get(String key) { return properties.getProperty(key); }
    public String get(String key, String defaultValue) { return properties.getProperty(key, defaultValue); }
    public int getInt(String key, int defaultValue) {
        String val = properties.getProperty(key);
        return val != null ? Integer.parseInt(val) : defaultValue;
    }
}
```

### Revision Notes

- **Modern**: Path, Files, Files.lines(), Files.walk()
- **Encoding**: Always specify StandardCharsets.UTF_8
- **try-with-resources**: Auto-closes AutoCloseable
- **Serialization**: Serializable (marker), transient (skip), serialVersionUID (version)
- **NIO**: Channels + Buffers, memory-mapped files, WatchService

---
## Chapter 7: Multithreading & Concurrency

### Introduction

Multithreading enables concurrent execution of multiple tasks within a single process. Java's concurrency APIs have evolved from low-level `synchronized`/`wait/notify` to high-level `ExecutorService`, `CompletableFuture`, and Virtual Threads (Java 21).

### Why It Matters

Concurrency enables throughput, responsiveness, scalability, and hardware utilization. But it is **hard**: race conditions, deadlocks, and memory visibility issues are notoriously difficult to debug.

### Real World Analogy

Think of a **restaurant kitchen**:
- **Thread**: A chef
- **Synchronized**: Only one chef can use the oven at a time
- **Lock**: The recipe book held by one chef
- **Deadlock**: Chef A has pan, Chef B has spatula; each needs the other's tool
- **ThreadPool**: Fixed number of chefs, tasks queue up
- **CompletableFuture**: "Start grilling; when ready, plate it and notify the waiter"
- **Volatile**: The "Order Up!" board -- changes immediately visible to all

### Theory

#### Thread vs Runnable

```java
// Extending Thread (not recommended)
public class Worker extends Thread {
    @Override
    public void run() {
        System.out.println("Working in: " + Thread.currentThread().getName());
    }
}
new Worker().start();

// Implementing Runnable (recommended)
public class Task implements Runnable {
    @Override
    public void run() {
        System.out.println("Task in: " + Thread.currentThread().getName());
    }
}
new Thread(new Task()).start();

// Lambda (Java 8+)
new Thread(() -> System.out.println("Lambda thread")).start();
```

#### Thread Lifecycle

```
NEW -> RUNNABLE -> BLOCKED/WAITING/TIMED_WAITING -> TERMINATED
```

#### The `synchronized` Keyword

```java
public class Counter {
    private int count = 0;

    public synchronized void increment() { count++; }
    public static synchronized void resetAll() { }
    public void decrement() {
        synchronized (this) { count--; }
    }

    private final Object lock = new Object();
    public void add(int value) {
        synchronized (lock) { count += value; }
    }
}
```

**What `synchronized` guarantees:**
1. **Mutual exclusion**: Only one thread executes at a time
2. **Visibility**: Changes visible to threads that acquire the same lock
3. **Atomicity**: Synchronized block executes as a single unit

> ✅ **Best Practice:** Prefer `java.util.concurrent` high-level APIs over raw `synchronized`. `ConcurrentHashMap`, `ReentrantLock`, `Semaphore`, `CountDownLatch`, and `Atomic*` classes offer better performance, more features, and clearer intent. Use `synchronized` only for simple critical sections.

#### Volatile

```java
public class RunningFlag {
    private volatile boolean running = true;

    public void stop() { running = false; }
    public void run() {
        while (running) { /* guaranteed to see the update */ }
    }
}
```

#### Lock API

```java
// ReentrantLock -- more flexible than synchronized
public class BankAccount {
    private final Lock lock = new ReentrantLock();
    private double balance;

    public void transfer(BankAccount to, double amount) {
        lock.lock();
        try {
            if (balance >= amount) {
                this.balance -= amount;
                to.deposit(amount);
            }
        } finally { lock.unlock(); }
    }

    public boolean tryTransfer(BankAccount to, double amount) {
        if (lock.tryLock(100, TimeUnit.MILLISECONDS)) {
            try { /* transfer */ return true; }
            finally { lock.unlock(); }
        }
        return false;
    }
}

// ReadWriteLock -- multiple readers OR single writer
public class Cache {
    private final Map<String, Object> data = new HashMap<>();
    private final ReadWriteLock rwLock = new ReentrantReadWriteLock();

    public Object get(String key) {
        rwLock.readLock().lock();
        try { return data.get(key); }
        finally { rwLock.readLock().unlock(); }
    }

    public void put(String key, Object value) {
        rwLock.writeLock().lock();
        try { data.put(key, value); }
        finally { rwLock.writeLock().unlock(); }
    }
}
```

#### ExecutorService

```java
// ThreadPoolExecutor
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    4, 10, 60, TimeUnit.SECONDS,
    new LinkedBlockingQueue<>(100),
    new ThreadPoolExecutor.CallerRunsPolicy()
);

// Common executors
ExecutorService fixedPool = Executors.newFixedThreadPool(4);
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);

// Usage
Future<String> future = executor.submit(() -> {
    Thread.sleep(1000);
    return "Result";
});
String result = future.get(2, TimeUnit.SECONDS);

// Scheduled tasks
scheduler.scheduleAtFixedRate(() -> {}, 0, 10, TimeUnit.SECONDS);

// Shutdown
executor.shutdown();
executor.awaitTermination(30, TimeUnit.SECONDS);
```

#### CompletableFuture

```java
public class OrderService {
    public CompletableFuture<Order> placeOrder(Cart cart) {
        return CompletableFuture
            .supplyAsync(() -> inventory.checkStock(cart))
            .thenApply(stockResult -> {
                if (!stockResult.available()) throw new OutOfStockException();
                return cart;
            })
            .thenComposeAsync(validCart -> payment.processPayment(validCart.total()))
            .thenApplyAsync(paymentResult -> shipping.schedule(createOrder(cart, paymentResult)))
            .exceptionally(ex -> {
                log.error("Order failed", ex);
                return Order.failed(ex.getMessage());
            });
    }

    // Parallel composition
    public CompletableFuture<Dashboard> loadDashboard() {
        CompletableFuture<SalesData> sales = CompletableFuture.supplyAsync(this::getSalesData);
        CompletableFuture<List<Order>> recent = CompletableFuture.supplyAsync(this::getRecentOrders);
        CompletableFuture<Map<String, Integer>> inventory = CompletableFuture.supplyAsync(this::getInventoryCounts);

        return CompletableFuture.allOf(sales, recent, inventory)
            .thenApply(v -> new Dashboard(sales.join(), recent.join(), inventory.join()));
    }
}
```

> 💡 **Pro Tip:** Virtual Threads (Java 21+) make it practical to have millions of threads for I/O-bound tasks. Unlike platform threads, virtual threads have minimal overhead. Use `Executors.newVirtualThreadPerTaskExecutor()` for high-throughput I/O workloads. They're not suitable for CPU-bound tasks — use platform threads for those.

#### Concurrent Collections

```java
// ConcurrentHashMap
ConcurrentHashMap<String, Integer> counts = new ConcurrentHashMap<>();
counts.putIfAbsent("key", 1);
counts.computeIfAbsent("key", k -> 1);
counts.merge("key", 1, Integer::sum);

// CopyOnWriteArrayList -- thread-safe, snapshot iteration
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
list.add("item");  // creates new array copy!

// BlockingQueue -- producer-consumer
BlockingQueue<Runnable> queue = new LinkedBlockingQueue<>(100);
queue.put(task);       // blocks if full
Runnable task = queue.take();  // blocks if empty
```

#### Atomic Variables

```java
public class AtomicCounter {
    private final AtomicInteger count = new AtomicInteger(0);

    public int increment() { return count.incrementAndGet(); }
    public boolean compareAndSet(int expected, int update) {
        return count.compareAndSet(expected, update);
    }
}

// LongAdder -- higher throughput for high-contention stats
LongAdder adder = new LongAdder();
adder.increment();
adder.sum();
```

#### Fork/Join Framework

```java
public class SumTask extends RecursiveTask<Long> {
    private static final int THRESHOLD = 1_000;
    private final long[] array;
    private final int start, end;

    public SumTask(long[] array, int start, int end) {
        this.array = array; this.start = start; this.end = end;
    }

    @Override
    protected Long compute() {
        if (end - start <= THRESHOLD) {
            long sum = 0;
            for (int i = start; i < end; i++) sum += array[i];
            return sum;
        }
        int mid = (start + end) / 2;
        SumTask left = new SumTask(array, start, mid);
        SumTask right = new SumTask(array, mid, end);
        left.fork();
        long rightResult = right.compute();
        long leftResult = left.join();
        return leftResult + rightResult;
    }

    public static long sum(long[] array) {
        return ForkJoinPool.commonPool().invoke(new SumTask(array, 0, array.length));
    }
}
```

### Internal Working

**How `synchronized` is implemented:**
- Each object has a **monitor** (implicit lock)
- Biased locking (Java 5-15) -> lightweight lock (CAS) -> heavy lock (OS mutex)
- Java 21 removed biased locking

**How ForkJoinPool works:**
- Each worker thread has a **deque** of tasks
- **Work-stealing**: Idle threads steal tasks from others' deque

### Common Mistakes

| Mistake | Wrong | Correct |
|---------|-------|---------|
| Double-checked locking (without volatile) | if (instance == null) { synchronized {} } | Declare instance volatile |
| Thread starvation | Fixed pool for all tasks | Separate pools for I/O vs CPU |
| Deadlock | Nested synchronized in different order | Always acquire locks in same order |
| Swallowing InterruptedException | catch (InterruptedException e) {} | Restore interrupt flag |

### Best Practices

- Prefer `ExecutorService` over raw `Thread`
- Prefer high-level concurrency over low-level
- Use immutable objects to eliminate synchronization
- Use `ConcurrentHashMap` instead of synchronizing HashMap
- Use `LongAdder` for high-contention counters
- Restore interrupt flag when catching InterruptedException

### Interview Questions

1. **What is a race condition?** -- Two threads modifying shared data without synchronization.
2. **What is a deadlock?** -- Threads each holding a lock the other needs.
3. **How does volatile work?** -- Prevents CPU caching; forces main memory reads/writes.
4. **Difference between synchronized and ReentrantLock?** -- ReentrantLock provides tryLock, fairness, conditions.
5. **What are virtual threads (Java 21)?** -- Lightweight JVM-managed threads.
6. **How does ConcurrentHashMap achieve high concurrency?** -- Lock-free reads; fine-grained locking on bins.

### Practical Exercises

1. Implement a thread-safe bounded blocking queue.
2. Refactor to use ReentrantLock and Condition.
3. Write a program using CompletableFuture to combine 3 async API calls.
4. Use LongAdder to count requests in a multi-threaded server simulation.

### Mini Project: Concurrent Job Scheduler

```java
public class JobScheduler {
    private final ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);
    private final ExecutorService jobPool = Executors.newFixedThreadPool(4);
    private final ConcurrentHashMap<String, ScheduledFuture<?>> jobs = new ConcurrentHashMap<>();
    private final AtomicInteger jobCounter = new AtomicInteger(0);

    public record JobResult(String jobId, String name, long durationMs, boolean success) {}

    public String scheduleRecurring(String name, Runnable task, long interval, TimeUnit unit) {
        String jobId = "JOB-" + jobCounter.incrementAndGet();
        ScheduledFuture<?> future = scheduler.scheduleAtFixedRate(
            () -> submitJob(jobId, name, task), 0, interval, unit);
        jobs.put(jobId, future);
        return jobId;
    }

    private void submitJob(String jobId, String name, Runnable task) {
        CompletableFuture.runAsync(() -> {
            long start = System.currentTimeMillis();
            try { task.run(); onJobComplete(new JobResult(jobId, name, System.currentTimeMillis() - start, true)); }
            catch (Exception e) { onJobComplete(new JobResult(jobId, name, System.currentTimeMillis() - start, false)); }
        }, jobPool);
    }

    protected void onJobComplete(JobResult result) {
        System.out.println("Job " + result.name() + " done in " + result.durationMs() + "ms");
    }

    public boolean cancelJob(String jobId) {
        ScheduledFuture<?> future = jobs.get(jobId);
        if (future != null) return future.cancel(true);
        return false;
    }

    public void shutdown() { scheduler.shutdown(); jobPool.shutdown(); }
}
```

### Revision Notes

- **Thread states**: NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING, TERMINATED
- **synchronized**: Mutual exclusion + visibility
- **volatile**: Visibility only; no atomicity
- **Lock API**: ReentrantLock, ReadWriteLock
- **ExecutorService**: ThreadPoolExecutor, ScheduledExecutorService
- **CompletableFuture**: thenApply, thenCompose, thenCombine, allOf
- **ConcurrentHashMap**: Lock-free reads, fine-grained writes
- **Atomic classes**: CAS-based lock-free operations
- **Fork/Join**: Work-stealing for divide-and-conquer

---
## Chapter 8: JDBC

### Introduction

JDBC (Java Database Connectivity) is the standard API for connecting Java applications to relational databases. It provides a vendor-independent interface.

### Why It Matters

Despite ORMs (Hibernate, jOOQ), JDBC remains the foundation of all Java database frameworks. Essential for simple queries, bulk operations, and when ORMs don't fit.

### Real World Analogy

Think of JDBC like a **universal wall outlet adapter**:
- **Driver**: Physical adapter (MySQL, PostgreSQL, etc.)
- **Connection**: The wall outlet
- **Statement**: A specific appliance
- **PreparedStatement**: Appliance with surge protector (prevents SQL injection)
- **ResultSet**: The readout

### Theory

#### Connection Management

```java
String url = "jdbc:mysql://localhost:3306/mydb";
String user = "app_user";
String password = "secret";

try (Connection conn = DriverManager.getConnection(url, user, password)) {
    System.out.println("Connected: " + conn.isValid(5));
    conn.setAutoCommit(false);
    conn.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);
}
```

#### Statement vs PreparedStatement

```java
// Statement -- NEVER use with user input
public User findById_Unsafe(long id) throws SQLException {
    String sql = "SELECT * FROM users WHERE id = " + id;  // BAD: SQL injection
    try (Statement stmt = connection.createStatement();
         ResultSet rs = stmt.executeQuery(sql)) {
        return rs.next() ? mapUser(rs) : null;
    }
}

// PreparedStatement -- ALWAYS use this
public User findById_Safe(long id) throws SQLException {
    String sql = "SELECT * FROM users WHERE id = ?";
    try (PreparedStatement ps = connection.prepareStatement(sql)) {
        ps.setLong(1, id);
        try (ResultSet rs = ps.executeQuery()) {
            return rs.next() ? mapUser(rs) : null;
        }
    }
}
```

**PreparedStatement benefits:**
1. **SQL injection prevention**: Parameters are escaped
2. **Performance**: Precompiled, reusable
3. **Type safety**: setInt, setString, setDate
4. **Binary data**: setBlob, setBytes

#### Batch Processing

```java
public void insertEmployees(List<Employee> employees) throws SQLException {
    String sql = "INSERT INTO employees (name, email, dept, salary) VALUES (?, ?, ?, ?)";
    try (PreparedStatement ps = conn.prepareStatement(sql)) {
        int batchSize = 0;
        for (Employee emp : employees) {
            ps.setString(1, emp.name()); ps.setString(2, emp.email());
            ps.setString(3, emp.dept()); ps.setDouble(4, emp.salary());
            ps.addBatch();
            batchSize++;
            if (batchSize % 1000 == 0) { ps.executeBatch(); ps.clearBatch(); }
        }
        ps.executeBatch();
    }
}
// Without batch: N round trips. With batch: N/1000 round trips
```

#### Connection Pooling (HikariCP)

```java
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:postgresql://localhost:5432/mydb");
config.setUsername("app_user");
config.setPassword("secret");
config.setMaximumPoolSize(20);
config.setMinimumIdle(5);
config.setConnectionTimeout(5000);
config.setMaxLifetime(600000);
config.addDataSourceProperty("cachePrepStmts", "true");
config.addDataSourceProperty("prepStmtCacheSize", "250");

HikariDataSource dataSource = new HikariDataSource(config);

// Usage
try (Connection conn = dataSource.getConnection()) { /* fast -- no TCP handshake */ }
```

> **Never** use `DriverManager.getConnection()` in production. Always use connection pooling.

#### Transaction Management

```java
public void transferFunds(long fromId, long toId, BigDecimal amount) throws SQLException {
    String debitSql = "UPDATE accounts SET balance = balance - ? WHERE id = ? AND balance >= ?";
    String creditSql = "UPDATE accounts SET balance = balance + ? WHERE id = ?";

    Connection conn = dataSource.getConnection();
    try {
        conn.setAutoCommit(false);
        conn.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);

        try (PreparedStatement ps = conn.prepareStatement(debitSql)) {
            ps.setBigDecimal(1, amount); ps.setLong(2, fromId); ps.setBigDecimal(3, amount);
            if (ps.executeUpdate() == 0) throw new InsufficientFundsException();
        }

        try (PreparedStatement ps = conn.prepareStatement(creditSql)) {
            ps.setBigDecimal(1, amount); ps.setLong(2, toId);
            ps.executeUpdate();
        }

        conn.commit();
    } catch (SQLException e) { conn.rollback(); throw e; }
    finally { conn.setAutoCommit(true); conn.close(); }
}
```

### Internal Working

**Connection pooling flow:**
1. Application calls `dataSource.getConnection()`
2. Pool checks for idle connections
3. If idle available: return wrapped connection (no TCP handshake)
4. If none idle and pool not full: create new connection
5. If pool full: block until timeout or connection returned
6. On close(): connection returned to pool (not actually closed)

### Common Mistakes

| Mistake | Wrong | Correct |
|---------|-------|---------|
| SQL injection | String concatenation | PreparedStatement |
| Not closing resources | Open Connection etc. | try-with-resources |
| No connection pooling | DriverManager each time | Use HikariCP |
| Not handling transactions | Auto-commit for multi-step | Manual commit/rollback |
| Ignoring batch processing | INSERT each record individually | Use addBatch() + executeBatch() |

### Best Practices

- Always use `PreparedStatement` over `Statement`
- Use connection pooling (HikariCP)
- Use try-with-resources for all JDBC resources
- Batch processing for bulk inserts/updates
- Set fetchSize for large result sets
- Use transaction isolation levels appropriately

### Interview Questions

1. **What is JDBC?** -- Standard API for database access.
2. **Difference between Statement and PreparedStatement?** -- PreparedStatement precompiles, prevents SQL injection.
3. **What is connection pooling?** -- Reuses connections to avoid TCP handshake overhead.
4. **How does batch processing work?** -- addBatch() + executeBatch() for bulk operations.
5. **What are transaction isolation levels?** -- READ_UNCOMMITTED, READ_COMMITTED, REPEATABLE_READ, SERIALIZABLE.

### Practical Exercises

1. Write a JDBC program to connect and list all tables.
2. Implement batch insert of 10,000 records.
3. Write a transfer operation with transaction management.
4. Set up HikariCP connection pool.

### Mini Project: DAO Layer with JDBC

```java
public class UserDao {
    private final DataSource dataSource;

    public UserDao(DataSource dataSource) { this.dataSource = dataSource; }

    public Optional<User> findById(long id) {
        String sql = "SELECT id, name, email, created_at FROM users WHERE id = ?";
        try (Connection conn = dataSource.getConnection();
             PreparedStatement ps = conn.prepareStatement(sql)) {
            ps.setLong(1, id);
            try (ResultSet rs = ps.executeQuery()) {
                if (rs.next()) return Optional.of(new User(
                    rs.getLong("id"), rs.getString("name"),
                    rs.getString("email"),
                    rs.getTimestamp("created_at").toLocalDateTime()));
            }
        } catch (SQLException e) { throw new DataAccessException(e); }
        return Optional.empty();
    }

    public User save(User user) {
        String sql = "INSERT INTO users (name, email, created_at) VALUES (?, ?, ?)";
        try (Connection conn = dataSource.getConnection();
             PreparedStatement ps = conn.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS)) {
            ps.setString(1, user.name()); ps.setString(2, user.email());
            ps.setTimestamp(3, Timestamp.valueOf(user.createdAt()));
            ps.executeUpdate();
            try (ResultSet keys = ps.getGeneratedKeys()) {
                if (keys.next()) return new User(keys.getLong(1), user.name(), user.email(), user.createdAt());
            }
        } catch (SQLException e) { throw new DataAccessException(e); }
        return user;
    }
}
```

### Revision Notes

- **JDBC**: Standard API for database access
- **Connection**: DriverManager or DataSource (pooled)
- **PreparedStatement**: Parameterized queries, SQL injection prevention
- **ResultSet**: Cursor-based; set fetchSize for large data
- **Batch processing**: addBatch() + executeBatch()
- **Connection pooling**: HikariCP (always use in production)
- **Transactions**: setAutoCommit(false), commit(), rollback()

---
## Chapter 9: Reflection & Annotations

### Introduction

Reflection allows a Java program to inspect and manipulate itself at runtime. Combined with annotations, reflection enables frameworks like Spring, Hibernate, JUnit, and Jackson to work their magic.

### Why It Matters

Reflection is the backbone of:
- **Frameworks**: Spring (DI, AOP), Hibernate (ORM), JUnit (test discovery)
- **Serialization**: Jackson, Gson (field inspection)
- **Dynamic proxies**: AOP, transaction management
- **Code generation**: Lombok, MapStruct

### Real World Analogy

Reflection is like a **building inspector** who can:
- Read blueprints (Class metadata)
- Inspect rooms (Fields)
- Test doors (Methods)
- Operate switches (Invoke methods)
- Read labels (Annotations)

### Theory

#### The Class Class

```java
// Getting a Class object
Class<String> stringClass = String.class;
Class<?> stringClass2 = "hello".getClass();
Class<?> stringClass3 = Class.forName("java.lang.String");

// Class metadata
Class<?> clazz = User.class;
clazz.getName();              // "com.example.User"
clazz.getSimpleName();        // "User"
clazz.getSuperclass();        // Class for parent
clazz.getInterfaces();        // Class[] for interfaces
clazz.getModifiers();         // int bitmask
Modifier.isPublic(clazz.getModifiers());
clazz.isInterface();
clazz.isEnum();
clazz.isAnnotation();
```

#### Constructor Reflection

```java
// Getting constructors
Constructor<?>[] constructors = User.class.getConstructors();       // public only
Constructor<?>[] allCtors = User.class.getDeclaredConstructors();   // all

// Specific constructor
Constructor<User> ctor = User.class.getConstructor(String.class);
Constructor<User> privateCtor = User.class.getDeclaredConstructor(String.class, int.class);
privateCtor.setAccessible(true);

// Creating instances
User user1 = ctor.newInstance("Alice");
User user2 = privateCtor.newInstance("Bob", 30);
```

#### Field Reflection

```java
Field[] fields = User.class.getFields();            // public only
Field[] allFields = User.class.getDeclaredFields(); // all

Field nameField = User.class.getDeclaredField("name");
nameField.setAccessible(true);

// Reading/writing
User user = new User("Alice", 25);
String name = (String) nameField.get(user);         // "Alice"
nameField.set(user, "Bob");

// Metadata
nameField.getType();         // Class<String>
nameField.getName();         // "name"
nameField.getModifiers();    // private
```

#### Method Reflection

```java
Method[] methods = Calculator.class.getMethods();      // public + inherited
Method[] allMethods = Calculator.class.getDeclaredMethods();  // all declared

Method addMethod = Calculator.class.getMethod("add", int.class, int.class);
Method formatMethod = Calculator.class.getDeclaredMethod("format", double.class);
formatMethod.setAccessible(true);

// Invoking
Calculator calc = new Calculator();
int result = (int) addMethod.invoke(calc, 3, 4);       // 7

// Static method -- first arg is null
String formatted = (String) formatMethod.invoke(null, 3.14159);

// Metadata
addMethod.getReturnType();
addMethod.getParameterTypes();
addMethod.getName();
```

#### Custom Annotations

```java
// Defining a custom annotation
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD, ElementType.METHOD})
public @interface JsonField {
    String name() default "";
    boolean required() default true;
}

// Using the annotation
public class Product {
    @JsonField(name = "product_id", required = true)
    private String id;

    @JsonField(name = "product_name")
    private String name;
}

// Reading annotations
Field[] fields = Product.class.getDeclaredFields();
for (Field field : fields) {
    JsonField annotation = field.getAnnotation(JsonField.class);
    if (annotation != null) {
        String jsonName = annotation.name().isEmpty() ? field.getName() : annotation.name();
        System.out.println(field.getName() + " -> " + jsonName);
    }
}
```

#### Annotation Retention & Target

```java
@Retention(RetentionPolicy.SOURCE)     // Discarded by compiler (e.g., @Override)
@Retention(RetentionPolicy.CLASS)      // In .class but not at runtime (e.g., Lombok)
@Retention(RetentionPolicy.RUNTIME)    // Available at runtime (via reflection)

@Target(ElementType.TYPE)              // Class, interface, enum
@Target(ElementType.FIELD)             // Field
@Target(ElementType.METHOD)            // Method
@Target(ElementType.PARAMETER)         // Parameter
@Target({ElementType.FIELD, ElementType.METHOD})  // Multiple
```

#### Built-in Annotations

```java
@Override                    // Ensures method overrides superclass
@FunctionalInterface        // Ensures exactly one abstract method
@Deprecated(since = "2.0", forRemoval = true)
@SuppressWarnings("unchecked")
@SafeVarargs                // Suppresses varargs heap pollution
```

### Internal Working

**How Spring uses reflection:**
1. Classpath scanning finds @Component, @Service, etc.
2. Reflection inspects constructors for @Autowired
3. `Constructor.newInstance()` creates beans
4. `Proxy.newProxyInstance()` creates AOP proxies

### Common Mistakes

| Mistake | Why It's Wrong |
|---------|----------------|
| Committing `target/` or `build/` directories | Generated artifacts inflate repo size and cause merge conflicts |
| Not pinning dependency versions | `LATEST` or `RELEASE` causes non-reproducible builds |
| Mixing dependency scopes incorrectly | `provided` dependencies missing at runtime; `test` scope leaking into production |
| Forgetting to add `maven-wrapper` or `gradle-wrapper` | Every team member must install the right build tool version manually |

> 💡 **Pro Tip:** Use a Bill of Materials (BOM) to manage dependency versions consistently across multi-module projects. Spring Boot's `spring-boot-dependencies` BOM and the `java-platform` / `enforced-platform` Gradle plugin are excellent examples of version management.

> ⚠️ **Warning:** Reflection breaks compile-time safety. A method rename won't fail at compile time — it crashes at runtime with `NoSuchMethodException`. Always write integration tests for reflective code. Consider annotation processors (compile-time code generation) as a safer alternative.

### Best Practices

- Use reflection sparingly
- Cache reflective objects for reuse
- Always call setAccessible with caution
- Use isAnnotationPresent() before getAnnotation()
- Consider MethodHandle (Java 7+) for better performance

### Interview Questions

1. **What is reflection?** -- Runtime inspection of classes.
2. **How do you get a Class object?** -- .class, getClass(), Class.forName().
3. **What is setAccessible(true)?** -- Suppresses access control checks.
4. **What are annotations?** -- Metadata added to code; processed at compile time or runtime.
5. **Difference between RetentionPolicy SOURCE, CLASS, RUNTIME?** -- SOURCE discarded by compiler; CLASS in .class; RUNTIME available via reflection.

### Practical Exercises

1. Write a method that prints all fields of any object using reflection.
2. Create a @LogMethod annotation that logs method entry/exit.
3. Implement a simple JSON serializer using reflection.
4. Build a mini DI container.

### Mini Project: Simple ORM-like Framework

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Table { String name(); }

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface Column {
    String name() default "";
    boolean primaryKey() default false;
}

@Table(name = "users")
public class UserEntity {
    @Column(name = "id", primaryKey = true) private Long id;
    @Column(name = "user_name") private String name;
    @Column(name = "email") private String email;
}

public class SimpleORM {
    public String toInsertSQL(Object entity) {
        Class<?> clazz = entity.getClass();
        Table table = clazz.getAnnotation(Table.class);
        if (table == null) throw new IllegalArgumentException("Missing @Table");

        StringBuilder columns = new StringBuilder();
        StringBuilder values = new StringBuilder();

        for (Field field : clazz.getDeclaredFields()) {
            Column column = field.getAnnotation(Column.class);
            if (column == null) continue;
            field.setAccessible(true);
            String colName = column.name().isEmpty() ? field.getName() : column.name();
            try {
                Object value = field.get(entity);
                columns.append(colName).append(", ");
                values.append(formatValue(value)).append(", ");
            } catch (IllegalAccessException e) { throw new RuntimeException(e); }
        }

        columns.setLength(columns.length() - 2);
        values.setLength(values.length() - 2);
        return "INSERT INTO " + table.name() + " (" + columns + ") VALUES (" + values + ")";
    }

    private String formatValue(Object value) {
        if (value == null) return "NULL";
        if (value instanceof String) return "'" + value.toString().replace("'", "''") + "'";
        return value.toString();
    }
}
```

### Revision Notes

- **Class object**: .class, getClass(), Class.forName()
- **Reflection**: Constructor, Method, Field
- **setAccessible(true)**: Bypasses access control
- **Annotations**: @Retention (SOURCE/CLASS/RUNTIME), @Target (TYPE/FIELD/METHOD)
- **Custom annotations**: @interface
- **Built-in**: @Override, @FunctionalInterface, @Deprecated, @SuppressWarnings

---
## Chapter 10: Build Tools

### Introduction

Build tools automate compiling, dependency management, testing, packaging, and deployment. **Maven** and **Gradle** are the two dominant Java build tools.

### Why It Matters

Without build tools: manual JAR downloads, manual classpath management, non-reproducible builds, no CI/CD integration. Build tools ensure "it works on my machine" becomes "it works everywhere."

### Real World Analogy

- **Maven's POM**: A detailed recipe card (ingredients, steps, in order)
- **Gradle's build.gradle**: A programmable recipe (conditional logic, automation)
- **Dependencies**: Ingredients from warehouses (repositories)
- **Lifecycle**: The predefined production workflow

### Theory

#### Maven POM

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>

    <properties>
        <java.version>21</java.version>
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>0.8.11</version>
            </plugin>
        </plugins>
    </build>

    <profiles>
        <profile>
            <id>production</id>
            <properties>
                <app.environment>production</app.environment>
            </properties>
        </profile>
    </profiles>
</project>
```

#### Maven Lifecycle

**Default lifecycle phases:**
```
validate -> compile -> test -> package -> verify -> install -> deploy
```

| Phase | Description |
|-------|-------------|
| `validate` | Validates project structure |
| `compile` | Compiles src/main/java |
| `test` | Runs tests with test framework |
| `package` | Creates JAR/WAR |
| `verify` | Runs integration tests |
| `install` | Copies to local repository (~/.m2) |
| `deploy` | Copies to remote repository |

```bash
mvn clean install           # clean, compile, test, package, install
mvn clean test              # compile + test
mvn package -DskipTests     # package without tests
mvn deploy -P production    # deploy with production profile
```

#### Dependency Scope

| Scope | Description | Transitive? |
|-------|-------------|-------------|
| compile | Available everywhere | Yes |
| provided | Provided by runtime (e.g., Servlet API) | No |
| runtime | Not needed at compile, needed at runtime | Yes |
| test | Only for test execution | No |

#### Gradle build.gradle (Groovy DSL)

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.0'
    id 'io.spring.dependency-management' version '1.1.4'
    id 'jacoco'
}

group = 'com.example'
version = '1.0.0-SNAPSHOT'
sourceCompatibility = '21'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    runtimeOnly 'org.postgresql:postgresql'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

test {
    useJUnitPlatform()
    finalizedBy jacocoTestReport
}
```

#### Gradle build.gradle.kts (Kotlin DSL)

```kotlin
plugins {
    java
    id("org.springframework.boot") version "3.2.0"
    id("io.spring.dependency-management") version "1.1.4"
    jacoco
}

group = "com.example"
version = "1.0.0-SNAPSHOT"

repositories { mavenCentral() }

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    testImplementation("org.springframework.boot:spring-boot-starter-test")
}

tasks.test { useJUnitPlatform() }
```

```bash
./gradlew build              # compile, test, package
./gradlew clean              # delete build directory
./gradlew test               # run tests
./gradlew bootRun            # run Spring Boot app
./gradlew dependencies       # show dependency tree
```

#### Gradle vs Maven

| Feature | Maven | Gradle |
|---------|-------|--------|
| Configuration | XML (verbose) | Groovy/Kotlin DSL (concise) |
| Performance | Fixed lifecycle (slower) | Incremental, build cache (faster) |
| Convention | Strict conventions | Flexible |
| Build speed | Slower | 2-10x faster |
| Learning curve | Moderate | Steeper |

#### Build vs Compile vs Package vs Install

```bash
# Compile
mvn compile
gradle compileJava

# Package (compile + test + package)
mvn package
gradle build

# Install (package + copy to local repo)
mvn install
gradle publishToMavenLocal

# Deploy (install + copy to remote repo)
mvn deploy
gradle publish
```

### Internal Working

**Maven dependency resolution:**
1. Check local repository (~/.m2/repository)
2. If not found, check remote repositories
3. Download POM of each dependency
4. Recursively resolve transitive dependencies
5. Resolve conflicts (nearest wins)

**Gradle incremental builds:**
1. Track inputs and outputs
2. Cache task results by fingerprint
3. If inputs unchanged -> UP-TO-DATE
4. If cached result exists -> FROM-CACHE

### Common Mistakes

| Mistake | Impact |
|---------|--------|
| Not specifying plugin versions | Unpredictable builds |
| Using SNAPSHOT in production | Non-reproducible builds |
| Dependency conflicts | ClassLoader issues |
| Mixed scopes | Wrong classpath |

### Best Practices

- Lock dependency versions (use BOM or version catalog)
- Use SNAPSHOT only during development
- Use multi-module projects for large codebases
- Set Java source/target version explicitly
- Configure JaCoCo for test coverage

### Interview Questions

1. **Maven vs Gradle?** -- Maven for convention/stability; Gradle for performance/flexibility.
2. **What are Maven phases?** -- validate, compile, test, package, verify, install, deploy.
3. **Difference between package and install?** -- Package creates artifact; Install copies to local repo.
4. **How does dependency resolution work?** -- Nearest wins; exclusions or dependencyManagement.
5. **What is a BOM?** -- Bill of Materials for centralized version management.

### Practical Exercises

1. Create a Maven project with JUnit and Logback dependencies.
2. Add JaCoCo plugin and generate coverage report.
3. Create a multi-module Gradle project.
4. Use `mvn dependency:tree` to resolve a conflict.

### Mini Project: Multi-Module Build Setup

```xml
<!-- Parent pom.xml -->
<project>
    <groupId>com.example.petclinic</groupId>
    <artifactId>petclinic-parent</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>

    <modules>
        <module>petclinic-common</module>
        <module>petclinic-domain</module>
        <module>petclinic-service</module>
        <module>petclinic-web</module>
    </modules>
</project>
```

```groovy
// settings.gradle
rootProject.name = 'petclinic'
include 'petclinic-common', 'petclinic-domain', 'petclinic-service', 'petclinic-web'
```

### Revision Notes

- **Maven**: Convention-over-configuration; XML; strict lifecycle
- **Gradle**: DSL-based; incremental builds; flexible; faster
- **Coordinates**: groupId:artifactId:version
- **Scope**: compile, provided, runtime, test
- **Phases**: validate, compile, test, package, verify, install, deploy

---
## Chapter 11: Testing (JUnit 5 + Mockito)

### Introduction

Testing is the practice of verifying code correctness. JUnit 5 is the standard testing framework, and Mockito is the leading mocking library.

### Why It Matters

Testing ensures:
- Code works as expected
- Regression protection
- Documentation via examples
- Design feedback (testable code is well-designed)

### Real World Analogy

Think of testing as **safety inspections**:
- **Unit Test**: Inspecting each individual component
- **Mock**: A test dummy for components not yet built
- **Integration Test**: Testing assembled parts together
- **Coverage**: Percentage of components inspected

### Theory

#### JUnit 5 Annotations

```java
import org.junit.jupiter.api.*;

class CalculatorTest {

    @BeforeAll
    static void setupAll() {
        System.out.println("Runs once before all tests");
    }

    @BeforeEach
    void setup() {
        System.out.println("Runs before each test");
    }

    @Test
    void shouldAddTwoNumbers() {
        Calculator calc = new Calculator();
        int result = calc.add(2, 3);
        assertEquals(5, result);
    }

    @Test
    void shouldHandleNegativeNumbers() {
        Calculator calc = new Calculator();
        assertEquals(-1, calc.add(2, -3));
    }

    @AfterEach
    void tearDown() {
        System.out.println("Runs after each test");
    }

    @AfterAll
    static void tearDownAll() {
        System.out.println("Runs once after all tests");
    }
}
```

#### Assertions

```java
import static org.junit.jupiter.api.Assertions.*;

@Test
void testAssertions() {
    assertEquals(4, 2 + 2);
    assertNotEquals(5, 2 + 2);
    assertTrue(10 > 5);
    assertFalse(10 < 5);
    assertNull(null);
    assertNotNull("hello");
    assertThrows(IllegalArgumentException.class, () -> {
        new Account(null);
    });
    assertDoesNotThrow(() -> new Account("valid"));
    assertArrayEquals(new int[]{1, 2, 3}, new int[]{1, 2, 3});
    assertLinesMatch(List.of("a", "b"), List.of("a", "b"));
}
```

#### Assumptions

```java
@Test
void testOnlyOnWindows() {
    assumeTrue(System.getProperty("os.name").contains("Windows"));
    // Test only runs on Windows
}

@Test
void testWithEnvironment() {
    assumeFalse(System.getenv("CI") != null);
    // Skipped in CI
}
```

#### Parameterized Tests

```java
@ParameterizedTest
@ValueSource(ints = {1, 2, 3, 4, 5})
void shouldBePositive(int number) {
    assertTrue(number > 0);
}

@ParameterizedTest
@CsvSource({
    "2, 3, 5",
    "0, 0, 0",
    "-1, 1, 0"
})
void shouldAddCorrectly(int a, int b, int expected) {
    assertEquals(expected, new Calculator().add(a, b));
}

@ParameterizedTest
@MethodSource("provideUsers")
void shouldProcessUser(User user) {
    assertNotNull(user.name());
}

static Stream<Arguments> provideUsers() {
    return Stream.of(
        Arguments.of(new User("Alice", 30)),
        Arguments.of(new User("Bob", 25))
    );
}
```

#### Nested Tests

```java
class BankAccountTest {

    @Nested
    class DepositTests {
        @Test
        void shouldIncreaseBalance() {
            BankAccount acc = new BankAccount(100);
            acc.deposit(50);
            assertEquals(150, acc.getBalance());
        }

        @Test
        void shouldRejectNegativeDeposit() {
            BankAccount acc = new BankAccount(100);
            assertThrows(IllegalArgumentException.class, () -> acc.deposit(-10));
        }
    }

    @Nested
    class WithdrawTests {
        @Test
        void shouldDecreaseBalance() {
            BankAccount acc = new BankAccount(100);
            acc.withdraw(40);
            assertEquals(60, acc.getBalance());
        }

        @Test
        void shouldRejectOverdraft() {
            BankAccount acc = new BankAccount(100);
            assertThrows(InsufficientFundsException.class, () -> acc.withdraw(200));
        }
    }
}
```

#### Mockito Basics

```java
import static org.mockito.Mockito.*;

public class OrderServiceTest {

    @Mock
    private InventoryService inventoryService;

    @Mock
    private PaymentService paymentService;

    @InjectMocks
    private OrderService orderService;

    @BeforeEach
    void setup() {
        MockitoAnnotations.openMocks(this);
    }

    @Test
    void shouldPlaceOrderWhenStockAvailable() {
        // Given
        Cart cart = new Cart(List.of("item1", "item2"));
        when(inventoryService.checkStock(cart)).thenReturn(true);
        when(paymentService.processPayment(anyDouble())).thenReturn(new PaymentResult(true));

        // When
        Order order = orderService.placeOrder(cart);

        // Then
        assertNotNull(order);
        assertEquals(OrderStatus.CONFIRMED, order.status());
        verify(inventoryService).checkStock(cart);
        verify(paymentService).processPayment(anyDouble());
    }

    @Test
    void shouldThrowWhenStockUnavailable() {
        // Given
        Cart cart = new Cart(List.of("item1"));
        when(inventoryService.checkStock(cart)).thenReturn(false);

        // When/Then
        assertThrows(OutOfStockException.class, () -> orderService.placeOrder(cart));
        verify(inventoryService).checkStock(cart);
        verify(paymentService, never()).processPayment(anyDouble());
    }
}
```

#### ArgumentCaptor

```java
@Test
void shouldSendEmailWithCorrectContent() {
    // Given
    EmailService emailService = mock(EmailService.class);
    NotificationService notificationService = new NotificationService(emailService);

    // When
    notificationService.sendWelcomeEmail("user@example.com");

    // Then
    ArgumentCaptor<Email> captor = ArgumentCaptor.forClass(Email.class);
    verify(emailService).send(captor.capture());

    Email sent = captor.getValue();
    assertEquals("Welcome!", sent.subject());
    assertEquals("user@example.com", sent.to());
    assertTrue(sent.body().contains("Welcome to our platform"));
}
```

#### Mockito Verify

```java
@Test
void shouldVerifyInteractions() {
    List<String> mockList = mock(List.class);

    mockList.add("one");
    mockList.add("two");
    mockList.clear();

    verify(mockList).add("one");               // specific call
    verify(mockList).add("two");
    verify(mockList).clear();

    verify(mockList, times(2)).add(any());     // called 2 times
    verify(mockList, atLeastOnce()).clear();   // at least once
    verify(mockList, never()).remove(any());   // never called
    verifyNoMoreInteractions(mockList);         // no other calls
}
```

#### Mockito Stubbing

```java
// Basic stubbing
when(mock.method()).thenReturn(value);
when(mock.method()).thenThrow(new RuntimeException());

// Multiple calls
when(mock.method()).thenReturn("first", "second", "third");
// or
when(mock.method()).thenReturn("first").thenReturn("second");

// Void methods
doThrow(new RuntimeException()).when(mock).voidMethod();
doNothing().when(mock).voidMethod();
doAnswer(invocation -> {
    Object arg = invocation.getArgument(0);
    return "Processed " + arg;
}).when(mock).method(any());

// Spying (partial mocking)
List<String> spyList = spy(new ArrayList<>());
spyList.add("one");
when(spyList.size()).thenReturn(100);  // override real method
```

#### Test Coverage (JaCoCo)

```xml
<!-- Maven -->
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.11</version>
    <executions>
        <execution><goals><goal>prepare-agent</goal></goals></execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals><goal>report</goal></goals>
        </execution>
        <execution>
            <id>check</id>
            <goals><goal>check</goal></goals>
            <configuration>
                <rules>
                    <rule><element>CLASS</element>
                        <limits>
                            <limit><counter>LINE</counter><value>COVEREDRATIO><minimum>0.80</minimum></limit>
                        </limits>
                    </rule>
                </rules>
            </configuration>
        </execution>
    </executions>
</plugin>
```

> ✅ **Best Practice:** Write tests as specifications of behavior, not implementations. Test the *what*, not the *how*. This means tests survive refactoring — if you rename a private method or change internal implementation, tests shouldn't break. Mock only external boundaries (databases, APIs), never internal collaborators.

### Best Practices

- One test class per production class
- Use descriptive test method names (shouldXxxWhenYyy)
- Follow AAA pattern: Arrange, Act, Assert
- Test edge cases (null, empty, boundaries)
- Avoid testing private methods (test through public API)
- Use mocks only for external dependencies
- Keep tests independent and repeatable
- Aim for >80% coverage on core logic

### Interview Questions

1. **What is the difference between JUnit 4 and 5?** -- JUnit 5 has modular architecture, new annotations, parameterized tests, extensions.
2. **What is Mockito?** -- Mocking framework for creating test doubles.
3. **Difference between @Mock and @InjectMocks?** -- @Mock creates a mock; @InjectMocks injects mocks into the tested object.
4. **What is a spy?** -- Partial mock; real methods called unless stubbed.
5. **What is the AAA pattern?** -- Arrange (setup), Act (invoke), Assert (verify).
6. **What is test coverage?** -- Percentage of code executed by tests. JaCoCo measures line, branch, and method coverage.

### Practical Exercises

1. Write unit tests for a Calculator class (add, subtract, multiply, divide).
2. Mock a database repository and test a service layer.
3. Use ArgumentCaptor to verify email content.
4. Write parameterized tests for a StringUtils class.
5. Configure JaCoCo and generate a coverage report.

### Mini Project: Complete Test Suite

```java
public class UserService {
    private final UserRepository repository;
    private final EmailService emailService;

    public UserService(UserRepository repository, EmailService emailService) {
        this.repository = repository;
        this.emailService = emailService;
    }

    public User createUser(String name, String email) {
        if (name == null || name.isBlank()) throw new IllegalArgumentException("Name required");
        if (email == null || !email.contains("@")) throw new IllegalArgumentException("Invalid email");

        User user = new User(name, email);
        User saved = repository.save(user);
        emailService.sendWelcomeEmail(saved);
        return saved;
    }
}

class UserServiceTest {
    @Mock UserRepository repository;
    @Mock EmailService emailService;
    @InjectMocks UserService userService;

    @BeforeEach void setup() { MockitoAnnotations.openMocks(this); }

    @Test void shouldCreateUserSuccessfully() {
        User user = new User("Alice", "alice@example.com");
        when(repository.save(any())).thenReturn(user);

        User result = userService.createUser("Alice", "alice@example.com");

        assertNotNull(result);
        assertEquals("Alice", result.name());
        verify(repository).save(any());
        verify(emailService).sendWelcomeEmail(user);
    }

    @Test void shouldRejectBlankName() {
        assertThrows(IllegalArgumentException.class,
            () -> userService.createUser("", "alice@example.com"));
        verifyNoInteractions(repository, emailService);
    }

    @Test void shouldRejectInvalidEmail() {
        assertThrows(IllegalArgumentException.class,
            () -> userService.createUser("Alice", "not-an-email"));
        verifyNoInteractions(repository, emailService);
    }

    @ParameterizedTest
    @CsvSource({", alice@example.com, Name required",
                "'', alice@example.com, Name required",
                "Alice, not-email, Invalid email"})
    void shouldRejectInvalidInputs(String name, String email, String expectedMsg) {
        Exception ex = assertThrows(IllegalArgumentException.class,
            () -> userService.createUser(name, email));
        assertTrue(ex.getMessage().contains(expectedMsg));
    }
}
```

### Revision Notes

- **JUnit 5**: @Test, @BeforeEach, @BeforeAll, @Nested, @ParameterizedTest
- **Assertions**: assertEquals, assertThrows, assertTrue, assertAll
- **Assumptions**: assumeTrue, assumeFalse (skip conditions)
- **Mockito**: @Mock, @InjectMocks, when/thenReturn, verify
- **ArgumentCaptor**: Capture arguments for verification
- **Coverage**: JaCoCo measures line/branch/method coverage
- **AAA**: Arrange, Act, Assert

---
## Chapter 12: Memory Management & GC

### Introduction

Java's automatic memory management (garbage collection) is one of its key features. The JVM handles allocation and deallocation of memory, freeing developers from manual memory management. However, understanding how it works is essential for writing performant applications and diagnosing memory issues.

### Why It Matters

Poor memory management leads to:
- **OutOfMemoryError**: Application crashes
- **Excessive GC pauses**: Application freezes
- **Memory leaks**: Gradual performance degradation
- **Inefficient memory use**: Wasted resources

Understanding GC helps you tune JVM parameters, choose the right GC algorithm, and write memory-efficient code.

### Real World Analogy

Think of memory management like **a city waste management system**:
- **Heap**: The city's land (where objects live)
- **Young Generation**: Residential areas (objects come and go quickly)
- **Old Generation**: Industrial areas (objects stay longer)
- **Metaspace**: City planning office (class metadata)
- **GC**: Garbage collection trucks
- **Minor GC**: Regular neighborhood trash pickup
- **Major GC**: City-wide cleanup
- **Full GC**: Complete city renovation (pauses all traffic)

### Theory

#### JVM Memory Model

```
+------------------------------------------------------------------+
|                         JVM Memory                               |
+------------------------------------------------------------------+
|  Heap                                   | Stack   | Metaspace    |
|  +----------------+-------------------+ | (per    | (class       |
|  | Young Gen      | Old Gen           | | thread) | metadata)    |
|  | +--+---+---+--+|                   | |         |              |
|  | |E|S0|S1|   | ||                   | | Frames  | Runtime      |
|  | |d|1 | 2 |   | ||                   | | (local  | Constant     |
|  | |e|  |   |   | ||                   | | vars,   | Pool         |
|  | +--+---+---+--+|                   | | operands|              |
|  +----------------+-------------------+ |         |              |
+------------------------------------------------------------------+
|  PC Registers (per thread) | Native Method Stacks (per thread)   |
+------------------------------------------------------------------+
```

**Heap Structure:**

| Region | Description | GC Events |
|--------|-------------|-----------|
| **Eden** | New objects allocated here | Minor GC |
| **Survivor 0 (S0)** | Objects that survived 1st GC | Copied during Minor GC |
| **Survivor 1 (S1)** | Objects that survived subsequent GCs | Copied during Minor GC |
| **Old Generation** | Long-lived objects promoted from Survivor | Major GC |
| **Metaspace** | Class metadata (replaced PermGen in Java 8) | Full GC |

#### Object Allocation and Aging

```java
// 1. Object allocated in Eden
Object obj = new Object();

// 2. After Minor GC, if still referenced, moved to S0 (age = 1)
// 3. After next Minor GC, if still referenced, moved to S1 (age = 2)
// 4. After -XX:MaxTenuringThreshold (default 15), promoted to Old Gen
// 5. Old Gen eventually collected by Major GC
```

#### GC Algorithms

| Algorithm | Description | Pros | Cons |
|-----------|-------------|------|------|
| **Mark-Sweep** | Mark live objects, sweep dead ones | Simple | Fragmentation |
| **Mark-Compact** | Mark, sweep, compact (defragment) | No fragmentation | Slower than sweep |
| **Copying** | Copy live objects to empty space | Fast allocation, no fragmentation | Requires double space |

#### GC Types

```bash
# Serial GC (single thread, client-class)
-XX:+UseSerialGC

# Parallel GC (multi-thread, throughput-oriented)
-XX:+UseParallelGC
-XX:ParallelGCThreads=4

# CMS (low-latency, deprecated in Java 9, removed in Java 14)
-XX:+UseConcMarkSweepGC

# G1 GC (default since Java 9, balanced)
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200    # target pause time
-XX:G1HeapRegionSize=4m     # region size (1-32MB)

# ZGC (ultra-low latency, Java 15+)
-XX:+UseZGC
-XX:ZAllocationSpikeTolerance=2.0

# Shenandoah (low-pause, Java 12+)
-XX:+UseShenandoahGC
```

**GC Comparison:**

| GC | Pause Time | Throughput | Heap Size | Use Case |
|----|------------|------------|-----------|----------|
| Serial | Long | Low | <100MB | Single-threaded, small apps |
| Parallel | Medium | High | 4-8GB | Batch processing, data analytics |
| G1 | Short (configurable) | High | 4-100GB | Web servers, enterprise apps |
| ZGC | <10ms (sub-millisecond) | High | 8MB-16TB | Low-latency, large heaps |
| Shenandoah | <10ms | High | Large | Low-latency, concurrent compaction |

> 💡 **Pro Tip:** Don't optimize GC prematurely. Start with G1GC (default since Java 9) and tune only when you observe problems. Use `-Xlog:gc:gc.log` to log GC events and visualize with tools like GCeasy or GCViewer. The most impactful optimization is reducing object allocation rate, not tuning GC parameters.

#### Memory Leaks in Java

```java
// Memory Leak 1: Static collections
public class LeakyClass {
    private static final List<Object> cache = new ArrayList<>();

    public void add(Object obj) {
        cache.add(obj);  // Never removed -- memory leak!
    }
}

// Memory Leak 2: Unclosed resources
public void processFile(String path) throws IOException {
    InputStream in = new FileInputStream(path);
    // Process -- but never closed!
    // Solution: try-with-resources
}

// Memory Leak 3: Incorrect equals/hashCode
public class Person {
    private String name;

    // If equals/hashCode not overridden, HashMap will keep duplicates
    // and never find existing entries for removal
}

// Memory Leak 4: Inner class capturing outer reference
public class Outer {
    private String data;

    public void process() {
        Runnable r = new Runnable() {
            public void run() {
                System.out.println(data);  // Captures Outer.this
            }
        };
        new Thread(r).start();
    }
}

// Memory Leak 5: ThreadLocal misuse
public class ThreadLocalLeak {
    private static final ThreadLocal<byte[]> tl = ThreadLocal.withInitial(() -> new byte[1024 * 1024]);
    // If thread is not terminated (e.g., thread pool), memory stays forever
    // Solution: tl.remove() in finally block
}
```

#### JVM Tuning Flags

```bash
# Heap Sizing
-Xms512m                    # Initial heap size
-Xmx4g                      # Maximum heap size
-Xmn512m                    # Young generation size
-XX:NewRatio=3              # Old:Young = 3:1
-XX:SurvivorRatio=8         # Eden:Survivor = 8:1:1

# Metaspace
-XX:MetaspaceSize=256m      # Initial metaspace
-XX:MaxMetaspaceSize=512m   # Maximum metaspace

# GC Logging
-XX:+PrintGCDetails
-XX:+PrintGCDateStamps
-Xloggc:gc.log
-XX:+PrintHeapAtGC
-XX:+PrintTenuringDistribution

# GC Tuning
-XX:MaxGCPauseMillis=200
-XX:GCTimeRatio=9           # 90% throughput, 10% GC time
-XX:ParallelGCThreads=4
-XX:ConcGCThreads=2
-XX:MaxTenuringThreshold=15 # Age before promotion

# OOM Handling
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/path/to/dumps/
-XX:+ExitOnOutOfMemoryError

# Performance
-XX:+AlwaysPreTouch         # Pre-allocate physical memory
-XX:+UseStringDeduplication  # Deduplicate identical strings
-XX:StringTableSize=1000003  # Increase string pool size

# Example production settings
java -Xms4g -Xmx4g -Xmn1g      -XX:+UseG1GC      -XX:MaxGCPauseMillis=200      -XX:+HeapDumpOnOutOfMemoryError      -XX:HeapDumpPath=/var/log/app/heapdump.hprof      -Xloggc:/var/log/app/gc.log      -XX:+PrintGCDetails      -jar myapp.jar
```

#### JVM Monitoring Tools

```bash
# jps -- list Java processes
jps -l

# jstat -- GC statistics (live)
jstat -gc <pid> 1000           # every 1 second
jstat -gcutil <pid> 1000       # percentage usage
jstat -gccapacity <pid>        # generation capacities

# jmap -- heap dump
jmap -heap <pid>                # heap summary
jmap -histo <pid>               # object histogram
jmap -dump:live,format=b,file=heap.hprof <pid>  # heap dump

# jstack -- thread dump
jstack <pid>                    # all threads
jstack -l <pid>                 # with lock information

# jcmd -- Swiss army knife
jcmd <pid> help                 # available commands
jcmd <pid> GC.heap_info        # heap info
jcmd <pid> Thread.print        # thread dump
jcmd <pid> VM.native_memory    # native memory (requires -XX:NativeMemoryTracking=summary)

# jvisualvm -- visual monitor (separate tool)
# VisualVM provides real-time monitoring, heap dump analysis, CPU profiling

# JMX Monitoring
# Add to JVM args:
-Dcom.sun.management.jmxremote
-Dcom.sun.management.jmxremote.port=9010
-Dcom.sun.management.jmxremote.ssl=false
-Dcom.sun.management.jmxremote.authenticate=false
```

### Internal Working

**How G1 GC works:**
1. Heap divided into 1-32MB regions
2. Young GC: Eden regions collected, live objects copied to Survivor regions
3. Concurrent marking: Mark live objects concurrently
4. Mixed GC: Collect both young and old regions with most garbage first
5. All operations aim for MaxGCPauseMillis target

**How ZGC achieves sub-millisecond pauses:**
1. Colored pointers (bits in pointer for metadata)
2. Load barriers (intercept object references)
3. Concurrent compaction (no stop-the-world)
4. Can handle up to 16TB heaps with <10ms pauses

### Step-by-Step: Diagnosing a Memory Leak

```bash
# Step 1: Find the Java process
jps -l

# Step 2: Check heap usage over time
jstat -gcutil <pid> 2s

# Step 3: Take a heap dump
jmap -dump:live,format=b,file=leak.hprof <pid>

# Step 4: Analyze with Eclipse MAT or VisualVM
# MAT: Look for "Suspects" report
# VisualVM: Load heap dump, find largest objects

# Step 5: Check GC logs
# Look for increasing heap usage after Full GC
# Look for promotion failures, concurrent mode failures
```

### Common Mistakes

| Mistake | Impact | Solution |
|---------|--------|----------|
| No -Xmx set | Default too small for production | Set -Xmx based on workload |
| Using System.gc() | Forces Full GC unnecessarily | Let JVM decide |
| String concatenation in loops | Creates many intermediate strings | Use StringBuilder |
| Holding references longer than needed | Memory leaks | Null out references, use WeakHashMap |
| Large objects in young gen | Premature promotion | Allocate directly in old gen if needed |
| Ignoring GC logs | Can't diagnose issues | Always enable GC logging |
| Not setting MaxMetaspaceSize | Metaspace can grow unbounded | Set explicit limit |

### Best Practices

- Set -Xms and -Xmx to same value (prevents resizing)
- Use G1 GC for most server applications (default since Java 9)
- Enable GC logging in production
- Take heap dumps on OutOfMemoryError
- Monitor GC pause times in production
- Use connection pools with limits
- Avoid finalizers (deprecated in Java 9, removed in Java 18)
- Use try-with-resources for all Closeables
- Profile before optimizing (don't guess)

### Interview Questions

1. **How does Java manage memory?** -- JVM manages heap; GC collects unreachable objects.
2. **What is the difference between stack and heap?** -- Stack stores local variables and references; heap stores objects.
3. **Explain the JVM memory model.** -- Heap (Young + Old + Metaspace), Stack (per thread), PC Registers, Native Method Stacks.
4. **What is a stop-the-world event?** -- All application threads paused for GC.
5. **What is the difference between Minor, Major, and Full GC?** -- Minor: Young gen; Major: Old gen; Full: Both + Metaspace.
6. **How does G1 GC work?** -- Heap in regions; concurrent marking; mixed collections; pause time target.
7. **What is a memory leak in Java?** -- Objects no longer needed but still referenced, preventing GC.
8. **What JVM flags would you use for a production server?** -- -Xms, -Xmx, -XX:+UseG1GC, -XX:MaxGCPauseMillis, GC logging, HeapDumpOnOutOfMemoryError.
9. **What is the difference between Parallel and G1 GC?** -- Parallel focuses on throughput; G1 focuses on bounded pause times.
10. **How do you diagnose a memory leak?** -- jstat for trends, jmap for heap dump, MAT/VisualVM for analysis.

### Practical Exercises

1. Write a program that creates a memory leak (static collection growing) and observe with jstat.
2. Enable GC logging and analyze the output.
3. Take a heap dump of a running application and analyze with VisualVM.
4. Tune G1 GC parameters for a sample application and measure pause times.
5. Use jstack to capture thread dumps and identify deadlock.

### Mini Project: Memory Leak Detector

```java
import java.lang.management.*;
import javax.management.*;

public class MemoryMonitor {
    private final MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
    private final List<MemoryUsage> history = new ArrayList<>();

    public void sample() {
        MemoryUsage heap = memoryBean.getHeapMemoryUsage();
        MemoryUsage nonHeap = memoryBean.getNonHeapMemoryUsage();
        history.add(heap);
        log("Heap: used=%dMB, max=%dMB, committed=%dMB",
            heap.getUsed() / 1024 / 1024,
            heap.getMax() / 1024 / 1024,
            heap.getCommitted() / 1024 / 1024);
    }

    public void detectLeak(long thresholdMB) {
        if (history.size() < 2) return;
        MemoryUsage first = history.get(0);
        MemoryUsage last = history.get(history.size() - 1);
        long growth = last.getUsed() - first.getUsed();
        if (growth > thresholdMB * 1024 * 1024) {
            log.warn("Potential memory leak: heap grown by %dMB since start", growth / 1024 / 1024);
        }
    }

    public void startPeriodicSampling(long intervalMs) {
        ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);
        scheduler.scheduleAtFixedRate(() -> {
            sample();
            detectLeak(100);
        }, 0, intervalMs, TimeUnit.MILLISECONDS);
    }

    // Usage in application
    public static void main(String[] args) {
        MemoryMonitor monitor = new MemoryMonitor();
        monitor.startPeriodicSampling(5000);
        // Run application...
    }
}
```

### Revision Notes

- **JVM Memory**: Heap (Young: Eden + S0 + S1, Old), Metaspace, Stack (per thread), PC Registers
- **GC Algorithms**: Mark-Sweep, Mark-Compact, Copying
- **GC Types**: Serial, Parallel, G1 (default), ZGC, Shenandoah
- **G1 GC**: Heap regions, concurrent marking, mixed collections, pause time target
- **ZGC**: Colored pointers, load barriers, <10ms pauses
- **Memory Leaks**: Static collections, unclosed resources, inner classes, ThreadLocal misuse
- **JVM Flags**: -Xms, -Xmx, -XX:+UseG1GC, -XX:MaxGCPauseMillis
- **Tools**: jps, jstat, jmap, jstack, jcmd, jvisualvm, Eclipse MAT
- **GC Logging**: -Xloggc, -XX:+PrintGCDetails, -XX:+PrintGCDateStamps
- **Heap Dump**: -XX:+HeapDumpOnOutOfMemoryError, -XX:HeapDumpPath

---

