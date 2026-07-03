# Part 10: Python — The Lingua Franca of AI and Full-Stack Development

Python is the single most important language in the modern AI stack. From scripting infrastructure to training deep learning models, from backend web servers to data pipelines, Python dominates. This part covers the language from first principles to production-grade patterns.

---

## Chapter 58: Python Fundamentals

### Introduction

Python is a high-level, interpreted, dynamically typed, garbage-collected programming language created by Guido van Rossum and first released in 1991. Its design philosophy emphasizes code readability and simplicity, famously expressed in the Zen of Python.

### Why It Matters

Python occupies a unique position: it is both an approachable first language and a powerful tool for production systems. It runs Instagram's backend, powers YouTube, drives Netflix's recommendation engine, and is the primary language for machine learning frameworks (PyTorch, TensorFlow, JAX). Understanding Python deeply — not just how to write it, but how it works internally — separates competent developers from exceptional ones.

### Real World Analogy

Python is like a well-organized workshop. The tools (standard library) are all in predictable places. The workbench (interpreter) is sturdy and consistent. You can build anything from a toothpick (one-liner script) to a house (Django monolith) with the same set of fundamental tools. But knowing how the workshop is wired — how memory is managed, how the toolbelt is organized — lets you work faster and avoid burning the place down.

### Theory and Internal Working

#### Python History

| Version | Released | Key Changes |
|---------|----------|-------------|
| Python 1.0 | Jan 1994 | Initial release, modutils, built-in types |
| Python 2.0 | Oct 2000 | List comprehensions, garbage collection, Unicode |
| Python 2.6 | Oct 2008 | Forward compatibility with 3.0 |
| Python 2.7 | Jul 2010 | Last of 2.x, EOL Jan 2020 |
| Python 3.0 | Dec 2008 | print as function, Unicode strings, // integer division |
| Python 3.5 | Sep 2015 | async/await, type hints, `@` matrix multiply |
| Python 3.6 | Dec 2016 | f-strings, dict ordering, async generators |
| Python 3.7 | Jun 2018 | dataclasses, `breakpoint()`, guaranteed dict ordering |
| Python 3.8 | Oct 2019 | walrus operator `:=`, positional-only params `/` |
| Python 3.9 | Oct 2020 | dict union `|`, `zoneinfo`, `str.removeprefix` |
| Python 3.10 | Oct 2021 | `match`/`case`, parenthesized context managers, `X | Y` union types |
| Python 3.11 | Oct 2022 | Zero-cost exceptions, faster CPython (~25% faster), `Self` type |
| Python 3.12 | Oct 2023 | `type` statement, `@override`, immortal objects, better error messages |
| Python 3.13 | Oct 2024 | `nogil` experimental builds, improved JIT, `@deprecated` in warnings |

#### The Python Interpreter

When you run `python script.py`, the following happens:

1. **Lexing**: The source text is tokenized into a stream of tokens
2. **Parsing**: Tokens are parsed into an Abstract Syntax Tree (AST) via the `Parser/` C code
3. **Compilation**: The AST is compiled into bytecode — a low-level, platform-independent representation stored in `__pycache__/*.pyc`
4. **Execution**: The CPython virtual machine executes the bytecode in a `PyEval_EvalFrameDefault` loop (the "eval loop")

```
Source Code → Tokens → AST → Bytecode → CPython VM → Output
    (lexer)    (parser)  (compiler)  (interpreter)
```

The `.pyc` files are cached bytecode. Python checks the modification timestamp of the `.py` source against the cached `.pyc`. If the source is newer, it recompiles. The `__pycache__` directory organizes these by Python version (`script.cpython-311.pyc`).

#### CPython vs PyPy vs Jython vs IronPython

| Implementation | Description | Performance | Use Case |
|---------------|-------------|-------------|----------|
| **CPython** | Reference implementation in C | Baseline | Production standard |
| **PyPy** | Python-in-Python with JIT | 2-10x faster for long-running CPU tasks | Long-running services, numeric code |
| **Jython** | Python on JVM | Slow, limited Python 2 | Java integration |
| **IronPython** | Python on .NET DLR | Similar to CPython | .NET ecosystem |
| **GraalPy** | Python on GraalVM JIT | Fast, Java interop | Polyglot JVM environments |
| **Codon** | Python-to-native compiler | 10-100x faster | HPC, bioinformatics |
| **Numba** | JIT for numeric Python subset | Near-C speeds | NumPy-heavy loops |

CPython is what 99% of developers use. It is an interpreter, not a compiler — though Python 3.13+ ships with an experimental JIT compiler (copy-and-patch JIT based on LLVM).

#### The Zen of Python

Type `import this` in any Python interpreter:

```
Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one — and preferably only one — obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea — let's do more of those!
```

This is not poetry. It is a design manifesto. Every line encodes a rule. "Explicit is better than implicit" is why Python requires `self` in method definitions. "Flat is better than nested" is why `import this` works without deep package qualification.

> 📖 **Instructor Note:** The Zen of Python isn't just philosophical — it's a practical decision-making tool. When facing design ambiguity, ask which option is more explicit, flatter, or simpler. These heuristics have guided Python's evolution for decades.

### Variables and Dynamic Typing

Python variables are **names bound to objects**. They are not memory boxes; they are sticky notes attached to values.

```python
x = 1000        # x → PyObject (int) at address 0x7f...
y = x           # y → same PyObject (NOT a copy)
x = 2000        # x → new PyObject (int); y still → 1000
```

Every Python object has:
- **ob_refcnt**: reference count (for garbage collection)
- **ob_type**: pointer to type object
- **ob_size**: variable-length object size (for str, list, etc.)

```python
import sys
a = 42
sys.getrefcount(a)  # typically ~50+ due to small integer caching
```

Python caches small integers (-5 to 256) — they are singleton objects. Every assignment of `a = 5` and `b = 5` points to the same PyObject.

> 💡 **Pro Tip:** The small integer cache is a common interview topic. Because -5 to 256 are singletons, `a is b` returns `True` for them, but `a is 1000` may return `False` even if both are 1000. Always use `==` for value comparison — `is` checks identity, not equality.

### Data Types Deep Dive

#### int (Unbounded)

Python 3 `int` is an arbitrary-precision integer. It is implemented as an array of "digits" (30-bit or 15-bit limbs on 64-bit vs 32-bit systems). No overflow.

```python
import sys
x = 2**1000
print(len(str(x)))  # 302 digits
sys.getsizeof(0)     # 28 bytes (base overhead)
sys.getsizeof(2**10000)  # grows with magnitude ~4 bytes per 30 bits
```

Internally, `PyLongObject` has `ob_digit[1]` — flexible array member for the actual digits. Operations use `_PyLong_Add`, `_PyLong_Multiply` which call `long_add`, `long_mul` — these use careful algorithms (Karatsuba for large multiplication, etc.).

#### float (IEEE 754)

Python `float` maps to C `double` — 64-bit IEEE 754. It has 53 bits of mantissa (~15-17 decimal digits), 11 exponent bits, 1 sign bit.

```python
import sys, math
sys.float_info  # sys.float_info(max=1.7976931348623157e+308, max_exp=1024, ...)
0.1 + 0.2       # 0.30000000000000004 — IEEE 754 precision error
math.isclose(0.1 + 0.2, 0.3)  # True — use this for comparison
```

#### complex

```python
z = 3 + 4j
z.real      # 3.0
z.imag      # 4.0
z.conjugate()  # (3-4j)
abs(z)      # 5.0 (magnitude)
```

#### str (Unicode)

Python 3 strings are Unicode sequences, stored as one of:
- **Latin-1 (1 byte per char)**: if all codepoints < 256
- **UCS-2 (2 bytes per char)**: if any codepoint >= 256 but < 65536
- **UCS-4 (4 bytes per char)**: if any codepoint >= 65536

```python
import sys
sys.getsizeof("a")       # 50 bytes (PyObject + 1 byte + overhead)
sys.getsizeof("euro")    # 76 bytes (UCS-4)
sys.getsizeof("a" * 100) # 151 bytes (compact Latin-1)
```

#### bytes and bytearray

`bytes` is immutable, `bytearray` is mutable. Both are sequences of integers in range 0-255.

```python
b = b"hello"
ba = bytearray(b"hello")
ba[0] = 72  # mutable
b[0] = 72   # TypeError: 'bytes' object does not support item assignment
```

### Type Conversion and Coercion

```python
# Explicit conversion
int("42")       # 42
float("3.14")   # 3.14
str(42)         # "42"
bool(1)         # True
list("hello")   # ['h', 'e', 'l', 'l', 'o']

# Coercion in operations
1 + 2.0         # 3.0 (int → float)
True + 1        # 2 (bool → int — True is 1, False is 0)
```

### Operators

| Category | Operators | Notes |
|----------|-----------|-------|
| Arithmetic | `+ - * / // % **` | `//` floor division, `**` power |
| Comparison | `== != < > <= >=` | Chainable: `1 < x < 10` |
| Logical | `and or not` | Short-circuit evaluation |
| Bitwise | `& | ^ ~ << >>` | Operate on integer bits |
| Identity | `is`, `is not` | Object identity, not equality |
| Membership | `in`, `not in` | Container membership |
| Walrus | `:=` | Assignment expression (3.8+) |

```python
# Walrus operator
if (n := len(data)) > 100:
    print(f"Large dataset: {n} items")

# Chained comparison
age = 25
if 18 <= age < 65:
    print("Working age")
```

### Control Flow

#### match/case (Python 3.10+)

```python
def http_status(code):
    match code:
        case 200:
            return "OK"
        case 201 | 202:
            return "Created"
        case 400:
            return "Bad Request"
        case 404:
            return "Not Found"
        case _:
            return "Unknown"

# Structural pattern matching with objects
def process_point(point):
    match point:
        case (0, 0):
            return "Origin"
        case (x, 0):
            return f"On X axis at {x}"
        case (0, y):
            return f"On Y axis at {y}"
        case (x, y):
            return f"At ({x}, {y})"
        case _:
            return "Not a 2D point"

# Guard expressions
def classify(x):
    match x:
        case int() as n if n < 0:
            return "Negative integer"
        case int():
            return "Positive integer"
        case str():
            return "String"
```

#### Loop else clause

```python
# else runs when loop terminates NORMALLY (not via break)
for item in items:
    if condition(item):
        break
else:
    print("No item matched condition")
```

### Comprehensions

```python
# List comprehension
squares = [x**2 for x in range(10)]

# Dict comprehension
square_map = {x: x**2 for x in range(10)}

# Set comprehension
even_squares = {x**2 for x in range(10) if x % 2 == 0}

# Nested comprehension (matrix transpose)
matrix = [[1, 2, 3], [4, 5, 6]]
transpose = [[row[i] for row in matrix] for i in range(3)]

# Generator expression (lazy)
sum_of_squares = sum(x**2 for x in range(10_000_000))
```

> ✅ **Best Practice:** Use generator expressions instead of list comprehensions when you only need to iterate once. For `sum()`, `any()`, `all()`, and similar consumers, generator expressions avoid materializing the entire sequence, saving memory and improving cache performance.

### Functions

```python
def example(pos, /, pos_or_kwd, *, kwd_only, default=None):
    """
    Parameters:
      pos         — positional-only (before /)
      pos_or_kwd  — positional or keyword
      kwd_only    — keyword-only (after *)
      default     — keyword-only with default
    """
    ...

def process_items(*args, **kwargs):
    print(f"Args: {args}")
    print(f"Kwargs: {kwargs}")

process_items(1, 2, 3, name="Alice", role="engineer")
# Args: (1, 2, 3)
# Kwargs: {'name': 'Alice', 'role': 'engineer'}
```

### Scope and LEGB Rule

Python resolves names in this order:
1. **Local** — inside the current function
2. **Enclosing** — any enclosing function (nested functions)
3. **Global** — module level
4. **Built-in** — `builtins` module (`print`, `len`, `int`, etc.)

```python
x = "global"

def outer():
    x = "enclosing"
    
    def inner():
        x = "local"
        print(x)  # "local"
    
    inner()
    print(x)  # "enclosing"

outer()
print(x)  # "global"
```

The `global` and `nonlocal` keywords allow rebinding names in outer scopes:

```python
counter = 0

def increment():
    global counter
    counter += 1  # without global, this creates a LOCAL counter
```

### Recursion

Python has a default recursion limit of 1000 frames to prevent C stack overflow.

```python
import sys
sys.getrecursionlimit()   # 1000
sys.setrecursionlimit(5000)

def factorial(n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)
```

### Common Mistakes

| Mistake | Wrong | Correct |
|---------|-------|---------|
| Mutable default argument | `def f(x=[])` | `def f(x=None): x = x or []` |
| Using `is` for value comparison | `x is 1000` (may fail) | `x == 1000` |
| Modifying list while iterating | `for x in lst: lst.remove(x)` | `for x in lst[:]` |
| Late binding in closures | `[lambda: i for i in range(5)]` | `[lambda i=i: i for i in range(5)]` |
| String concatenation in loops | `s += x` in loop | `''.join(parts)` |
| Shadowing built-ins | `list = [1,2,3]` | `my_list = [1,2,3]` |
| Using `==` for NaN | `float('nan') == float('nan')` → False | `math.isnan(x)` |
| Forgetting `return` | `def f(): "docstring"` | `def f(): return "value"` |

### Best Practices

- Use `snake_case` for functions and variables, `PascalCase` for classes, `UPPER_CASE` for constants
- Prefer `''.join()` over `+` for concatenating many strings
- Use `with` for resource management (files, locks, connections)
- Prefer comprehensions over `map()`/`filter()` with lambdas
- Use type hints for all public function signatures
- Use `if x:` rather than `if x is True:` or `if len(x) > 0:`
- Prefer `for item in sequence` over `for i in range(len(sequence))`
- Use `enumerate()` when you need both index and value
- Always use `if __name__ == "__main__":` guard for script entry points

### Interview Questions

**Q: What is the difference between `is` and `==`?**
A: `==` checks value equality (calls `__eq__`). `is` checks identity (same object, compares `id()`). Small integers and strings may be interned, causing `is` to appear to work.

**Q: How does Python manage memory?**
A: Python uses reference counting (primary) and generational garbage collection (cycle detection). Each object has `ob_refcnt`; when it hits 0, the object is deallocated immediately. The GC (`gc` module) handles reference cycles.

**Q: What is the GIL?**
A: The Global Interpreter Lock prevents multiple native threads from executing Python bytecode simultaneously. It protects CPython's memory model but limits CPU-bound multi-threading.

