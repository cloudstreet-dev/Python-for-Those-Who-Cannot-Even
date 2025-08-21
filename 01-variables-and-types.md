# Chapter 1: Variables and Types (Or: How Python Pretends Types Don't Matter)

*"Dynamic typing is great until it isn't."* - Every Python developer at 3 AM

Remember when you were young and optimistic, writing C++ or Java, and the compiler would tell you when you were being stupid? Those were the days. Welcome to Python, where types are made up and the points don't matter until runtime.

## Everything Is An Object (Even Your Disappointment)

In Python, everything is an object. And I mean *everything*. Numbers? Objects. Strings? Objects. Functions? Objects. The existential dread you feel when debugging a TypeError at runtime? Probably also an object.

```python
x = 42
print(type(x))  # <class 'int'>
print(x.__class__)  # <class 'int'>
print(x.__sizeof__())  # 28 bytes for the number 42. TWENTY-EIGHT.
```

That's right. The number 42 takes 28 bytes in Python. In C, it would take 4. This is the price of "everything is an object." It's like shipping a single grape in a refrigerator box.

## Variable Assignment (Or: Names Bound to Objects, If We're Being Pedantic)

In Python, variables aren't really variables. They're names bound to objects. This distinction matters exactly never, except when Python developers want to sound smart at parties nobody invited them to.

```python
x = 5  # x is now bound to the integer object 5
y = x  # y is now bound to the same integer object
x = 10  # x is now bound to a different integer object
print(y)  # Still 5, because y didn't move
```

In Ruby, we'd just say "x is 5" and move on with our lives. But Python needs you to understand that x isn't really 5, it's just pointing at 5, like a very confused GPS.

## Dynamic Typing: Because Who Needs Compile-Time Safety?

Python's type system is like a speed limit sign in Italy: technically there, but nobody's really paying attention.

```python
age = 25
print(f"I am {age} years old")

age = "twenty-five"  # Sure, why not
print(f"I am {age} years old")

age = ["twenty", "five"]  # This is fine
print(f"I am {age} years old")  # I am ['twenty', 'five'] years old

age = None  # Everything is fine
print(f"I am {age} years old")  # I am None years old (mood)
```

No compiler errors. No warnings. Just vibes.

## The Type System (Such As It Is)

Python has types. It just doesn't care about them until it absolutely has to.

### The Built-in Types Nobody Asked For

```python
# Integers (at least these are infinite precision)
x = 42
y = 1_000_000_000_000_000_000_000_000  # Python's fine with this
z = 0b1010  # Binary, because why not
w = 0o12  # Octal, for people who hate themselves
v = 0xFF  # Hexadecimal, the only alternative notation that matters

# Floats (IEEE 754 double precision, like everyone else)
pi = 3.14159
avogadro = 6.022e23  # Scientific notation works
sadness = float('inf')  # Infinite sadness is a valid float

# Complex numbers (because Python thinks it's MATLAB)
imaginary_friend = 3 + 4j
print(imaginary_friend.real)  # 3.0
print(imaginary_friend.imag)  # 4.0
# Why does Python have built-in complex numbers but not a switch statement until 3.10?

# Booleans (the most confused type)
this_is_good = True
this_is_python = False
# Fun fact: True is 1 and False is 0
print(True + True)  # 2. Because of course it is.
print(True == 1)  # True
print(False == 0)  # True
# Booleans are just integers in a trench coat

# Strings (at least these are good)
single = 'Single quotes work'
double = "Double quotes also work"
triple = """Triple quotes for when you have
a lot to complain about"""
raw = r"Raw strings don't process \n escapes"
formatted = f"F-strings are {single}"

# Bytes (for when strings aren't confusing enough)
just_bytes = b"These are bytes"
print(just_bytes[0])  # 84 (the ASCII value of 'T')
# Because accessing a byte string should definitely return integers

# NoneType (the loneliest type)
nothing = None
# None is Python's null, nil, undefined, whatever
# It's the only instance of NoneType
# It's falsy, it's nothing, it's perfect for representing my enthusiasm
```

## Type Checking (Or: How to Pretend Python Has Types)

Since Python 3.5, you can add type hints. They're like comments that look important but do absolutely nothing at runtime.

```python
def add(x: int, y: int) -> int:
    return x + y

result = add("hello", "world")  # This works fine
print(result)  # "helloworld"
```

The type hints are lies. Python doesn't care. It's like putting a "Please" sign on a door that's already open.

Want actual type checking? You need a third-party tool:

```bash
pip install mypy
mypy your_file.py
# error: Argument 1 to "add" has incompatible type "str"; expected "int"
```

"But wait," you say, "I have to install a separate tool to check the types I wrote in my code?" Yes. Yes, you do. Welcome to Python.

## The Walrus Operator := (Yes, That's Really What It's Called)