### Practical Exercises

1. Implement a Fibonacci calculator using iterators, generators, and recursion — compare performance
2. Write a function that deep-compares two nested dictionaries and reports differences
3. Re-implement `range()` using a generator class with `__iter__` and `__next__`
4. Benchmark `''.join()` vs `+=` for concatenating 10,000 strings

### Mini Project: Python REPL Tool

Build a simple REPL that evaluates mathematical expressions with variable support, history, and error handling.

### Revision Notes

- Python is dynamically typed, garbage-collected, interpreted (to bytecode)
- Everything is an object with reference count
- `int` is arbitrary precision; `float` is IEEE 754 double; `str` is Unicode (3 internal representations)
- LEGB rule for name resolution
- `match`/`case` for structural pattern matching (3.10+)
- Walrus operator `:=` for assignment expressions
- Comprehensions are Pythonic — prefer over `map`/`filter` for readability
- Mutable default arguments are evaluated once at function definition

---

## Chapter 59: Data Structures in Python

### Introduction

Python's built-in data structures are implemented in C for performance. Understanding their internals — how they allocate memory, how hash tables work, when resizing occurs — is essential for writing efficient code.

### Why It Matters

The difference between `O(n)` and `O(1)` operations can mean minutes vs milliseconds in production. Knowing which data structure to use for which pattern is the foundation of algorithm design.

### Real World Analogy

Python's data structures are like a well-stocked toolbox. Lists are adjustable wrenches — great for many tasks. Dictionaries are labeled drawers — instant lookup if you know the label. Sets are like a bouncer at a club — fast membership checking. Tuples are sealed envelopes — guaranteed to stay as-is.

### Lists (Dynamic Array)

Internally, `list` is a `PyListObject` containing:
- `ob_item` points to a C array of `PyObject*` pointers
- `allocated` is the current capacity; `ob_size` is the actual length
- On append: if `ob_size == allocated`, Python over-allocates: `new_allocated = (size_t)newsize + (newsize >> 3) + (newsize < 9 ? 3 : 6)`
  - This gives amortized O(1) append
  - Growth factor ~1.125 for large lists

```python
import sys
lst = []
sys.getsizeof(lst)  # 56 bytes (empty)

for i in range(10):
    lst.append(i)
    print(f"len={len(lst):2}, size={sys.getsizeof(lst):3} bytes")
```

#### Slicing

```python
lst = [0, 1, 2, 3, 4, 5]
lst[1:4]     # [1, 2, 3] — copies elements
lst[::-1]    # [5, 4, 3, 2, 1, 0] — reversed copy
lst[::2]     # [0, 2, 4] — every other element
lst[-3:]     # [3, 4, 5] — last 3 elements
```

Slicing creates a **shallow copy**. For large slices, it allocates a new `PyListObject` and copies all pointers.

#### Sorting

```python
lst.sort()          # in-place, returns None (TimSort, O(n log n))
sorted(lst)         # returns new sorted list
sorted(lst, key=len, reverse=True)  # key function, optional reverse

# bisect module maintains sorted order
import bisect
sorted_lst = [1, 3, 5, 7]
bisect.insort(sorted_lst, 4)  # [1, 3, 4, 5, 7]
pos = bisect.bisect_left(sorted_lst, 4)  # find insertion point
```

TimSort is Python's sorting algorithm — a hybrid of merge sort and insertion sort. It detects natural runs in data and merges them efficiently. Worst-case O(n log n), best-case O(n) on already-sorted data.

### Tuples

Tuples are immutable arrays stored as `PyTupleObject` — the array is stored inline for small tuples.

```python
t = (1, 2, 3)
t[0] = 42  # TypeError

# namedtuple
from collections import namedtuple
Point = namedtuple('Point', ['x', 'y'])
p = Point(10, 20)
p.x        # 10
x, y = p   # tuple unpacking
```

### Dictionaries (Hash Table)

Internally, `dict` is a hash table using **open addressing** with **quadratic probing**. Python 3.6+ preserves insertion order using a **compact dict** representation:
- Entries are stored in insertion order
- The hash table stores only indices into the entries array
- This reduces memory (~20-30%) and preserves order

| Operation | Average Case | Worst Case |
|-----------|-------------|------------|
| `get`/`set`/`del` | O(1) | O(n) — hash collision |
| `in` | O(1) | O(n) |
| Iteration | O(n) | O(n) |

> 💡 **Pro Tip:** Python 3.7+ guarantees dict insertion order. This means you can rely on dicts being ordered for serialization, testing, and data processing. Before 3.7, this was implementation-specific (CPython 3.6) but not guaranteed.

#### Specialized Dictionaries

```python
from collections import defaultdict, Counter, OrderedDict

# defaultdict — factory for missing keys
dd = defaultdict(list)
dd["a"].append(1)  # no KeyError

# Counter — count hashable items
c = Counter("mississippi")
c  # Counter({'i': 4, 's': 4, 'p': 2, 'm': 1})
c.most_common(2)  # [('i', 4), ('s', 4)]

# OrderedDict — remembers insertion order, move_to_end, popitem
od = OrderedDict.fromkeys("abcde")
od.move_to_end("a")
```

### Sets

Sets are hash tables with only keys (no values). Same hash collision resolution as dicts.

```python
s = {1, 2, 3, 4, 5}
s.add(6)
s.remove(1)

s1 = {1, 2, 3, 4}
s2 = {3, 4, 5, 6}
s1 | s2   # union: {1, 2, 3, 4, 5, 6}
s1 & s2   # intersection: {3, 4}
s1 - s2   # difference: {1, 2}
s1 ^ s2   # symmetric difference: {1, 2, 5, 6}

# frozenset — immutable, hashable
fs = frozenset([1, 2, 3])
d = {fs: "value"}  # works
```

### Strings

```python
s = "hello world"
s.split()                                # ['hello', 'world']
", ".join(["a", "b", "c"])              # "a, b, c"
s.replace("world", "python")

# f-strings (3.6+)
name, age = "Alice", 30
f"{name} is {age} years old"            # "Alice is 30 years old"
f"{value:.2f}"                           # formatted: "3.14"
f"{value:>10}"                           # right-aligned in 10 chars
```

### collections Module

```python
from collections import deque, ChainMap

# deque — double-ended queue, O(1) append/pop from both ends
dq = deque(maxlen=100)
dq.append(1)
dq.appendleft(2)
dq.pop()
dq.popleft()

# ChainMap — combine multiple dicts into a single view
defaults = {"theme": "dark", "lang": "en"}
user = {"theme": "light"}
settings = ChainMap(user, defaults)
settings["theme"]   # "light" (first match)
settings["lang"]    # "en" (from defaults)
```

> ⚠️ **Warning:** Shallow copies are a common source of subtle bugs. Slicing (`lst[:]`), `list.copy()`, and `dict.copy()` all create shallow copies — the outer container is new, but nested objects are shared. For deeply nested data, ALWAYS use `copy.deepcopy()` or be explicit about your copying intent.

### copy Module

```python
import copy

original = [[1, 2, 3], [4, 5, 6]]
shallow = copy.copy(original)    # new outer list, same inner lists
deep = copy.deepcopy(original)   # completely independent copy

shallow[0][0] = 99
original[0][0]  # 99 — shared inner list
deep[0][0]      # 1 — independent
```

### struct Module

```python
import struct

# Pack integers into bytes
data = struct.pack(">II", 1, 2)   # big-endian, two 4-byte unsigned ints

a, b = struct.unpack(">II", data)  # (1, 2)

# Format codes:
# > = big-endian, < = little-endian, ! = network
# I = unsigned int (4 bytes), H = unsigned short (2 bytes)
# f = float (4 bytes), d = double (8 bytes)
```

### Common Mistakes

| Mistake | Issue | Fix |
|---------|-------|-----|
| `list.remove(x)` in loop | Only removes first occurrence | List comprehension or while loop |
| `dict[key]` without checking | KeyError | `dict.get(key)` or `key in dict` |
| Modifying dict while iterating | RuntimeError: changed size | Iterate over `list(dict.items())` |
| Shallow copy confusion | `list2 = list1` doesn't copy | `list1[:]` or `copy.deepcopy()` |
| String concatenation with `+=` | O(n^2) performance | `''.join(list_of_strings)` |
| `defaultdict` with `int` | Missing keys return 0 | Verify intent before use |

### Best Practices

- Use `in` for membership checks (not `.index()`)
- Use `dict.get(key, default)` instead of try/except KeyError
- Use `dict.setdefault(key, [])` for dict-of-lists patterns
- Prefer `collections.defaultdict` over manual key checking
- Use `deque` for queues, not `list.pop(0)` (O(n))
- Use `sorted()` and `bisect` for maintaining sorted lists
- Use `is` for None checks: `if x is None`, not `if x == None`

### Interview Questions

**Q: How does Python's dict preserve insertion order?**
A: Python 3.6+ uses a compact dict representation. The hash table stores indices into a separate insertion-ordered entries array. This became a language guarantee in 3.7.

**Q: What is the time complexity of `list.append()`?**
A: Amortized O(1). When the capacity is reached, Python over-allocates by ~12.5%, making resizes increasingly rare.

**Q: How are sets implemented?**
A: As hash tables with only keys (no values), using open addressing with quadratic probing.

### Practical Exercises

1. Implement an LRU cache using `OrderedDict`
2. Implement a `Counter` from scratch using `defaultdict`
3. Compare `list.append` vs `deque.append` performance for 1 million items
4. Parse a binary file header using `struct`

### Mini Project: Prefix Tree (Trie)

Implement a Trie data structure for autocomplete using nested dictionaries.

### Revision Notes

- `list`: dynamic array, O(1) amortized append, O(n) insert/pop(0)
- `dict`: hash table, O(1) average lookup, insertion-ordered since 3.7
- `set`: hash table (keys only), O(1) membership
- `tuple`: fixed-size immutable array, hashable
- `str`: immutable Unicode, efficient slicing creates new strings
- `bisect` for ordered inserts, `deque` for queues, `Counter` for counting
- `copy.deepcopy` for full independence; `copy.copy` for shallow

---

## Chapter 60: OOP in Python (Deep Dive)

### Introduction

Python's object model is uniform: everything is an object, including classes, functions, and modules. This chapter covers not just how to write classes, but how Python's object system works under the hood — attribute lookup, descriptors, metaclasses, MRO, and slots.

### Why It Matters

Production Python systems rely on OOP for code organization, framework design (Django, SQLAlchemy), and API boundaries. Understanding the internals lets you debug attribute errors, optimize memory, and design APIs that feel native to Python.

### Real World Analogy

Python's object system is like a bureaucratic government. Every object has a type (its birth certificate). Every attribute access goes through a chain of lookups (committees). Descriptors are like department-specific procedures. Metaclasses are the legislature that defines how the government itself operates.

### Class Definition

```python
class Employee:
    company = "ACME Corp"
    employee_count = 0
    
    def __init__(self, name: str, salary: float):
        self.name = name
        self.salary = salary
        Employee.employee_count += 1
    
    @classmethod
    def from_string(cls, data: str):
        """Alternative constructor"""
        name, salary = data.split(",")
        return cls(name.strip(), float(salary))
    
    @staticmethod
    def is_workday(day):
        return day.weekday() < 5
    
    @property
    def bonus(self):
        return self.salary * 0.1
    
    @bonus.setter
    def bonus(self, value):
        self.salary = value * 10
```

### `__init__`, `__new__`, `__del__`

```python
class Singleton:
    _instance = None
    
    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self, value):
        self.value = value
    
    def __del__(self):
        # NOT guaranteed to be called
        print(f"Deleting {self}")
```

`__new__` is rarely overridden except for: Singleton pattern, immutable type subclasses, and metaclass operations.

### Method Types

| Method Type | First Argument | Access | Common Use |
|-------------|---------------|--------|------------|
| Instance method | `self` (instance) | Instance & class | Normal behavior |
| `@classmethod` | `cls` (class) | Class only | Alternative constructors |
| `@staticmethod` | None | Neither | Utility functions related to class |

### Property Decorator

```python
class Temperature:
    def __init__(self, celsius=0):
        self._celsius = celsius
    
    @property
    def celsius(self):
        return self._celsius
    
    @celsius.setter
    def celsius(self, value):
        if value < -273.15:
            raise ValueError("Below absolute zero")
        self._celsius = value
    
    @celsius.deleter
    def celsius(self):
        self._celsius = 0
    
    @property
    def fahrenheit(self):
        return self._celsius * 9/5 + 32
```

### Descriptors

Descriptors power properties, methods, `classmethod`, `staticmethod`, and `__slots__`.

```python
class PositiveNumber:
    def __set_name__(self, owner, name):
        self._name = name
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return obj.__dict__[self._name]
    
    def __set__(self, obj, value):
        if value <= 0:
            raise ValueError(f"{self._name} must be positive")
        obj.__dict__[self._name] = value

class Order:
    quantity = PositiveNumber()
    price = PositiveNumber()
    
    def __init__(self, quantity, price):
        self.quantity = quantity
        self.price = price
    
    @property
    def total(self):
        return self.quantity * self.price
```

### Encapsulation

```python
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self._protected = "convention only"
        self.__private = "name mangled"  # becomes _BankAccount__private
```

Name mangling: `__private` becomes `_ClassName__private` to prevent accidental overriding in subclasses. It is accessible via `obj._BankAccount__private`.

### Inheritance and MRO

```python
class A:
    def method(self):
        return "A"

class B(A):
    def method(self):
        return "B"

class C(A):
    def method(self):
        return "C"

class D(B, C):
    pass

d = D()
d.method()  # "B" (follows MRO)
D.__mro__   # (D, B, C, A, object)
```

> 📖 **Instructor Note:** C3 linearization guarantees a monotonic resolution order — each class appears before its parents, left-to-right order is respected, and no class appears more than once. This prevents the "diamond problem" where a method could be inherited through multiple conflicting paths.

MRO (Method Resolution Order) uses **C3 linearization**: children before parents, left-to-right order preserved, each class appears only once.

```python
# super() deep dive — follows MRO, not just parent
class Base:
    def __init__(self, name):
        self.name = name

class Mixin:
    def __init__(self, **kwargs):
        super().__init__(**kwargs)

class Concrete(Mixin, Base):
    def __init__(self, name, **kwargs):
        super().__init__(name=name, **kwargs)
```

### ABC (Abstract Base Classes)

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self) -> float:
        ...
    
    @abstractmethod
    def perimeter(self) -> float:
        ...
    
    def describe(self):
        return f"Area: {self.area()}, Perimeter: {self.perimeter()}"

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
    
    def area(self):
        return 3.14159 * self.radius ** 2
    
    def perimeter(self):
        return 2 * 3.14159 * self.radius
```

### Protocols (Structural Subtyping, PEP 544)

```python
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> str:
        ...

class Circle:
    def draw(self) -> str:
        return "Drawing circle"

class Square:
    def draw(self) -> str:
        return "Drawing square"

def render(obj: Drawable):
    print(obj.draw())

render(Circle())  # OK — structural subtyping
render(Square())  # OK
```

### Slots

```python
class MemoryEfficient:
    __slots__ = ('x', 'y')
    
    def __init__(self, x, y):
        self.x = x
        self.y = y

# Without __slots__
class Normal:
    def __init__(self, x, y):
        self.x = x
        self.y = y

import sys
n = Normal(1, 2)
m = MemoryEfficient(1, 2)
sys.getsizeof(n)  # ~56 bytes + dict
sys.getsizeof(m)  # ~56 bytes (no __dict__)
```

> 💡 **Pro Tip:** `__slots__` is a memory optimization, not a design pattern. Use it when creating millions of instances (e.g., data science records, game entities). The memory savings come at the cost of flexibility — no dynamic attributes, no `__dict__`, and potential complications with multiple inheritance and weak references.

`__slots__` eliminates the per-instance `__dict__`, saving ~40+ bytes per instance and providing faster attribute access, but prevents adding new attributes dynamically.

### Dunder Methods

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __repr__(self):          # repr() — unambiguous
        return f"Vector({self.x}, {self.y})"
    
    def __str__(self):           # str(), print() — readable
        return f"({self.x}, {self.y})"
    
    def __eq__(self, other):     # ==
        if not isinstance(other, Vector):
            return NotImplemented
        return self.x == other.x and self.y == other.y
    
    def __hash__(self):          # hash() — for dict key/set
        return hash((self.x, self.y))
    
    def __lt__(self, other):     # < (enables sorting)
        return (self.x, self.y) < (other.x, other.y)
    
    def __len__(self):           # len()
        return 2
    
    def __getitem__(self, idx):  # v[0]
        return (self.x, self.y)[idx]
    
    def __iter__(self):          # for x in v:
        return iter((self.x, self.y))
    
    def __enter__(self):         # with Vector() as v:
        return self
    
    def __exit__(self, *args):   # cleanup
        pass
    
    def __call__(self, scale):   # v(2.0)
        return Vector(self.x * scale, self.y * scale)
    
    def __bool__(self):          # bool(v)
        return self.x != 0 or self.y != 0
    
    def __add__(self, other):    # v1 + v2
        return Vector(self.x + other.x, self.y + other.y)
```

### Metaclasses

A metaclass is the class of a class. `type` is the default metaclass.

```python
# Classes are objects created by metaclasses
MyClass = type('MyClass', (object,), {'x': 10})

# Custom metaclass
class SingletonMeta(type):
    _instances = {}
    
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class DatabaseConnection(metaclass=SingletonMeta):
    def __init__(self, url):
        self.url = url
```

Metaclasses are a "solve the last 1% of problems" tool. Use dataclasses, descriptors, or decorators before reaching for metaclasses.

### Dataclasses

```python
from dataclasses import dataclass, field
from typing import List

@dataclass(frozen=True)
class Point:
    x: float
    y: float
    label: str = field(default="", compare=False)
    
    def __post_init__(self):
        if self.x < 0 or self.y < 0:
            raise ValueError("Coordinates must be non-negative")

@dataclass
class Employee:
    name: str
    employee_id: str
    salary: float = field(repr=False)
    skills: List[str] = field(default_factory=list)
```

### Common Mistakes

| Mistake | Issue | Fix |
|---------|-------|-----|
| Forgetting `super().__init__()` in child | Parent not initialized | Call `super().__init__()` |
| Mutable class variables | `class A: items = []` shared | Instance attribute in `__init__` |
| `__slots__` missing parent slots | AttributeError in subclasses | Include `__slots__` in child too |
| Overriding `__eq__` without `__hash__` | Object becomes unhashable | Define both or set `__hash__ = None` |
| Using `__private` names unnecessarily | Name mangling confusion | Use `_protected` convention |
| Returning from `__exit__` non-None | Suppresses exceptions | Return None (or False) to propagate |

### Best Practices

- Favor composition over inheritance
- Use `@dataclass` for data containers (reduces boilerplate 80%)
- Use `@property` for computed attributes (not getter/setter methods)
- Use ABCs for formal interfaces, Protocols for duck typing
- Use `__slots__` only when creating millions of instances
- Prefer `super()` over calling parent class directly
- Keep inheritance depth <= 3

### Interview Questions

**Q: Explain MRO and C3 linearization.**
A: C3 linearization creates a consistent, predictable method resolution order that respects: children before parents, left-to-right, and each class appears only once. `ClassName.__mro__` shows the resolution order.

**Q: What is the difference between `@classmethod` and `@staticmethod`?**
A: `@classmethod` receives the class as first argument (`cls`), enabling class-specific behavior and alternative constructors. `@staticmethod` receives no implicit first argument.

**Q: How do descriptors work?**
A: When a class attribute has `__get__`, `__set__`, or `__delete__` methods, accessing that attribute on an instance triggers the descriptor protocol.

### Practical Exercises

1. Implement a `User` class with validation (email format, age >= 0, password hashing)
2. Create a descriptor that logs every attribute access
3. Implement multiple inheritance diamond with cooperative `super()`
4. Convert a complex class to use `@dataclass` and compare code size

### Mini Project: Simple ORM

Build a minimal ORM that maps classes to database tables using metaclasses and descriptors for field validation.

### Revision Notes

- `__init__` = initializer, `__new__` = allocator, `__del__` = finalizer (unreliable)
- MRO: C3 linearization — `super()` follows MRO, not just parent
- Descriptors: `__get__`, `__set__`, `__delete__` — power properties and methods
- `__slots__`: eliminates `__dict__`, saves memory, speeds attribute access
- Metaclasses: class of a class — rarely needed for production code
- ABC: formal interfaces; Protocols: structural (duck typing at type-check level)
- Dataclasses: auto-generate `__init__`, `__repr__`, `__eq__`, `__hash__`

---

## Chapter 61: Generators, Iterators, Context Managers

### Introduction

Iterators and generators are Python's mechanism for lazy evaluation — producing values on demand rather than materializing entire collections. Context managers handle resource lifecycle deterministically.

### Why It Matters

These patterns enable processing data streams that don't fit in memory (terabyte-sized files), building efficient data pipelines, and ensuring resources are always released (even on exceptions). They are fundamental to Python's async model.

### Real World Analogy

Iterators are like a streaming service — you don't download the entire movie at once. Generators are like an assembly line — each station produces one part before passing it along. Context managers are like hotel checkout — no matter how you leave, the room gets cleaned up.

### Iterator Protocol

```python
class Countdown:
    def __init__(self, start):
        self.current = start
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        value = self.current
        self.current -= 1
        return value

# Usage
for n in Countdown(5):
    print(n)  # 5, 4, 3, 2, 1

# Manual iteration
cd = Countdown(3)
it = iter(cd)  # calls __iter__
next(it)  # 3
next(it)  # 2
next(it)  # 1
next(it)  # StopIteration
```

Any object with `__iter__` returning an iterator (object with `__next__`) is **iterable**. Any object with `__next__` is an **iterator**.

### Generator Functions

Generators are the easiest way to create iterators:

```python
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

# Usage
fib = fibonacci()
[next(fib) for _ in range(10)]  # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

When `yield` is encountered:
1. Current frame state is saved (locals, instruction pointer)
2. The yielded value is returned to the caller
3. On next `next()` call, execution resumes after the yield

> ✅ **Best Practice:** Generators are ideal for processing large datasets that don't fit in memory. A generator processing a 10GB file uses constant memory, while loading it into a list would crash. Always use generators for streaming, pipelines, and infinite sequences.

### Generator Expressions

```python
# Generator expression (lazy) vs list comprehension (eager)
squares_gen = (x**2 for x in range(10_000_000))  # zero memory
squares_list = [x**2 for x in range(10_000_000)]  # ~300MB

# Generator expressions can be consumed once
squares = (x**2 for x in range(5))
list(squares)  # [0, 1, 4, 9, 16]
list(squares)  # [] — exhausted
```

### send(), throw(), close()

```python
def coroutine():
    print("Started")
    try:
        while True:
            value = yield
            print(f"Received: {value}")
    except GeneratorExit:
        print("Generator closed")
    except ValueError:
        print("Caught ValueError, continuing")

coro = coroutine()
next(coro)       # Prime the generator
coro.send(10)    # "Received: 10"
coro.throw(ValueError)  # "Caught ValueError, continuing"
coro.close()     # "Generator closed"

# Two-way communication
def accumulate():
    total = 0
    while True:
        value = yield total
        total += value

acc = accumulate()
next(acc)      # prime → 0
acc.send(10)   # 10
acc.send(20)   # 30
```

### yield from

`yield from` delegates iteration to a sub-generator:

```python
def flatten(nested):
    for sublist in nested:
        yield from sublist

list(flatten([[1, 2], [3, 4], [5]]))  # [1, 2, 3, 4, 5]

# Recursive yield from
def tree_walk(node):
    yield node.value
    for child in node.children:
        yield from tree_walk(child)
```

`yield from` also propagates `send()`, `throw()`, and `close()` to the sub-generator.

### Context Managers

```python
class ManagedFile:
    def __init__(self, filename, mode='r'):
        self.filename = filename
        self.mode = mode
    
    def __enter__(self):
        self.file = open(self.filename, self.mode)
        return self.file
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type:
            print(f"Error occurred: {exc_val}")
            return False  # propagate exception
        self.file.close()
        return True

with ManagedFile('test.txt', 'w') as f:
    f.write('Hello, World!')
```

### contextlib Module

```python
from contextlib import contextmanager, suppress, redirect_stdout, closing
import io

# @contextmanager — generator-based context manager
@contextmanager
def temporary_file(name):
    print(f"Opening {name}")
    f = open(name, 'w')
    try:
        yield f
    finally:
        print(f"Closing {name}")
        f.close()

with temporary_file('test.txt') as f:
    f.write('data')

# suppress — ignore specific exceptions
with suppress(FileNotFoundError):
    os.remove('nonexistent.txt')

# redirect_stdout — capture print output
buf = io.StringIO()
with redirect_stdout(buf):
    print("Hello")
output = buf.getvalue()  # "Hello\n"

# Multiple context managers
with open('a.txt') as a, open('b.txt') as b:
    a_data = a.read()
    b_data = b.read()