Python 3.8 introduced the walrus operator, because apparently what Python was missing was more operators that look like emoticons.

```python
# Old way (how normal people write code)
data = input("Enter something: ")
if data:
    print(f"You entered: {data}")

# New way (with walrus)
if (data := input("Enter something: ")):
    print(f"You entered: {data}")

# Why though?
# It's useful in while loops, I guess
while (line := input(">>> ")) != "quit":
    print(f"You said: {line}")

# In Ruby, we'd just write:
# while (line = gets.chomp) != "quit"
# But Python needed a special operator for this. Because Python.
```

It's called the walrus operator because `:=` looks like a walrus on its side. This is the level of decision-making we're dealing with.

## Truthiness and Falsiness (Everything Is Something Until It Isn't)

In Python, everything has a truth value, and the rules are about as consistent as a coin flip in a tornado.

```python
# These are falsy:
bool(False)  # False (duh)
bool(None)   # False
bool(0)      # False
bool(0.0)    # False
bool(0j)     # False (even complex zero is false)
bool('')     # False (empty string)
bool([])     # False (empty list)
bool(())     # False (empty tuple)
bool({})     # False (empty dict)
bool(set())  # False (empty set)

# Everything else is truthy:
bool(True)   # True (duh)
bool(42)     # True
bool(-1)     # True (even negative numbers)
bool(0.1)    # True
bool('0')    # True (string '0' is not numeric 0)
bool('False') # True (it's a non-empty string)
bool([0])    # True (list with falsy element is still truthy)
bool(' ')    # True (space is not empty)

# This leads to fun bugs:
def process_items(items=None):
    items = items or []  # Seems reasonable
    # But what if someone passes an empty list intentionally?
    # Too bad, they get a new list!
    
# The Pythonic way:
def process_items(items=None):
    if items is None:
        items = []
    # More verbose, but at least it's correct
```

## Identity vs Equality (Or: is vs ==)

Python has two ways to compare things, because one would be too simple.

```python
# == checks equality (value)
a = [1, 2, 3]
b = [1, 2, 3]
print(a == b)  # True, same values

# is checks identity (same object)
print(a is b)  # False, different objects

c = a
print(a is c)  # True, same object

# The weird part:
x = 256
y = 256
print(x is y)  # True (!!!)

x = 257
y = 257
print(x is y)  # False (??????)

# Python caches small integers (-5 to 256) for performance
# So they're the same object
# This is called "interning" and it's as weird as it sounds

# Also, never do this:
if x == None:  # Bad
    pass
    
if x is None:  # Good
    pass
    
# Unless you overrode __eq__, then == might not work
# Just use 'is None' and move on with your life
```

## Type Conversion (Coercion Is Too Strong a Word)

Python won't automatically convert types for you, except when it will.

```python
# Explicit is better than implicit (supposedly)
x = "5"
y = 3
# z = x + y  # TypeError: can only concatenate str (not "int") to str

# You must explicitly convert:
z = int(x) + y  # 8
# or
z = x + str(y)  # "53"

# But then there's this:
print(5 + True)  # 6 (because True is 1, remember?)
print("ha" * 3)  # "hahaha" (string multiplication is a thing)
print([1, 2] * 2)  # [1, 2, 1, 2] (so is list multiplication)

# And division always returns a float:
print(10 / 2)  # 5.0, not 5
print(10 // 2)  # 5 (floor division if you want an int)

# The zen of Python: "Explicit is better than implicit"
# Python: "Hold my implicitly converted boolean"
```

## Variable Scope (Local, Global, and Nonlocal Walk Into a Bar)

Python's scoping rules are like a mystery novel where the butler did it, but also the butler doesn't exist unless you explicitly declare him.

```python
x = "global"

def outer():
    x = "outer"
    
    def inner():
        x = "inner"
        print(f"Inner x: {x}")
    
    inner()
    print(f"Outer x: {x}")

outer()
print(f"Global x: {x}")

# Output:
# Inner x: inner
# Outer x: outer  
# Global x: global

# So far so good, but then:

def broken():
    print(x)  # UnboundLocalError: local variable 'x' referenced before assignment
    x = "local"

# Python sees x is assigned in the function, so it's local
# But you're trying to use it before assignment
# Thanks, Python

# To fix it:
def fixed():
    global x
    print(x)  # Now it works
    x = "modified global"  # This changes the global x

# And for nested functions:
def outer2():
    x = "outer"
    
    def inner2():
        nonlocal x  # Because 'global' would be too simple
        x = "modified outer"
    
    inner2()
    print(x)  # "modified outer"

# In Ruby: @instance, @@class, $global, local
# In Python: local, nonlocal, global, builtin, and suffering
```

## The Mutable Default Argument Trap

This is Python's favorite way to hurt you:

```python
def append_to_list(item, target=[]):
    target.append(item)
    return target

list1 = append_to_list(1)
list2 = append_to_list(2)

print(list1)  # [1, 2]  Wait, what?
print(list2)  # [1, 2]  They're the same list!

# The default [] is created ONCE when the function is defined
# Not each time it's called
# This is everyone's favorite Python interview question

# The fix:
def append_to_list_fixed(item, target=None):
    if target is None:
        target = []
    target.append(item)
    return target

# Why is it like this? Because Python evaluates default arguments
# at function definition time, not call time
# Why? Because Guido said so
```

## Constants (Just Kidding, Python Doesn't Have Constants)

```python
# Python convention: CAPS means constant
PI = 3.14159
GRAVITY = 9.8
SPEED_OF_LIGHT = 299_792_458

# But nothing stops you from doing this:
PI = 4  # Python: "Sure, why not?"

# Want actual constants? Too bad!
# Best you can do is:
from typing import Final  # Python 3.8+

MAX_SIZE: Final = 100
# MAX_SIZE = 200  # Your IDE might complain, but Python won't

# Or use a class with properties (ugh):
class Constants:
    @property
    def PI(self):
        return 3.14159

const = Constants()
print(const.PI)  # 3.14159
# const.PI = 4  # AttributeError! Finally!
# But who's going to write all that?

# In other languages: const, final, readonly, #define
# In Python: "Please don't change this, pretty please"
```

## Type Annotations: The Lies We Tell Ourselves

Python 3.5+ lets you add type annotations. They're like promises you make to yourself and then immediately break.

```python
from typing import List, Dict, Optional, Union, Tuple, Any

# Basic annotations
name: str = "Python"
age: int = 33
height: float = 5.9
is_typed: bool = False  # Accurate

# Collection types
numbers: List[int] = [1, 2, 3]
scores: Dict[str, int] = {"Alice": 100, "Bob": 95}
coordinates: Tuple[float, float] = (10.5, 20.3)

# Optional (can be None)
middle_name: Optional[str] = None  # or str | None in Python 3.10+

# Union types (can be multiple types)
identifier: Union[int, str] = "ID123"  # or int | str in Python 3.10+

# Any (giving up)
data: Any = "Could be anything"

# Function annotations
def greet(name: str, times: int = 1) -> str:
    return f"Hello {name}! " * times

# Complex annotations
def process_data(
    items: List[Dict[str, Union[str, int]]]
) -> Optional[Dict[str, Any]]:
    """This signature is longer than the function."""
    return {"processed": True} if items else None

# The beauty is, you can lie:
numbers.append("not a number")  # Python: "Looks good to me!"
result = greet(123, "oops")  # Python: "Sure thing, boss!"

# Remember: These are just hints
# Python ignores them completely at runtime
# They're documentation that looks like code
```

## Exercises: Try Not to Cry

1. **The Identity Crisis**: 
   ```python
   a = 256
   b = 256
   c = 257
   d = 257
   ```
   Without running the code, predict the output of:
   - `a is b`
   - `c is d`
   Now run it and contemplate why Python is like this.

2. **The Mutable Default Disaster**:
   Write a function that has a mutable default argument. Call it multiple times. Watch the chaos. Now fix it and pretend this is normal.

3. **Type Hint Lies**:
   ```python
   def divide(a: int, b: int) -> int:
       return a / b
   ```
   What's wrong with this type hint? (Hint: division always returns float in Python 3)

4. **The Truthy Tournament**:
   Without running the code, predict if these are truthy or falsy:
   - `bool("False")`
   - `bool(0.0)`
   - `bool([])`
   - `bool(" ")`
   - `bool([False])`

5. **Scope Shenanigans**:
   ```python
   x = 1
   def confusing():
       print(x)
       x = 2
       print(x)
   ```
   What happens when you call `confusing()`? Why? How would you fix it?

6. **The Walrus Challenge**:
   Rewrite this code using the walrus operator:
   ```python
   data = expensive_operation()
   if data is not None:
       process(data)
   ```
   Now ask yourself: was it worth it?

## Summary: What We've Learned

1. **Everything is an object**, even when it shouldn't be
2. **Variables are just names** pointing at objects, like very needy labels
3. **Types exist** but Python only cares when it's inconvenient
4. **Type hints are decoration**, not enforcement
5. **The walrus operator exists** because `:=` looks like a walrus
6. **Truthiness is weird**: empty things are false, except when they're not
7. **Mutable defaults are traps** set by Guido himself
8. **Constants don't exist**, only strongly-worded suggestions

In the next chapter, we'll explore control flow, where Python's meaningful whitespace really starts to shine (and by shine, I mean drive you insane). We'll learn about loops that have else clauses, because apparently that's a thing someone wanted.

Until then, remember: in Python, types are optional, but suffering is guaranteed.

---

*"Python's type system is like a traffic light in Grand Theft Auto: technically there, but nobody's stopping."* - Anonymous developer at 3 AM