```

### Common Mistakes

| Mistake | Issue | Fix |
|---------|-------|-----|
| Reusing generator | Generator exhausted after first iteration | Create new generator each time |
| Not priming coroutine | TypeError | Call `next(coro)` first |
| Forgetting `__iter__` vs `__next__` | Object not iterable | Define both |
| Suppressing exceptions in `__exit__` | Bugs hidden | Return False unless intentional |
| Infinite generator without break | Memory leak if partially consumed | Use `itertools.islice` |

### Best Practices

- Use generators for large data streams (files, network, DB queries)
- Use `yield from` for delegating to sub-generators
- Use `@contextmanager` over class-based for simple cases
- Use `itertools.islice` for bounded consumption of infinite generators
- Prefer generator expressions over list comprehensions when iterating once

### Interview Questions

**Q: How do generators save memory?**
A: Generators produce values on-demand without storing the entire sequence. Each `yield` pauses execution, and the frame state is saved on the heap rather than the C stack.

**Q: What is `yield from` and why use it?**
A: `yield from` delegates iteration to a sub-generator, passing through values, exceptions, and `send()` calls for clean composition.

**Q: How does the `with` statement work?**
A: `with obj as x:` calls `obj.__enter__()` and assigns the result to `x`. When the block exits, `obj.__exit__()` is called. If `__exit__` returns True, any exception is suppressed.

### Practical Exercises

1. Build a file reader that yields lines from multiple files in sequence
2. Implement `range()` using a generator
3. Write a context manager that measures execution time
4. Create a pipeline: CSV reader → filter → transform → JSON writer using generators

### Mini Project: Lazy CSV Processor

Build a streaming CSV processor that reads, transforms, and writes CSV files without loading the entire file into memory.

### Revision Notes

- Iterator: `__iter__` returns self, `__next__` yields next item, raises `StopIteration`
- Generator: function with `yield` — produces values lazily
- `send()`: send value into generator (two-way communication)
- `yield from`: delegate to sub-generator
- `with` statement: calls `__enter__` / `__exit__` protocol
- `@contextmanager`: turn a generator into a context manager

---

## Chapter 62: Decorators and Metaprogramming

### Introduction

Decorators are Python's primary mechanism for metaprogramming — code that modifies or enhances other code. They allow you to wrap functions and classes with additional behavior without changing their internal implementation.

### Why It Matters

Decorators are ubiquitous in production Python: Flask routes, Django's `@login_required`, pytest fixtures, caching (`@lru_cache`), rate limiting, logging, timing, and retry logic.

### Real World Analogy

Decorators are like the stamp at a passport control. The passport (original function) remains unchanged, but it goes through extra processing each time it passes through. The process is reusable across different passports.

### Decorator Pattern from Scratch

```python
# Basic decorator
def uppercase(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        return result.upper()
    return wrapper

@uppercase
def greet(name):
    return f"Hello, {name}"

greet("Alice")  # "HELLO, ALICE"
```

### functools.wraps — Why It Matters

```python
from functools import wraps

def uppercase(func):
    @wraps(func)  # preserves __name__, __doc__, __module__, __annotations__
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        return result.upper()
    return wrapper

@uppercase
def greet(name):
    """Greet someone politely"""
    return f"Hello, {name}"

greet.__name__  # 'greet' (without @wraps would be 'wrapper')
greet.__doc__   # 'Greet someone politely' (without @wraps would be None)
```

### Decorators with Arguments

```python
from functools import wraps

def repeat(times=1):
    """Decorator with arguments — 3 levels deep"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(times=3)
def say(msg):
    print(msg)

say("Hello")  # prints "Hello" 3 times
```

### Class-Based Decorators

```python
from functools import wraps

class CountCalls:
    def __init__(self, func):
        wraps(func)(self)
        self.func = func
        self.count = 0
    
    def __call__(self, *args, **kwargs):
        self.count += 1
        print(f"Call {self.count} of {self.func.__name__}")
        return self.func(*args, **kwargs)

@CountCalls
def say_hello(name):
    print(f"Hello {name}")

say_hello("Alice")  # "Call 1 of say_hello" / "Hello Alice"
say_hello("Bob")    # "Call 2 of say_hello" / "Hello Bob"
```

### Production Decorators

```python
from functools import wraps
import time
import logging

logger = logging.getLogger(__name__)

# Timing decorator
def timed(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        logger.info(f"{func.__name__} took {elapsed:.3f}s")
        return result
    return wrapper

# Retry decorator
def retry(max_attempts=3, delay=1.0, backoff=2.0):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            last_exception = None
            wait = delay
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    last_exception = e
                    logger.warning(f"Attempt {attempt} failed: {e}")
                    if attempt < max_attempts:
                        time.sleep(wait)
                        wait *= backoff
            raise last_exception
        return wrapper
    return decorator

> 💡 **Pro Tip:** The `@lru_cache` decorator can dramatically speed up recursive or expensive functions through memoization. Use `maxsize=None` for deterministic functions with bounded inputs (like Fibonacci), but set a reasonable `maxsize` for caching varying inputs to avoid memory leaks.

# LRU Cache (built-in)
from functools import lru_cache

@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

fibonacci(100)
fibonacci.cache_info()  # CacheInfo(hits=98, misses=101, maxsize=128, currsize=101)
```

### Singledispatch

```python
from functools import singledispatch

@singledispatch
def serialize(obj):
    return str(obj)

@serialize.register(int)
def _(obj):
    return f"int:{obj}"

@serialize.register(list)
def _(obj):
    return f"list:[{','.join(str(x) for x in obj)}]"

@serialize.register(dict)
def _(obj):
    items = ','.join(f"{k}={v}" for k, v in obj.items())
    return f"dict:{{{items}}}"

serialize(42)        # "int:42"
serialize([1, 2, 3]) # "list:[1,2,3]"
```

### functools Utilities

```python
from functools import partial, reduce, total_ordering, cached_property

# partial — fix function arguments
def power(base, exponent):
    return base ** exponent

square = partial(power, exponent=2)
cube = partial(power, exponent=3)
square(5)  # 25
cube(5)    # 125

# reduce — accumulate across sequence
sum_all = reduce(lambda a, b: a + b, [1, 2, 3, 4])  # 10

# total_ordering — derive all comparisons from __eq__ and __lt__
@total_ordering
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    def __eq__(self, other):
        return self.age == other.age
    def __lt__(self, other):
        return self.age < other.age

# cached_property
class Expensive:
    @cached_property
    def data(self):
        print("Computing... (once)")
        return heavy_computation()

e = Expensive()
e.data  # computed once, cached thereafter
```

### inspect Module

```python
import inspect

def validate(func):
    sig = inspect.signature(func)
    
    def wrapper(*args, **kwargs):
        bound = sig.bind(*args, **kwargs)
        for name, value in bound.arguments.items():
            param = sig.parameters[name]
            if param.annotation and not isinstance(value, param.annotation):
                raise TypeError(f"{name} should be {param.annotation}")
        return func(*args, **kwargs)
    return wrapper

@validate
def process(name: str, count: int):
    print(name, count)

process("test", 5)   # OK
process(123, "abc")  # TypeError
```

### Common Mistakes

| Mistake | Issue | Fix |
|---------|-------|-----|
| Forgetting `@wraps` | Lost function metadata | Always use `@wraps(func)` |
| Decorator without `()` | Function called at definition time | Return wrapper function |
| Mutable cross-call state | State shared across calls | Create state inside wrapper |
| Stacking decorators in wrong order | Unexpected execution order | Innermost wraps closest to function |

### Best Practices

- Always use `@wraps` to preserve metadata
- Keep decorator logic focused (one concern per decorator)
- Use `> 💡 **Pro Tip:** The `@lru_cache` decorator can dramatically speed up recursive or expensive functions through memoization. Use `maxsize=None` for deterministic functions with bounded inputs (like Fibonacci), but set a reasonable `maxsize` for caching varying inputs to avoid memory leaks.

functools.lru_cache` for memoization
- Use `functools.singledispatch` for type-dispatch over isinstance chains
- Stack decorators in logical order (inner-to-outer execution)
- Prefer class-based decorators for stateful behavior

### Interview Questions

**Q: What is the difference between a decorator and a decorator factory?**
A: A decorator takes a function and returns a function (`@deco`). A decorator factory takes arguments and returns a decorator (`@deco(args)`), requiring 3 levels of nesting.

**Q: How does `@lru_cache` work?**
A: It wraps the function with an LRU dictionary mapping arguments to results. On call, it checks the cache first, using LRU eviction when maxsize is exceeded.

**Q: What problems does `@wraps` solve?**
A: Without it, the decorated function's `__name__`, `__doc__`, `__module__`, and `__annotations__` become those of the wrapper, breaking introspection and debugging.

### Practical Exercises

1. Write a `@delay(seconds)` decorator that delays function execution
2. Write a `@memoize` decorator (handling hashable arguments)
3. Write a `@validate_types` decorator that checks type annotations at runtime
4. Write a `@log_calls` decorator that logs function entry/exit

### Mini Project: Plugin Registry

Build a decorator-based plugin system where classes can be registered using `@register("name")` and discovered at runtime.

### Revision Notes

- `@decorator`: syntactic sugar for `func = decorator(func)`
- `@wraps(func)`: preserves function metadata
- Decorators with arguments require 3-level nesting
- `@lru_cache`: built-in memoization with LRU eviction
- `@singledispatch`: function overloading by type
- `functools.partial`: fix function arguments
- `inspect` module: source, signatures, stack frames for metaprogramming

---

## Chapter 63: Functional Programming in Python

### Introduction

Python is multi-paradigm, supporting imperative, object-oriented, and functional styles. While not purely functional, Python provides map/filter/reduce, lambdas, itertools, functools, and operator modules.

### Why It Matters

Functional programming patterns lead to more predictable code (no side effects), more testable code (pure functions), and more composable designs. Python's `itertools` and `functools` are essential for data processing pipelines.

### Real World Analogy

Functional programming is like a factory assembly line. Each station takes an input, transforms it, and passes it to the next station without changing the original material.

### map, filter, reduce

```python
from functools import reduce

# map — transform each element
nums = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x**2, nums))  # [1, 4, 9, 16, 25]

# filter — keep matching elements
evens = list(filter(lambda x: x % 2 == 0, nums))  # [2, 4]

# reduce — accumulate
total = reduce(lambda a, b: a + b, nums)  # 15
total = sum(nums)  # Better: use built-in sum

# Multi-argument map
from operator import add, mul
list(map(add, [1, 2, 3], [4, 5, 6]))  # [5, 7, 9]
```

### lambda

```python
# Single expression only
square = lambda x: x**2

# Multi-line workaround — use a named function instead
def complex_func(x):
    y = x + 1
    z = y ** 2
    return z - 1
```

### itertools

```python
import itertools

# chain — concatenate iterables
list(itertools.chain([1, 2], [3, 4], [5, 6]))  # [1, 2, 3, 4, 5, 6]

# cycle — infinite repeat
counter = itertools.cycle(['A', 'B', 'C'])
[next(counter) for _ in range(5)]  # ['A', 'B', 'C', 'A', 'B']

# accumulate — running totals
list(itertools.accumulate([1, 2, 3, 4]))  # [1, 3, 6, 10]

> 💡 **Pro Tip:** The `itertools` module is a hidden gem for memory-efficient data processing. `itertools.chain` avoids creating intermediate lists, `itertools.product` replaces nested loops, and `itertools.groupby` handles sorted-group aggregation. Master these for cleaner, faster code.

# product — Cartesian product
list(itertools.product('AB', '12'))  # [('A','1'), ('A','2'), ('B','1'), ('B','2')]

# permutations — all orderings
list(itertools.permutations('ABC', 2))  # [('A','B'), ('A','C'), ('B','A'), ('B','C'), ('C','A'), ('C','B')]

# combinations — unordered selections
list(itertools.combinations('ABC', 2))  # [('A','B'), ('A','C'), ('B','C')]

# groupby — group consecutive elements (MUST be sorted by key)
data = [('A', 1), ('A', 2), ('B', 3), ('B', 4)]
for key, group in itertools.groupby(data, key=lambda x: x[0]):
    print(key, list(group))

# islice — slicing on iterators
iterator = iter(range(1000))
list(itertools.islice(iterator, 5))  # [0, 1, 2, 3, 4]

# takewhile / dropwhile
list(itertools.takewhile(lambda x: x < 5, [1, 3, 7, 2, 9]))  # [1, 3]
list(itertools.dropwhile(lambda x: x < 5, [1, 3, 7, 2, 9]))  # [7, 2, 9]
```

### operator Module

```python
from operator import itemgetter, attrgetter, methodcaller

# itemgetter — extract by index/key
data = [('Alice', 30), ('Bob', 25), ('Charlie', 35)]
sorted(data, key=itemgetter(1))  # sort by age

# attrgetter — extract by attribute
Person = namedtuple('Person', ['name', 'age'])
people = [Person('Alice', 30), Person('Bob', 25)]
sorted(people, key=attrgetter('age'))

# methodcaller — call a method
strings = ['hello', 'world']
list(map(methodcaller('upper'), strings))  # ['HELLO', 'WORLD']
```

### functools.partial

```python
from functools import partial

def power(base, exponent):
    return base ** exponent

square = partial(power, exponent=2)
cube = partial(power, exponent=3)
square(5)  # 25
cube(5)    # 125

# Partial with mapping
def format_log(level, message, timestamp=None):
    return f"[{level}] {timestamp or 'now'}: {message}"

info = partial(format_log, 'INFO')
error = partial(format_log, 'ERROR')
info("System started")      # [INFO] now: System started
```

### Function Composition Patterns

```python
from functools import reduce

def compose(*functions):
    """Compose functions right-to-left: compose(f, g)(x) = f(g(x))"""
    def composed(x):
        for func in reversed(functions):
            x = func(x)
        return x
    return composed

def pipe(*functions):
    """Pipe functions left-to-right: pipe(f, g)(x) = g(f(x))"""
    def piped(x):
        for func in functions:
            x = func(x)
        return x
    return piped

add_one = lambda x: x + 1
double = lambda x: x * 2

composed = compose(add_one, double)  # (5*2) + 1 = 11
piped = pipe(add_one, double)        # (5+1) * 2 = 12
```

### Common Mistakes

| Mistake | Issue | Fix |
|---------|-------|-----|
| Using map/filter with lambda over comprehensions | Less readable | Prefer comprehensions |
| Forgetting list() on map/filter | Returns iterator | Wrap in list() |
| reduce for simple operations | Less clear than built-in | Use sum, any, all |
| Lambda capturing loop variable | All closures reference same variable | `lambda x=i: x` |
| groupby without sorting | Wrong groups | Sort by key first |

### Best Practices

- Prefer comprehensions over `map`/`filter` with lambdas
- Use `map` with existing functions (not lambdas): `map(str.upper, items)`
- Use `itertools` for combinations, permutations, grouping
- Use `functools.partial` to create specialized functions
- Use `operator` module functions over equivalent lambdas
- Prefer `sum()`, `any()`, `all()`, `max()`, `min()` over `reduce`

### Interview Questions

**Q: When would you use `map` over a list comprehension?**
A: When applying an existing function to multiple iterables: `map(add, list1, list2)`. Also for lazy iteration without materializing a list.

**Q: How does `itertools.groupby` work?**
A: It groups consecutive elements with the same key. The iterable must be sorted by the key function first, as it only groups adjacent elements.

**Q: What is the difference between `partial` and lambda?**
A: `partial` freezes specific arguments of an existing function, preserving metadata. Lambda creates a new anonymous function.

### Practical Exercises

1. Implement `itertools.compress` from scratch
2. Use `itertools.product` to generate all possible 4-digit PIN combinations
3. Write a pipeline that reads a file, filters lines, transforms, and aggregates — all lazily
4. Implement a `group_by` function using `itertools.groupby`

### Mini Project: Functional Data Pipeline

Build a composable ETL pipeline using `pipe`, `map`, `filter`, and `itertools` operations.

### Revision Notes

- `map`, `filter` return lazy iterators; wrap in `list()` to materialize
- `itertools`: chain, product, permutations, combinations, groupby, islice, accumulate
- `operator`: itemgetter, attrgetter, methodcaller — faster than lambdas
- `functools.partial`: freeze arguments; compose/pipe for function composition
- Lambdas are single-expression only; use named functions for complexity

---

## Chapter 64: Error Handling and Logging

### Introduction

Robust error handling separates production-grade code from simple scripts. Python's exception system is powerful and nuanced, and its logging module provides enterprise-grade observability.

### Why It Matters

In production, errors are inevitable. Well-structured error handling and logging is your first line of defense for diagnosing and recovering from failures.

### Real World Analogy

Exception handling is like airbags in a car — you hope you never need them, but they must work perfectly when you do. Logging is the black box recorder — it captures what happened before, during, and after the crash.

### Exception Hierarchy

```
BaseException
├── SystemExit
├── KeyboardInterrupt
├── GeneratorExit
└── Exception
    ├── StopIteration
    ├── ArithmeticError (OverflowError, ZeroDivisionError)
    ├── LookupError (IndexError, KeyError)
    ├── TypeError
    ├── ValueError
    ├── OSError (FileNotFoundError, PermissionError)
    ├── RuntimeError
    ├── AttributeError
    ├── ImportError (ModuleNotFoundError)
    └── ...
```

Catch `Exception`, not `BaseException` (catching `BaseException` would suppress `SystemExit` and `KeyboardInterrupt`).

### try/except/else/finally

```python
try:
    result = risky_operation()
except ValueError as e:
    print(f"Value error: {e}")
except (TypeError, RuntimeError) as e:
    print(f"Type or runtime error: {e}")
except Exception as e:
    print(f"Unexpected error: {e}")
    raise
else:
    # Runs ONLY if no exception occurred
    print(f"Success: {result}")
finally:
    # ALWAYS runs — even on return, break, continue, or exception
    cleanup()
```

| Scenario | try | except | else | finally |
|----------|-----|--------|------|---------|
| No exception | Run | Skip | Run | Run |
| Exception caught | Run (until error) | Run | Skip | Run |
| Exception not caught | Run (until error) | Skip | Skip | Run (then propagate) |
| return in try | Run | — | — | Run (before return) |

### Custom Exceptions

```python
class APIError(Exception):
    def __init__(self, message, status_code=None, response=None):
        super().__init__(message)
        self.status_code = status_code
        self.response = response

class NotFoundError(APIError):
    pass

class RateLimitError(APIError):
    def __init__(self, message, retry_after=None):
        super().__init__(message, status_code=429)
        self.retry_after = retry_after

class ValidationError(APIError):
    pass
```

### Exception Chaining

```python
# Explicit chaining
try:
    result = parse_user_input(data)
except ValueError as e:
    raise APIError(f"Invalid input: {data}") from e

# Suppress context
try:
    result = connect()
except ConnectionError:
    raise APIError("Service unavailable") from None

# Implicit chaining (during handling of another exception)
try:
    result = connect()
except ConnectionError:
    raise APIError("Service unavailable")
```

### assert and -O Flag

```python
def divide(a, b):
    assert b != 0, "Division by zero"
    return a / b

# Assertions are removed with -O flag
# $ python -O script.py
```

Use assertions for debugging and internal invariants, NOT for data validation.

### warnings Module

```python
import warnings

def old_function():
    warnings.warn("old_function is deprecated, use new_function",
                  DeprecationWarning, stacklevel=2)

warnings.filterwarnings("ignore", category=DeprecationWarning)
```

### logging Module

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S'
)

logger = logging.getLogger(__name__)

logger.debug("Detailed debugging info")    # 10
logger.info("Normal operation")            # 20
logger.warning("Something unusual")        # 30
logger.error("Function failed")            # 40
logger.critical("System is down")          # 50
```

#### Logging Configuration (dictConfig)

```python
import logging.config

LOGGING_CONFIG = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'standard': {
            'format': '%(asctime)s [%(levelname)s] %(name)s: %(message)s'
        },
        'json': {
            'format': '{"time":"%(asctime)s","level":"%(levelname)s","logger":"%(name)s","message":"%(message)s"}'
        },
    },
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'level': 'INFO',
            'formatter': 'standard',
        },
        'file': {
            'class': 'logging.handlers.RotatingFileHandler',
            'level': 'DEBUG',
            'formatter': 'json',
            'filename': 'app.log',
            'maxBytes': 10 * 1024 * 1024,
            'backupCount': 5,
        },
    },
    'loggers': {
        '': {
            'handlers': ['console', 'file'],
            'level': 'DEBUG',
        },
        'third_party_lib': {
            'handlers': ['console'],
            'level': 'WARNING',
            'propagate': False,
        },
    },
}

logging.config.dictConfig(LOGGING_CONFIG)
```

### traceback Module

```python
import traceback
import sys

try:
    risky_call()
except Exception:
    traceback.print_exc()
    tb_str = traceback.format_exc()
    logger.error(tb_str)
    
    exc_type, exc_value, exc_tb = sys.exc_info()
    frames = traceback.extract_tb(exc_tb)
    for filename, lineno, funcname, text in frames:
        print(f"{filename}:{lineno} in {funcname}(): {text}")
```

### contextlib.suppress

```python
from contextlib import suppress

with suppress(FileNotFoundError):
    os.remove('temp_file.txt')
```

> ✅ **Best Practice:** Structured logging (JSON format) is essential for production systems. It enables log aggregation tools (ELK, Loki, CloudWatch) to parse and query logs automatically. Include request IDs, user IDs, and timestamps in every log entry for debugging distributed systems.

### Logging Best Practices

| Practice | Why |
|----------|-----|
| Use `__name__` as logger name | Enables per-module configuration |
| Log at correct level | DEBUG for dev, INFO for normal, ERROR for failures |
| Include context in log messages | Add user ID, request ID, relevant data |
| Use structured logging (JSON) | Enables log aggregation (ELK, Splunk) |
| Log rotation | Prevent disk full from log growth |
| Don't log sensitive data | No passwords, tokens, PII |
| Use exception handlers at boundaries | Top-level catch for unhandled errors |

### Common Mistakes

| Mistake | Issue | Fix |
|---------|-------|-----|
| Catching bare `except:` | Suppresses KeyboardInterrupt | `except Exception:` |
| Empty except block | Silently swallows errors | At least log the exception |
| Not using `logging.exception` | Loses traceback | `logger.exception("msg")` |
| Catching and re-raising wrong | Changes exception type | `raise` (bare) to preserve traceback |
| `assert` for validation | Disabled with -O flag | Use `if not condition: raise ValueError` |
| Not rotating logs | Disk full crash | Use `RotatingFileHandler` |

### Best Practices

- Define custom exceptions for your application domain
- Catch specific exceptions, not generic `Exception`
- Use `logging.exception()` in except blocks
- Use `raise ... from` for explicit exception chaining
- Configure logging at application startup, not in each module
- Use `contextlib.suppress` for intentional exception ignoring

### Interview Questions

**Q: What is the difference between `__cause__` and `__context__`?**
A: `__cause__` is set by explicit `raise X from Y`. `__context__` is set implicitly if an exception occurs during exception handling.

**Q: Should exceptions be used for control flow?**
A: In Python, sometimes yes — `StopIteration` controls iterator protocol. But avoid for frequent cases; use `if` checks.

**Q: How do you ensure cleanup code runs even on exception?**
A: Use `finally` block or `with` statement (context manager). `finally` always executes — even on `return`, `break`, or unhandled exceptions.

### Practical Exercises

1. Write a robust retry wrapper with exponential backoff and logging
2. Implement a custom `DatabaseError` hierarchy with connection, query, and timeout errors
3. Configure logging for a multi-module project with file rotation and JSON output
4. Write a decorator that logs all function calls with arguments

### Mini Project: Error Tracking Middleware

Build middleware for a web framework that catches all exceptions, logs them with request context, and returns appropriate HTTP error responses.

### Revision Notes

- Catch `Exception`, not `BaseException`
- try/except/else/finally — else runs on success, finally always runs
- `raise ... from X` for chaining; `raise ... from None` to suppress
- Custom exceptions inherit from Exception (or more specific)
- `logging.basicConfig()` for simple setup; dictConfig for production
- `logging.exception()` includes traceback
- `contextlib.suppress` for clean exception suppression

---

## Chapter 65: File I/O and Serialization

### Introduction

Python provides multiple layers of file I/O, from low-level os.open() to high-level pathlib.Path and serialization formats including JSON, CSV, XML, YAML, and Pickle.

### Why It Matters

Virtually every production system reads and writes data. Understanding encoding handling, binary vs text mode, serialization security, and streaming patterns is essential for data integrity.

### Real World Analogy

Files are like shipping containers. The open() call is the loading dock. Encoding is the language on the labels. Binary mode is the container's physical contents; text mode is unpacked items. Serialization is packing/unpacking the entire container.

### open() Modes

```python
# Mode flags
# 'r' — read (default)   'w' — write (truncate)    'a' — append
# 'x' — exclusive create  'b' — binary               't' — text (default)
# '+' — read and write

with open('file.txt', 'r') as f:     # text read
    content = f.read()

with open('file.txt', 'w') as f:     # write (overwrites)
    f.write("Hello")

with open('file.txt', 'a') as f:     # append
    f.write("World")

with open('file.txt', 'x') as f:     # exclusive create
    f.write("New file")

with open('file.bin', 'rb') as f:    # binary read
    data = f.read()

with open('file.txt', 'r+') as f:    # read and write
    content = f.read()
    f.seek(0)
    f.write("Replaced")

# ALWAYS specify encoding for text mode
with open('file.txt', 'r', encoding='utf-8') as f:
    content = f.read()
```

### Text vs Binary Mode

| Feature | Text (t) | Binary (b) |
|---------|----------|-------------|
| Data type | str (Unicode) | bytes |
| Newline translation | \n to \r\n (platform) | No translation |
| .read() returns | str | bytes |
| Encoding required | Yes | No |

### pathlib Path

```python
from pathlib import Path

p = Path('/home/user/data')

# Path components
p.name          # 'data'
p.stem          # 'data'
p.suffix        # ''
p.parent        # '/home/user'

# Reading/Writing
Path('file.txt').write_text("Hello", encoding='utf-8')
Path('file.txt').read_text(encoding='utf-8')
Path('file.bin').write_bytes(b'\x00\x01\x02')
Path('file.bin').read_bytes()

# File operations
Path('new_dir').mkdir(parents=True, exist_ok=True)
Path('file').unlink(missing_ok=True)

# Directory walking
for path in Path('.').glob('*.py'):
    print(path)

for path in Path('.').rglob('**/*.py'):
    print(path)

# File info
p = Path('file.txt')
p.exists()
p.is_file() | p.is_dir()
p.stat().st_size

# Path joining
data_dir = Path('/home/user/data')
csv_files = data_dir / 'subdir' / 'data.csv'
```

### tempfile Module

```python
import tempfile

# Temporary file (auto-deleted on close)
with tempfile.NamedTemporaryFile(mode='w', suffix='.txt', delete=True) as f:
    f.write("Temporary data")
    temp_path = f.name

# Temporary directory
with tempfile.TemporaryDirectory() as tmpdir:
    path = Path(tmpdir) / 'file.txt'
    path.write_text("test")
```

### Pickle

```python
import pickle

data = {'users': ['Alice', 'Bob'], 'scores': [95.5, 87.3]}

with open('data.pkl', 'wb') as f:
    pickle.dump(data, f, protocol=pickle.HIGHEST_PROTOCOL)

with open('data.pkl', 'rb') as f:
    loaded = pickle.load(f)

# Custom serialization
class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def __getstate__(self):
        return {'name': self.name, 'age': self.age}
    
    def __setstate__(self, state):
        self.name = state['name']
        self.age = state['age']

> ⚠️ **Warning:** Pickle is dangerous because it can execute arbitrary code during deserialization. A crafted pickle payload can run `os.system()`, exfiltrate data, or install malware. **NEVER** unpickle data from untrusted sources. Use JSON, MessagePack, or Protocol Buffers for untrusted data.

# WARNING: NEVER unpickle untrusted data — arbitrary code execution
```

### JSON

```python
import json
from datetime import datetime

data = {'name': 'Alice', 'age': 30, 'active': True, 'score': None}

json_str = json.dumps(data, indent=2, ensure_ascii=False)

with open('data.json', 'w', encoding='utf-8') as f:
    json.dump(data, f, indent=2)

with open('data.json', 'r', encoding='utf-8') as f:
    loaded = json.load(f)

# Custom encoder
class DateTimeEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.isoformat()
        return super().default(obj)

json.dumps({'time': datetime.now()}, cls=DateTimeEncoder)

# Custom decoder
def decode_datetime(obj):
    for key, value in obj.items():
        if isinstance(value, str):
            try:
                obj[key] = datetime.fromisoformat(value)
            except (ValueError, TypeError):
                pass
    return obj

data = json.loads(json_str, object_hook=decode_datetime)
```

### CSV

```python
import csv

with open('users.csv', 'w', newline='', encoding='utf-8') as f:
    writer = csv.DictWriter(f, fieldnames=['name', 'age', 'role'])
    writer.writeheader()
    writer.writerow({'name': 'Alice', 'age': 30, 'role': 'Engineer'})

with open('users.csv', 'r', newline='', encoding='utf-8') as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row['name'], row['age'])
```

### XML

```python
import xml.etree.ElementTree as ET

root = ET.Element('users')
user = ET.SubElement(root, 'user', id='1')
name = ET.SubElement(user, 'name')
name.text = 'Alice'

tree = ET.ElementTree(root)
tree.write('users.xml', encoding='utf-8', xml_declaration=True)

tree = ET.parse('users.xml')
root = tree.getroot()
for user in root.findall('user'):
    print(user.get('id'), user.find('name').text)
```

### YAML

```python
# Requires: pip install pyyaml
import yaml

data = {'version': 1, 'users': [{'name': 'Alice', 'age': 30}]}

with open('config.yaml', 'w') as f:
    yaml.dump(data, f, default_flow_style=False)

with open('config.yaml', 'r') as f:
    loaded = yaml.safe_load(f)  # ALWAYS use safe_load, not load
```

### ConfigParser

```python
import configparser

config = configparser.ConfigParser()
config['DEFAULT'] = {'debug': 'False', 'log_level': 'INFO'}
config['database'] = {'host': 'localhost', 'port': '5432'}

with open('app.ini', 'w') as f:
    config.write(f)

config.read('app.ini')
db_host = config['database']['host']
db_port = config.getint('database', 'port')
debug = config.getboolean('DEFAULT', 'debug')
```

### shelve

```python
import shelve

with shelve.open('data.db') as db:
    db['users'] = ['Alice', 'Bob']
    
    # IMPORTANT: returns a COPY, not reference
    users = db['users']
    users.append('Charlie')
    db['users'] = users  # must reassign
```

> ⚠️ **Warning:** A common file I/O bug is forgetting to close file handles. Using the `with` statement guarantees cleanup even if an exception occurs. Without it, file descriptors leak, which can crash long-running servers when the OS limit (typically 256-1024 per process) is exhausted.

### Common Mistakes

| Mistake | Issue | Fix |
|---------|-------|-----|
| Not specifying encoding | Platform-dependent | Always specify encoding='utf-8' |
| Using open() without with | File descriptor leak | Use `with open(...)` |
| Pickling untrusted data | Arbitrary code execution | Use JSON for external data |
| CSV newline issues | Blank lines in output | Add `newline=''` to open |
| yaml.load() without safe | Arbitrary code execution | Always use yaml.safe_load() |
| Binary vs text confusion | TypeError or corrupt data | Match mode to data type |

### Best Practices

- Always specify `encoding='utf-8'` for text files
- Use `pathlib.Path` over `os.path` for modern code
- Use `with` statements for all file operations
- Prefer JSON over Pickle for cross-language compatibility
- Use `yaml.safe_load()` (never `yaml.load()`)
- For large files, iterate line-by-line instead of .read()

### Interview Questions

**Q: What is the difference between `r+`, `w+`, and `a+`?**
A: r+ — read and write (file must exist, no truncation). w+ — read and write (creates or truncates). a+ — read and append (creates if needed, writes at end).

**Q: Why is Pickle dangerous?**
A: Pickle can execute arbitrary Python code during deserialization. A crafted payload can call os.system() or execute shell commands.

**Q: How does pathlib compare to os.path?**
A: pathlib provides object-oriented path manipulation with operator overloading (/ for join), consistent read/write methods, and is more readable.

### Practical Exercises

1. Write a script that recursively finds all .py files using pathlib
2. Parse a CSV, filter rows, and write to JSON
3. Create a binary file format using struct and read it back
4. Implement a simple INI config parser

### Mini Project: Configuration Loader

Build a configuration loader that supports JSON, YAML, INI, and environment variables with cascade (defaults → config file → env → CLI args).

### Revision Notes

- open() modes: r/w/a/x/b/t/+ — always specify encoding for text
- pathlib.Path for modern file handling
- pickle for Python-only serialization (NEVER unpickle untrusted data)
- json for cross-language; custom encoder/decoder for non-standard types
- csv.DictReader/DictWriter for structured tabular data
- yaml.safe_load() over yaml.load()
- tempfile for temporary files with automatic cleanup

---

## Chapter 66: Concurrency and Parallelism

### Introduction

Python offers three concurrency models: threading (I/O concurrency), multiprocessing (CPU parallelism), and asyncio (cooperative multitasking). Each has distinct characteristics and use cases.

### Why It Matters

Modern applications must handle I/O-bound workloads (web requests, DB queries) and CPU-bound workloads (data processing, ML inference) efficiently. Choosing the right model can yield 10-100x throughput improvements.

### Real World Analogy

- **Threading**: A single cook with multiple orders. When one bakes (I/O wait), the cook starts another. Only one order worked at a time.
- **Multiprocessing**: Multiple cooks, each working independently.
- **Asyncio**: A single cook who pauses one task to start another, switching efficiently.

### Threading

```python
import threading
import time

def worker(name, delay):
    print(f"Worker {name} starting")
    time.sleep(delay)
    print(f"Worker {name} done")

threads = [
    threading.Thread(target=worker, args=("A", 2)),
    threading.Thread(target=worker, args=("B", 1), daemon=True),
]

for t in threads:
    t.start()

for t in threads:
    t.join(timeout=5)
```

#### Thread Safety

```python
import threading

lock = threading.Lock()
rlock = threading.RLock()        # Reentrant
semaphore = threading.Semaphore(5)
event = threading.Event()
barrier = threading.Barrier(3)

shared_counter = 0
def increment():
    global shared_counter
    for _ in range(100000):
        with lock:
            shared_counter += 1

# Condition
condition = threading.Condition(lock)
def consumer():
    with condition:
        condition.wait()  # releases lock, reacquires on notify
def producer():
    with condition:
        condition.notify_all()
```

### GIL (Global Interpreter Lock)

The GIL prevents multiple threads from executing Python bytecode simultaneously. It protects CPython's memory model.

The GIL is released during:
- I/O operations (file, network, socket)
- C extension calls that release GIL explicitly (NumPy, cryptography)
- time.sleep()
- Blocking operations (Lock.acquire(), Queue.get())

| Scenario | GIL Impact |
|----------|-----------|
| CPU-bound threads | No parallelism (same speed as 1 thread) |
| I/O-bound threads | Good parallelism (GIL released during I/O) |
| C extensions (NumPy) | No GIL issue (explicit release) |

> 📖 **Instructor Note:** The GIL is CPython's trade-off: it simplifies memory management (no fine-grained locking on every object) at the cost of CPU-bound parallelism. PEP 703 (nogil) aims to make the GIL optional in Python 3.13+, but most production code still uses standard CPython. For CPU-bound work today, use multiprocessing or NumPy/Cython which release the GIL internally.

Python 3.13+ introduces an experimental `--disable-gil` (nogil) build.

### Multiprocessing

```python
from multiprocessing import Process, Pool, Queue, Pipe, Value, Array, Manager
import os

# Pool — worker process pool
def square(x):
    return x * x

with Pool(processes=4) as pool:
    results = pool.map(square, range(100))
    async_result = pool.map_async(square, range(100))
    results = async_result.get(timeout=10)
    for result in pool.imap(square, range(100)):
        pass

# Queue, Pipe, shared memory
from multiprocessing import Value, Array
counter = Value('i', 0)
shared_array = Array('d', 10)

# Manager for shared Python objects
with Manager() as manager:
    shared_dict = manager.dict()
    shared_list = manager.list()
```

### concurrent.futures

```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor, as_completed
import time

def fetch_url(url):
    time.sleep(1)
    return f"Data from {url}"

urls = [f"http://example.com/{i}" for i in range(10)]

# Thread pool (I/O-bound)
with ThreadPoolExecutor(max_workers=5) as executor:
    futures = [executor.submit(fetch_url, url) for url in urls]
    for future in as_completed(futures):
        result = future.result()
    results = list(executor.map(fetch_url, urls))

# Process pool (CPU-bound)
def cpu_task(n):
    return sum(i * i for i in range(n))

with ProcessPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(cpu_task, [10**7] * 8))
```

### Asyncio

```python
import asyncio
import aiohttp

async def fetch_url(session, url):
    async with session.get(url) as response:
        return await response.text()

async def main():
    async with aiohttp.ClientSession() as session:
        urls = [
            "http://example.com",
            "http://httpbin.org/get",
            "http://python.org",
        ]
        
        # Gather — run concurrently
        results = await asyncio.gather(
            *[fetch_url(session, url) for url in urls]
        )
        
        # Create tasks
        tasks = [asyncio.create_task(fetch_url(session, url)) for url in urls]
        
        # Wait for first to complete
        done, pending = await asyncio.wait(
            tasks, timeout=5.0, return_when=asyncio.FIRST_COMPLETED
        )

asyncio.run(main())
```

#### Asyncio Synchronization

```python
import asyncio

async def main():
    lock = asyncio.Lock()
    semaphore = asyncio.Semaphore(10)
    queue = asyncio.Queue(maxsize=100)
    
    async with semaphore:
        async with lock:
            await asyncio.sleep(0.1)
    
    await queue.put("item")
    item = await queue.get()
    queue.task_done()

asyncio.run(main())
```

### Decision Table

| Factor | Threading | Multiprocessing | Asyncio |
|--------|-----------|----------------|---------|
| Best for | I/O-bound | CPU-bound | I/O-bound (high concurrency) |
| Concurrency type | Preemptive | True parallelism | Cooperative |
| Memory | Shared | Separate | Shared (single thread) |
| GIL affected | Yes (CPU) | No | No |
| Startup cost | Low | High | Low |
| Max concurrency | ~1000 threads | ~CPU cores | 10,000+ tasks |
| Debugging | Hard (races) | Easier | Moderate |
| Communication | Shared + locks | Queue/Pipe | Direct (same thread) |
| Use case | Web scraping, DB queries | ML training, image processing | Web servers, proxies, chat |

### Common Mistakes

| Mistake | Issue | Fix |
|---------|-------|-----|
| Threading for CPU-bound | No speedup (GIL) | Use multiprocessing |
| No `if __name__ == '__main__'` with multiprocessing | Recursive spawning | Guard process creation |
| Forgetting `await` in asyncio | Coroutine never executed | Always await or create_task |
| Mixing asyncio and blocking I/O | Blocks event loop | Use loop.run_in_executor() |
| No timeout on join() | Hangs forever | join(timeout=N) |
| Shared mutable state without locks | Race conditions | Use locks or Queue |
| Creating too many threads | Overhead dominates | Use ThreadPoolExecutor |

> ✅ **Best Practice:** Use the `concurrent.futures` module as your default concurrency API. It provides a consistent interface for both thread and process pools, supports `as_completed()` for streaming results, and handles worker lifecycle automatically. Reserve raw `threading.Thread` and `multiprocessing.Process` only when you need fine-grained control.

### Best Practices

- Use `concurrent.futures` for pool management
- Use asyncio for I/O-bound with very high concurrency (1000+)
- Use `ProcessPoolExecutor` for CPU-bound work
- Use `ThreadPoolExecutor` for I/O-bound work
- Always protect shared state with locks
- Use Queue for producer/consumer patterns
- Set timeouts on all blocking operations

### Interview Questions

**Q: What is the GIL and why does it exist?**
A: The GIL is a mutex preventing multiple threads from executing Python bytecode simultaneously. It exists because CPython's memory management is not thread-safe. It simplifies implementation but limits CPU-bound parallelism.

**Q: When to use asyncio vs threading?**
A: Asyncio for very high I/O concurrency (10k+ connections) with cooperative multitasking. Threading for moderate I/O concurrency with blocking libraries that don't support async.

**Q: How does multiprocessing bypass the GIL?**
A: Each process has its own Python interpreter with its own GIL. Processes don't share memory by default, so the GIL per-process doesn't interfere with parallelism.

### Practical Exercises

1. Write a web scraper fetching 100 URLs using all three approaches
2. Implement producer/consumer with a bounded queue
3. Calculate pi using Monte Carlo with multiprocessing Pool
4. Implement a simple async HTTP server

### Mini Project: Concurrent File Processor

Build a tool that recursively processes all files using ThreadPoolExecutor for I/O-bound and ProcessPoolExecutor for CPU-bound files.

### Revision Notes

- **Threading**: GIL prevents CPU parallelism; good for I/O; use Lock, Queue, Event
- **Multiprocessing**: True parallelism; separate memory; use Pool, Queue, Pipe, Manager
- **Asyncio**: Single-threaded cooperative; async/await; best for 1000+ I/O tasks
- **concurrent.futures**: High-level interface for thread and process pools
- GIL released during I/O, sleep, and C extensions
- Always protect shared state; prefer Queue over shared memory

---

## Chapter 67: Type System and Testing

### Introduction

Python 3.5+ has optional type hints. Combined with modern testing tools (pytest, hypothesis, mock), they form a robust foundation for production code quality.

### Why It Matters

Type hints enable static analysis (mypy), better IDE support, and catch entire classes of bugs before runtime. Testing ensures correct behavior and prevents regressions.

### Real World Analogy

Type hints are architectural blueprints — they tell what connects to what before building. Tests are the building inspector — verifying every component works correctly.

### Type Hints

```python
from typing import (
    List, Dict, Tuple, Set, Optional, Union, Any,
    TypeVar, Generic, Callable, Iterator,
    Literal, Final, Protocol, TypedDict, NewType,
)

def greet(name: str) -> str:
    return f"Hello, {name}"

def process(items: List[int]) -> Dict[str, int]:
    return {str(i): i for i in items}

def find_user(user_id: int) -> Optional[str]:
    return users.get(user_id)

# TypeVar — generics
T = TypeVar('T')

def first(items: List[T]) -> T:
    return items[0]

class Stack(Generic[T]):
    def __init__(self) -> None:
        self._items: List[T] = []
    def push(self, item: T) -> None:
        self._items.append(item)
    def pop(self) -> T:
        return self._items.pop()

# Literal — specific values
Mode = Literal['read', 'write', 'append']

# Final — cannot be reassigned
MAX_RETRIES: Final = 3

# TypedDict — typed dictionaries
class Employee(TypedDict):
    name: str
    age: int
    department: str

# NewType — distinct types
UserID = NewType('UserID', int)
def get_user(uid: UserID) -> User: ...

# Callable
def apply(func: Callable[[int, int], int], a: int, b: int) -> int:
    return func(a, b)
```

### mypy Configuration

```ini
# mypy.ini
[mypy]
python_version = 3.11
strict = True
disallow_untyped_defs = True
no_implicit_optional = True
warn_redundant_casts = True
show_error_codes = True
```

```bash
mypy src/
mypy --strict src/
```

### Pydantic

```python
from pydantic import BaseModel, Field, field_validator, ConfigDict
from typing import Optional
from datetime import datetime

class User(BaseModel):
    model_config = ConfigDict(str_strip_whitespace=True)
    
    id: int
    name: str = Field(..., min_length=1, max_length=100)
    email: str
    age: Optional[int] = Field(None, ge=0, le=150)
    created_at: datetime = Field(default_factory=datetime.now)
    
    @field_validator('email')
    @classmethod
    def validate_email(cls, v: str) -> str:
        if '@' not in v:
            raise ValueError('Invalid email')
        return v.lower()

user = User(**{'id': 1, 'name': ' Alice ', 'email': 'Alice@Example.com'})
user.name   # 'Alice' (stripped)
user.email  # 'alice@example.com'
```

### pytest

```python
import pytest

# Fixtures
@pytest.fixture
def db_connection():
    conn = create_connection()
    yield conn
    conn.close()

# Fixture scopes
@pytest.fixture(scope='module')
def expensive_resource():
    return load_large_dataset()

@pytest.fixture(scope='session')
def config():
    return load_config()

# Autouse fixtures
@pytest.fixture(autouse=True)
def setup_test_db():
    setup_database()
    yield
    teardown_database()

# Parametrization
@pytest.mark.parametrize('input,expected', [
    (1, 2), (2, 4), (3, 6),
])
def test_double(input, expected):
    assert input * 2 == expected
```

#### pytest.mark

```python
@pytest.mark.skip(reason="Not implemented yet")
def test_future_feature():
    ...

@pytest.mark.skipif(sys.version_info < (3, 10), reason="Requires 3.10+")
def test_match_statement():
    ...

@pytest.mark.xfail(reason="Known bug", strict=True)
def test_known_bug():
    ...

@pytest.mark.slow
def test_performance():
    ...
```

### unittest vs pytest

| Feature | unittest | pytest |
|---------|----------|--------|
| Test discovery | unittest.TestLoader | Automatic (test_*.py) |
| Assertions | self.assertEqual() | assert (plain Python) |
| Fixtures | setUp/tearDown | @pytest.fixture (modular) |
| Parametrization | subTest() | @pytest.mark.parametrize |
| Plugins | Limited | Extensive ecosystem |
| Recommendation | Legacy projects | New projects |

### Mock

```python
from unittest.mock import Mock, MagicMock, patch

# MagicMock — comprehensive mock
mock = MagicMock()
mock.foo(1, 2, key='value')
mock.foo.assert_called_once_with(1, 2, key='value')

# Return values
mock.bar.return_value = 42
mock.bar()  # 42

# Side effects — sequence or callable
mock.baz.side_effect = [1, 2, 3, StopIteration]
mock.baz()  # 1

def dynamic_response(*args, **kwargs):
    if kwargs.get('error'):
        raise ValueError('error')
    return 'success'
mock.process.side_effect = dynamic_response

# patch — replace real objects
@patch('my_module.external_api')
def test_api_call(mock_api):
    mock_api.get.return_value.json.return_value = {'key': 'value'}
    result = my_module.process()
    mock_api.get.assert_called_once()

# spec — ensure mock only has real attributes
mock = MagicMock(spec=OrderedDict)
mock.keys()  # OK
mock.unknown_method()  # AttributeError
```

### Property-Based Testing

```python
from hypothesis import given, strategies as st, assume

@given(st.integers(), st.integers())
def test_commutative_add(a, b):
    assert a + b == b + a

@given(st.lists(st.integers()))
def test_reverse_twice(lst):
    assert lst[::-1][::-1] == lst

@given(st.lists(st.integers()))
def test_non_empty_lists(lst):
    assume(len(lst) > 0)
    assert max(lst) >= min(lst)
```

### Test Coverage

```bash
# pip install coverage pytest-cov
pytest --cov=src --cov-report=html --cov-report=term-missing
```

### Common Mistakes

| Mistake | Issue | Fix |
|---------|-------|-----|
| Over-mocking | Tests pass but code broken | Only mock external systems |
| Testing implementation | Brittle tests | Test public API |
| Not parametrizing tests | Copy-pasted tests | Use @pytest.mark.parametrize |
| Shared mutable fixtures | Test interference | scope='function' or yield fresh |
| Mocking without spec | Typo passes silently | Use spec=True |
| Using Any everywhere | Defeats type checking | Be specific with types |

### Best Practices

- Use type hints for all public function signatures
- Run mypy --strict in CI
- Prefer pytest over unittest for new projects
- Use conftest.py for shared fixtures
- Write one assertion per test (logically)
- Use hypothesis for property-based testing
- Mock at the import boundary (not internal call site)
- Aim for 80%+ coverage on critical paths

### Interview Questions

**Q: What is the difference between TypeVar and NewType?**
A: TypeVar creates a generic type parameter to preserve type relationships. NewType creates a distinct type for type-checking purposes — runtime value is the same, but mypy treats it as incompatible.

**Q: How does pytest fixture scoping work?**
A: 'function' (default) — per test. 'class' — per class. 'module' — per module. 'session' — once per run.

**Q: When to use mock.patch vs dependency injection?**
A: Use patch for testing code you can't modify (third-party libs). Prefer dependency injection for your own code.

### Practical Exercises

1. Add type hints to an existing module and verify with mypy
2. Write pytest fixtures for database connection with cleanup
3. Mock an external API and test error handling
4. Use hypothesis to test a sorting function

### Mini Project: Type-Safe API Client

Build an API client with Pydantic models for request/response validation, type hints, and comprehensive pytest tests with mocked HTTP calls.

### Revision Notes

- Type hints are optional but essential for production quality
- mypy --strict catches real bugs before runtime
- pytest with fixtures, parametrization, markers
- unittest.mock.patch for isolating external dependencies
- hypothesis for property-based testing
- Coverage: 80%+ is good; 100% rarely worth the effort
- TypedDict for typed dictionaries, Protocol for structural typing

---

## Chapter 68: Python for Data Science and Scripting

### Introduction

Python's standard library provides extensive modules for system scripting, command-line tools, and data processing. The scientific stack (NumPy, pandas, matplotlib) builds on these foundations.

### Why It Matters

Scripting and data processing are Python's original strengths. Understanding sys, os, subprocess, argparse, re, datetime, and related modules lets you automate infrastructure, process data, and build CLI tools.

### Real World Analogy

These modules are a Swiss Army knife for system administration. re is scissors (precision text cutting). datetime is the calendar/clock. subprocess is a telephone to other programs. argparse is the instruction manual for your tool.

### sys Module

```python
import sys

# Command-line arguments
print(sys.argv)  # ['script.py', '--name', 'Alice']

# Python path
sys.path.append('/custom/path')

# Standard streams
sys.stdout.write("Output\n")
sys.stderr.write("Error\n")
data = sys.stdin.read()

# Exit
sys.exit(0)       # success
sys.exit(1)       # error
sys.exit("Msg")   # prints message, exit 1

# Platform
sys.platform       # 'win32', 'linux', 'darwin'
sys.version        # '3.11.0 (...)'
sys.getsizeof(obj)
```

### os Module

```python
import os

# Environment
path = os.environ.get('PATH')
os.environ['MY_VAR'] = 'value'

# Directory operations
cwd = os.getcwd()
os.chdir('/tmp')
os.listdir('/tmp')
os.mkdir('/tmp/newdir')
os.makedirs('/tmp/a/b/c', exist_ok=True)
os.remove('/tmp/file.txt')
os.rename('old.txt', 'new.txt')

# Walk directory tree
for root, dirs, files in os.walk('/project'):
    for f in files:
        if f.endswith('.py'):
            print(os.path.join(root, f))

os.getpid()
os.cpu_count()
```

### shutil Module

```python
import shutil

shutil.copy('source.txt', 'dest.txt')
shutil.copy2('source.txt', 'dest/')         # preserve metadata
shutil.copytree('src/', 'dst/')
shutil.move('old.txt', 'new.txt')
shutil.rmtree('/tmp/workdir')               # rm -rf
shutil.make_archive('backup', 'zip', '/project/data')
shutil.unpack_archive('backup.zip', '/tmp/restore')
usage = shutil.disk_usage('/')
print(f"Free: {usage.free / 1e9:.1f} GB")
```

### subprocess Module

```python
import subprocess

result = subprocess.run(
    ['ls', '-la'],
    capture_output=True,
    text=True,
    check=True,
)
print(result.stdout)
print(result.returncode)

> ⚠️ **Warning:** NEVER use `shell=True` with unsanitized user input — it creates a shell injection vulnerability. An attacker could inject `; rm -rf /` or other commands. Always pass arguments as a list when possible, and use `shlex.quote()` if `shell=True` is unavoidable.

# AVOID shell=True with user input (shell injection risk)
subprocess.run(['python', 'script.py', input_arg])  # safe

# Popen for long-running processes
proc = subprocess.Popen(
    ['tail', '-f', '/var/log/syslog'],
    stdout=subprocess.PIPE,
    text=True
)
for line in proc.stdout:
    print(line.strip())
```

### argparse

```python
import argparse

parser = argparse.ArgumentParser(description='Process data')
parser.add_argument('input', help='Input file path')
parser.add_argument('-o', '--output', default='output.json', help='Output file')
parser.add_argument('-v', '--verbose', action='store_true', help='Verbose output')
parser.add_argument('--limit', type=int, default=100, help='Max items')
parser.add_argument('--format', choices=['json', 'csv'], default='json')

args = parser.parse_args()
print(args.input, args.output, args.verbose, args.limit, args.format)
```

### re Module

```python
import re

# Match at beginning
match = re.match(r'Hello', 'Hello World')
match.group() if match else None  # 'Hello'

# Search anywhere
match = re.search(r'World', 'Hello World')
match.span() if match else None  # (6, 11)

# Find all matches
re.findall(r'\d+', 'abc123def456')  # ['123', '456']

# Find with overlap (using lookahead)
re.findall(r'(?=(\d\d))', '12345')  # ['12', '23', '34', '45']

# Substitution
re.sub(r'\s+', '_', 'hello world python')  # 'hello_world_python'

# Compile for reuse
pattern = re.compile(r'\b\w+\b')
pattern.findall('Hello, World!')  # ['Hello', 'World']
```

### enum Module

```python
from enum import Enum, auto, IntEnum, Flag

class Color(Enum):
    RED = 1
    GREEN = 2
    BLUE = 3

class Status(IntEnum):
    PENDING = 0
    ACTIVE = 1
    COMPLETED = 2

class Permissions(Flag):
    READ = auto()
    WRITE = auto()
    EXECUTE = auto()

Color.RED.name      # 'RED'
Color.RED.value     # 1
Status.PENDING < Status.ACTIVE  # True (int comparison)
Permissions.READ | Permissions.WRITE  # Permissions.READ|WRITE
```

### datetime Module

```python
from datetime import datetime, date, time, timedelta, timezone

now = datetime.now()
today = date.today()
current_time = now.time()

# Create specific date
dt = datetime(2024, 6, 15, 14, 30, 0)

# Arithmetic
tomorrow = today + timedelta(days=1)
delta = datetime(2024, 12, 31) - datetime(2024, 1, 1)
delta.days  # 365

# Timezone
utc_now = datetime.now(timezone.utc)
eastern = timezone(timedelta(hours=-5))
eastern_now = now.astimezone(eastern)

# Formatting
now.strftime('%Y-%m-%d %H:%M:%S')  # '2024-06-15 14:30:00'
datetime.strptime('2024-06-15', '%Y-%m-%d')  # datetime(2024, 6, 15)
```

### math and statistics

```python
import math
import statistics

data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

math.sqrt(16)           # 4.0
math.factorial(5)       # 120
math.comb(10, 3)        # 120 (combinations)
math.perm(10, 3)        # 720 (permutations)
math.gcd(12, 8)         # 4
math.isclose(0.1 + 0.2, 0.3)  # True

statistics.mean(data)   # 5.5
statistics.median(data) # 5.5
statistics.stdev(data)  # ~3.03
statistics.mode([1, 1, 2, 3])  # 1
```

### random and secrets

```python
import random
import secrets

# random — general purpose
random.seed(42)           # reproducible results
random.random()           # [0.0, 1.0)
random.randint(1, 100)    # int in [1, 100]
random.choice(['a', 'b', 'c'])
random.sample(range(100), 10)  # 10 unique items
random.shuffle(items)     # in-place shuffle

# secrets — cryptographically secure
secrets.token_hex(16)     # 32 character hex string
secrets.token_urlsafe(32) # URL-safe base64 string
secrets.choice('abcdef')  # secure random choice
```

### hashlib and uuid

```python
import hashlib
import uuid

# Hashing
hashlib.md5(b"data").hexdigest()
hashlib.sha256(b"data").hexdigest()
hashlib.sha256(b"data").digest()  # raw bytes

# HMAC
import hmac
key = b"secret"
msg = b"message"
hmac.new(key, msg, hashlib.sha256).hexdigest()

# UUID
uuid.uuid4()          # random UUID: 'f47ac10b-58cc-...'
uuid.uuid1()          # based on host + time
uuid.NAMESPACE_DNS    # predefined namespace
uuid.uuid5(uuid.NAMESPACE_DNS, 'example.com')  # deterministic
```

### glob and fnmatch

```python
import glob
import fnmatch

# glob — file pattern matching
glob.glob('*.py')           # all .py files in current dir
glob.glob('**/*.py', recursive=True)  # recursive

# fnmatch — unix-style matching
fnmatch.fnmatch('hello.py', '*.py')  # True
fnmatch.filter(['a.py', 'b.txt', 'c.py'], '*.py')  # ['a.py', 'c.py']
```

### Common Mistakes

| Mistake | Issue | Fix |
|---------|-------|-----|
| shell=True with user input | Command injection | Pass list args, not string |
| Using os.path over pathlib | String-based, error-prone | Use pathlib.Path |
| Not setting seed in random | Non-reproducible results | random.seed() for tests |
| datetime naive vs aware | Timezone confusion | Use timezone.utc or pytz |
| Popen without communicate | Zombie processes | Always call communicate() or wait() |
| Regex without raw string | Backslash escaping errors | Use r'pattern' raw strings |

### Best Practices

- Prefer pathlib.Path over os.path
- Use subprocess.run() over os.system()
- Never use shell=True with untrusted input
- Use secrets module for crypto, random for non-security
- Always specify encoding='utf-8' for file operations
- Compile regex patterns with re.compile() for reuse
- Use argparse over sys.argv for production CLI tools
- Use pathlib.glob/rglob over glob module

### Interview Questions

**Q: What is the difference between re.match and re.search?**
A: re.match checks from the beginning of the string only. re.search scans the entire string for the first match.

**Q: Why should you avoid shell=True in subprocess?**
A: shell=True passes the command string to the system shell, creating shell injection vulnerabilities. Use a list of arguments instead.

**Q: What is the difference between random and secrets?**
A: random uses Mersenne Twister (deterministic, reproducible). secrets uses OS-provided cryptographic randomness (secure for passwords, tokens).

### Practical Exercises

1. Write a CLI tool using argparse that processes CSV files
2. Recursively find and rename files using pathlib
3. Implement a simple grep clone using re module
4. Generate UUIDs and hash passwords with hashlib

### Mini Project: Log Analyzer

Build a CLI log analyzer that reads log files, parses timestamps with datetime, extracts patterns with regex, and reports statistics.

### Revision Notes

- sys: argv, path, stdin/stdout/stderr, exit
- os/subprocess: system operations, process management
- pathlib: modern file path handling (prefer over os.path)
- argparse: robust CLI argument parsing
- re: regex via match/search/findall/sub/compile
- datetime: date/time arithmetic, formatting, timezone
- random (Mersenne Twister) vs secrets (crypto-safe)
- hashlib/uuid: hashing and identifier generation
- glob/fnmatch: file pattern matching

---

## Chapter 69: Python Packaging and Distribution

### Introduction

Python packaging has evolved significantly. The modern standard is pyproject.toml (PEP 517/518/621) with tools like setuptools, Poetry, and pip.

### Why It Matters

Packaging enables sharing code, creating installable CLI tools, managing dependencies, and distributing libraries to PyPI. Modern packaging is the difference between "works on my machine" and deployable software.

### Real World Analogy

Packaging is like preparing a meal kit for shipping. The recipe (pyproject.toml) lists ingredients (dependencies). The packaging (wheel) is the box. The distribution (PyPI) is the store. Installation (pip) is unboxing the kit in your kitchen.

### pyproject.toml

```toml
[build-system]
requires = ["setuptools>=68.0", "wheel"]
build-backend = "setuptools.backends._legacy:_Backend"

[project]
name = "mypackage"
version = "0.1.0"
description = "My awesome package"
readme = "README.md"
requires-python = ">=3.9"
license = {text = "MIT"}
authors = [
    {name = "Alice", email = "alice@example.com"},
]

dependencies = [
    "requests>=2.28",
    "click>=8.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "mypy>=1.0",
    "coverage>=7.0",
]

[project.urls]
homepage = "https://github.com/alice/mypackage"

[project.scripts]
mytool = "mypackage.cli:main"
```

### setuptools

```python
# setup.py (legacy — prefer pyproject.toml)
from setuptools import setup, find_packages

setup(
    name="mypackage",
    version="0.1.0",
    packages=find_packages(include=['mypackage', 'mypackage.*']),
    install_requires=[
        "requests>=2.28",
    ],
    entry_points={
        'console_scripts': [
            'mytool=mypackage.cli:main',
        ],
    },
    python_requires=">=3.9",
)
```

### pip

```bash
# Install from current directory
pip install -e .                # editable install (development mode)
pip install -e ".[dev]"         # with dev dependencies

# Requirements file
pip install -r requirements.txt

# Constraints file (pin versions without requiring install)
pip install -c constraints.txt

# Freeze current environment
pip freeze > requirements.txt
```

Requirements vs constraints:
| File | Purpose |
|------|---------|
| requirements.txt | Exact packages to install |
| constraints.txt | Version limits applied only when package is installed by other means |

### Poetry

```toml
# pyproject.toml (Poetry)
[tool.poetry]
name = "mypackage"
version = "0.1.0"
description = ""
authors = ["Alice <alice@example.com>"]

[tool.poetry.dependencies]
python = "^3.9"
requests = "^2.28"
click = "^8.0"

[tool.poetry.group.dev.dependencies]
pytest = "^7.0"
mypy = "^1.0"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

```bash
poetry new mypackage           # scaffold new project
poetry add requests            # add dependency
poetry add --group dev pytest  # dev dependency
poetry install                 # install all dependencies
poetry build                   # build wheel + sdist
poetry publish                 # publish to PyPI
```

### Namespace Packages

Namespace packages allow splitting a package across multiple distributions:

```python
# In mypackage/a/__init__.py and mypackage/b/__init__.py
# Both declare the same namespace without __init__.py in parent

# mypackage_a/pyproject.toml
# [project]
# name = "mypackage-a"
#
# mypackage_b/pyproject.toml
# [project]
# name = "mypackage-b"

# Both expose under the 'mypackage' namespace:
import mypackage.a
import mypackage.b
```

### CLI Tools with entry_points

```python
# mypackage/cli.py
def main():
    import argparse
    parser = argparse.ArgumentParser()
    parser.add_argument("name")
    args = parser.parse_args()
    print(f"Hello, {args.name}!")
```

After pip install -e ., the `mytool` command is available globally.

### Wheels vs Source Distributions

| Feature | Wheel (.whl) | sdist (.tar.gz) |
|---------|-------------|-----------------|
| Install speed | Fast (unzip) | Slow (run setup.py) |
| Platform-specific | Can be (manylinux, win32) | Cross-platform |
| C extensions | Pre-built | Must be compiled |
| Metadata | Immediate | Requires building |
| Pip priority | First choice | Fallback |

```bash
# Build both
python -m build

# Check wheel content
unzip -l dist/*.whl
```

### Publishing to PyPI

```bash
# Set up PyPI token
# Create API token at https://pypi.org/manage/account/token/

# Install twine
pip install twine

# Build
python -m build

# Upload
twine upload dist/*

# Test PyPI
twine upload --repository-url https://test.pypi.org/legacy/ dist/*
pip install --index-url https://test.pypi.org/simple/ mypackage

# .pypirc file (optional)
# [pypi]
# username = __token__
# password = pypi-xxxxx
```

### Common Mistakes

| Mistake | Issue | Fix |
|---------|-------|-----|
| Not pinning dependencies | Version conflicts | Pin upper bounds for stability |
| Forgetting .gitignore | Committing pycache, dist | Add to .gitignore |
| Missing long_description_content_type | PyPI formatting broken | Set to 'text/markdown' |
| Not including MANIFEST.in | Missing files in sdist | Include data files explicitly |
| Uploading without testing | Broken package | Test on TestPyPI first |
| Wrong build-system config | Build errors | Use modern pyproject.toml |
| Not checking name availability | Upload rejection | Check name on PyPI first |

### Best Practices

- Use pyproject.toml (not setup.py) for new projects
- Publish both wheel and sdist
- Test on TestPyPI before production PyPI
- Use semantic versioning (MAJOR.MINOR.PATCH)
- Pin dependency upper bounds for stability (but leave room)
- Include a LICENSE file
- Write a detailed README with installation and usage
- Add type stubs or use PEP 561 (py.typed marker)

### Interview Questions

**Q: What is the difference between setup.py and pyproject.toml?**
A: pyproject.toml (PEP 517/518/621) is the modern standard. It declares build-system requirements and project metadata declaratively. setup.py is legacy, imperative, and slower.

**Q: What is an editable install (pip install -e)?**
A: It installs the package as a symlink to the source directory. Changes to source are immediately available without re-installing. Used during development.

**Q: Why prefer wheels over sdists?**
A: Wheels are pre-built, faster to install, and don't require running setup.py or compiling C extensions. They skip the build step entirely.

### Practical Exercises

1. Convert a script into an installable package with pyproject.toml
2. Create a CLI tool with entry_points
3. Publish a package to TestPyPI
4. Add optional dependencies and install with extras

### Mini Project: Python Package Publisher

Build a package with CLI tool, comprehensive testing, and an automated publish script.

### Revision Notes

- pyproject.toml (PEP 517/518/621) is the modern standard
- setuptools, flit, poetry are build backends
- pip install -e . for editable development installs
- entry_points = console_scripts for CLI tools
- wheel (.whl) for fast install; sdist (.tar.gz) for source
- Test on TestPyPI before production upload
- Use twine for uploading; API tokens for authentication

---

## Chapter 70: Performance Optimization in Python

### Introduction

Python is not the fastest language, but with profiling, smart data structures, and the right tools, you can achieve significant speedups. Optimization is about measuring first, then targeting bottlenecks.

### Why It Matters

A slow script that runs once is fine. A slow API endpoint handling millions of requests is a disaster. Performance optimization is essential for production systems, especially in ML pipelines where every iteration costs time and money.

### Real World Analogy

Optimization is like renovating a house. You find the drafty windows (bottlenecks) and fix them. You don't replace the perfectly fine foundation. Measure first (profiling), then cut where it matters.

### Profiling: cProfile and timeit

```python
import cProfile
import pstats

# Profile a function
def slow_function():
    total = 0
    for i in range(10**6):
        total += i ** 2
    return total

cProfile.run('slow_function()', 'profile_stats')

# Analyze results
p = pstats.Stats('profile_stats')
p.sort_stats('cumulative').print_stats(10)  # top 10 by cumulative time
p.sort_stats('time').print_stats(10)        # top 10 by internal time

# timeit — micro-benchmarking
import timeit

timeit.timeit('"-".join(str(n) for n in range(100))', number=10000)

# From IPython/Jupyter:
# %timeit [x**2 for x in range(1000)]
```

### __slots__ for Memory Reduction

```python
class WithSlots:
    __slots__ = ('x', 'y')
    def __init__(self, x, y):
        self.x = x
        self.y = y

class WithoutSlots:
    def __init__(self, x, y):
        self.x = x
        self.y = y

import sys
w = WithoutSlots(1, 2)
s = WithSlots(1, 2)
sys.getsizeof(w)  # ~56 + 120 (__dict__) = 176 bytes
sys.getsizeof(s)  # ~56 bytes (no __dict__)

# For 10 million instances: 1.2 GB savings
```

### Using Built-in Functions

```python
from operator import add

# Slow: manual loop
total = 0
for i in range(10**7):
    total += i

# Fast: built-in sum
total = sum(range(10**7))  # 2-3x faster (C loop, not Python)

# map with existing function
list(map(str.upper, items))  # faster than comprehension for large lists
```

### Local vs Global Variable Speed

```python
# Slow: global variable access
x = 42
def slow():
    for i in range(10**7):
        _ = x ** 2

# Fast: local variable binding
def fast():
    local_x = x  # bind to local
    for i in range(10**7):
        _ = local_x ** 2

# ~30% faster due to LOAD_FAST vs LOAD_GLOBAL bytecode
```

### String Concatenation

```python
# Slow: O(n^2) — creates new string each iteration
s = ""
for piece in pieces:
    s += piece

# Fast: O(n) — efficient single allocation
s = "".join(pieces)

# For medium-sized lists, join is 10-50x faster
```

### List vs array vs numpy

| Operation | list | array.array | numpy.ndarray |
|-----------|------|-------------|---------------|
| Creation | `[0]*n` | `array('i', [0])*n` | `np.zeros(n)` |
| Element access | O(1) | O(1) | O(1) |
| Arithmetic | Python loop | Python loop | C-optimized vectorized |
| Memory | 28 bytes/element | 4 bytes/element (type 'i') | 8 bytes/element (float64) |
| 1M element sum | ~50ms | ~50ms | ~1ms |
| Broadcasting | No | No | Yes |

```python
# numpy vectorization
import numpy as np

a = np.random.rand(10**6)
b = np.random.rand(10**6)

# Vectorized (fast)
c = a * b + np.sin(a)

# Python loop (slow)
c = [a[i] * b[i] + math.sin(a[i]) for i in range(10**6)]
# numpy is ~100x faster
```

### Cython Basics

```python
# example.pyx
def fibonacci(int n):
    cdef int i
    cdef long long a = 0, b = 1, c
    if n == 0:
        return a
    for i in range(1, n):
        c = a + b
        a = b
        b = c
    return b

# setup.py
from setuptools import setup
from Cython.Build import cythonize

setup(ext_modules=cythonize('example.pyx'))
```

Cython compiles Python-like code to C extensions, achieving 10-100x speedups for numeric code.

### Numba JIT

```python
from numba import jit
import numpy as np

@jit(nopython=True)
def sum_squares(n):
    total = 0
    for i in range(n):
        total += i ** 2
    return total

# First call compiles, subsequent calls run at machine code speed
sum_squares(10**7)  # >100x faster than pure Python

# Vectorized functions
@jit(nopython=True, parallel=True)
def parallel_sum(arr):
    total = 0
    for i in range(len(arr)):
        total += arr[i] ** 2
    return total

arr = np.random.rand(10**7)
parallel_sum(arr)  # uses all CPU cores
```

### C Extensions

```python
# ctypes — call C libraries from Python
import ctypes

libc = ctypes.CDLL('libc.so.6')
libc.printf(b"Hello from C!\n")

# CFFI — cleaner interface
from cffi import FFI
ffi = FFI()
ffi.cdef("int printf(const char *format, ...);")
lib = ffi.dlopen('libc.so.6')
lib.printf(b"Hello from C!\n")

# CPython C API (most complex, most powerful)
# Write C extension module, compile, import from Python
```

### Memory Profiling

```python
# memory_profiler
# pip install memory_profiler

@profile  # run with: python -m memory_profiler script.py
def memory_intensive():
    a = [i for i in range(10**7)]
    b = [i * 2 for i in range(10**6)]
    del a
    return b

# tracemalloc — built-in memory tracking
import tracemalloc

tracemalloc.start()
snapshot1 = tracemalloc.take_snapshot()

# ... code to profile ...

snapshot2 = tracemalloc.take_snapshot()
stats = snapshot2.compare_to(snapshot1, 'lineno')
for stat in stats[:10]:
    print(stat)

# Memory usage per line
top_stats = tracemalloc.get_traced_memory()
print(f"Current: {top_stats[0] / 1024:.1f} KB")
```

### Common Optimization Patterns

| Pattern | Slow | Fast |
|---------|------|------|
| String building | `s += x` in loop | `''.join(list)` |
| Attribute lookups in loops | `self.x` each iteration | `x = self.x` before loop |
| Function calls in tight loops | `fn()` each iteration | Inline or use local ref |
| List membership | `if x in lst` (list) | `if x in set_items` (set) |
| Loops over numpy arrays | `for i in range(n):` | Vectorized operations |
| Deep attribute access | `obj.a.b.c` each time | Cache in local variable |
| Repeated method calls | `[x.foo() for x in items]` twice | Assign to variable |
| Middle insertion | `lst.insert(0, x)` | `deque.appendleft(x)` |

### Common Mistakes

| Mistake | Issue | Fix |
|---------|-------|-----|
| Optimizing without profiling | Wasted effort on non-bottlenecks | Profile first |
| Premature optimization | Complex, unreadable code | Write clean code, then profile |
| Using numpy for small data | Overhead dominates | List for small (<1000) |
| Ignoring memory | OOM crashes for large data | Use generators, profile memory |
| Using threads for CPU work | No speedup due to GIL | Use multiprocessing |
| Wrong data structure | O(n) instead of O(1) | Use dict/set for lookups |
| Not caching repeated work | Recomputation | Use lru_cache or manual cache |
| Too many small allocations | GC pressure | Reuse objects, pool connections |

> 📖 **Instructor Note:** The 90/10 rule (90% of execution time is spent in 10% of code) is the fundamental insight of performance optimization. Always profile FIRST — your intuition about what's slow is often wrong. Optimize the hot paths, then re-profile to verify improvement.

### Best Practices

- Profile before optimizing (90% of time is in 10% of code)
- Use built-in functions and libraries (they're C-optimized)
- Use vectorization (numpy) over Python loops for numeric work
- Use local variable bindings in hot loops
- Choose the right data structure (dict/set for O(1) lookups)
- Use generators for large data streams
- Consider __slots__ for millions of objects
- Use Cython/Numba for numeric computation hot spots
- Memory is often the real bottleneck — profile it too

### Interview Questions

**Q: How would you profile a slow Python script?**
A: Start with cProfile to identify hot functions. Use pstats or snakeviz to visualize. Then use timeit for micro-benchmarks of specific approaches. Profile memory with memory_profiler or tracemalloc.

**Q: What is the single most effective optimization technique?**
A: Choosing the right data structure. Replacing a list with a set for membership checks changes O(n) to O(1). This dwarfs most micro-optimizations.

**Q: When should you use numpy over plain Python?**
A: When processing large numeric arrays (10,000+ elements) with arithmetic operations. Numpy vectorization runs at C speed and supports broadcasting, reducing memory allocations.

### Practical Exercises

1. Profile a function with cProfile and identify the bottleneck
2. Benchmark list vs set for membership of 1 million items
3. Convert a numeric loop to use numpy vectorization
4. Use __slots__ and compare memory usage for 1 million objects

### Mini Project: Performance Benchmark Suite

Build a benchmarking tool that compares different implementations of common operations (string concatenation, sorting, searching) with automated reporting.

### Revision Notes

- Profile first: cProfile, timeit, memory_profiler, tracemalloc
- __slots__: eliminates __dict__, saves memory
- Local variable binding: LOAD_FAST > LOAD_GLOBAL
- ''.join() > += for string concatenation
- numpy vectorization: 100x faster than Python loops
- Cython: Python with C types, compile to C extension
- Numba @jit: JIT compile numeric functions to machine code
- ctypes/CFFI: call C libraries from Python
- Right data structure is the most impactful optimization

---

*This concludes Part 10. Python is not just a language — it is the backbone of modern AI, data science, and backend development. Mastery of these topics separates engineers who can write code from engineers who can build systems.*
