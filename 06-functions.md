# Chapter 6: Functions (def Doesn't Stand for Definitely)

*"Python functions are like a box of chocolates: you never know what you're gonna get, especially with default arguments."* - Forrest Gump, if he programmed

Functions in Python are first-class objects, which sounds impressive until you realize it just means you can pass them around like any other variable. They're also where Python's scoping rules reveal themselves to be the product of someone who really enjoyed making developers suffer.

## Basic Functions (Where Hope Goes to Die)

```python
# The simplest function
def do_nothing():
    pass  # Because Python needs something in the body

# With a return value
def add(a, b):
    return a + b

# Without return (returns None)
def print_something(text):
    print(text)
    # Implicitly returns None

result = print_something("Hello")
print(result)  # None

# Multiple return values (actually a tuple)
def get_coordinates():
    return 10, 20  # Returns (10, 20)

x, y = get_coordinates()  # Tuple unpacking

# Docstrings (documentation that nobody reads)
def complex_function(param1, param2):
    """
    This function does something complex.
    
    Args:
        param1: First parameter
        param2: Second parameter
    
    Returns:
        Something, probably
    """
    return param1 + param2

print(complex_function.__doc__)  # Prints the docstring
```

## Default Arguments: The Mutable Default Trap

This is Python's favorite gotcha. Default arguments are evaluated once at function definition time, not at call time.

```python
# The trap everyone falls into
def append_to_list(item, target=[]):
    target.append(item)
    return target

list1 = append_to_list(1)
list2 = append_to_list(2)
print(list1)  # [1, 2] - Wait, what?
print(list2)  # [1, 2] - They're the same list!

# The default [] is created ONCE when the function is defined
# Every call without target uses THE SAME LIST

# The fix
def append_to_list_fixed(item, target=None):
    if target is None:
        target = []
    target.append(item)
    return target

# Why is it like this?
# Because Guido wanted functions to be objects
# And objects are created once
# And default arguments are part of the function object
# Makes perfect sense if you don't think about it

# It gets worse
def problem(when=datetime.now()):  # Evaluated at definition!
    print(f"Called at {when}")

problem()  # Shows definition time
time.sleep(2)
problem()  # Still shows definition time!

# The fix (again)
def fixed(when=None):
    if when is None:
        when = datetime.now()
    print(f"Called at {when}")
```

## *args and **kwargs: When You've Given Up on Type Safety

```python
# *args: Variable positional arguments (tuple)
def print_all(*args):
    print(f"Type: {type(args)}")  # <class 'tuple'>
    for arg in args:
        print(arg)

print_all(1, 2, 3, "hello")

# **kwargs: Variable keyword arguments (dict)
def print_kwargs(**kwargs):
    print(f"Type: {type(kwargs)}")  # <class 'dict'>
    for key, value in kwargs.items():
        print(f"{key} = {value}")

print_kwargs(name="Alice", age=30)

# Both together (with rules)
def kitchen_sink(required, *args, keyword=None, **kwargs):
    print(f"Required: {required}")
    print(f"Args: {args}")
    print(f"Keyword: {keyword}")
    print(f"Kwargs: {kwargs}")

kitchen_sink(1, 2, 3, keyword="hi", extra="data", more="stuff")

# The order matters:
# 1. Positional parameters
# 2. *args
# 3. Keyword-only parameters
# 4. **kwargs

# Keyword-only arguments (Python 3+)
def keyword_only(*, name, age):  # * forces keyword-only
    print(f"{name} is {age}")

# keyword_only("Alice", 30)  # TypeError!
keyword_only(name="Alice", age=30)  # Works

# Positional-only arguments (Python 3.8+)
def positional_only(name, age, /):  # / forces positional-only
    print(f"{name} is {age}")

positional_only("Alice", 30)  # Works
# positional_only(name="Alice", age=30)  # TypeError!

# Both restrictions (because why not)
def weird(pos_only, /, normal, *, kw_only):
    pass
```

## Unpacking Arguments (The Splat Operators)

```python
# Unpacking with * and **
def add(a, b, c):
    return a + b + c

numbers = [1, 2, 3]
result = add(*numbers)  # Unpacks list to arguments

data = {"a": 1, "b": 2, "c": 3}
result = add(**data)  # Unpacks dict to keyword arguments

# Combining everything (chaos mode)
def chaos(a, b, *args, c=10, **kwargs):
    print(f"a={a}, b={b}, args={args}, c={c}, kwargs={kwargs}")

values = [1, 2, 3, 4]
options = {"c": 20, "d": 30, "e": 40}
chaos(*values, **options)
# a=1, b=2, args=(3, 4), c=20, kwargs={'d': 30, 'e': 40}

# Extended unpacking in calls
first, *rest = [1, 2, 3, 4]
print(first, rest)  # 1 [2, 3, 4]

# In function definitions
def merge(*lists):
    result = []
    for lst in lists:
        result.extend(lst)
    return result

merged = merge([1, 2], [3, 4], [5, 6])
```

## Lambda Functions: Anonymous Functions Nobody Can Read

Lambdas are Python's anonymous functions, limited to single expressions because multi-line lambdas would be too useful.

```python
# Basic lambda
square = lambda x: x ** 2
print(square(5))  # 25

# Why not just use def?
def square(x):
    return x ** 2
# Because sometimes you want to be mysterious

# Common use: sorting
students = [("Alice", 85), ("Bob", 92), ("Charlie", 78)]
students.sort(key=lambda x: x[1])  # Sort by grade

# In map/filter/reduce (functional programming cosplay)
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x**2, numbers))
evens = list(filter(lambda x: x % 2 == 0, numbers))

from functools import reduce
total = reduce(lambda x, y: x + y, numbers)

# Lambda limitations
# Can't have statements
# invalid = lambda x: print(x)  # Actually works in Python 3
# But can't have multiple statements
# invalid = lambda x: x += 1; return x  # SyntaxError

# Immediately invoked lambda (IIFE for Python)
result = (lambda x: x * 2)(5)  # 10
# Why would you do this? Nobody knows.

# Lambdas in comprehensions (maximum unreadability)
funcs = [lambda x, i=i: x + i for i in range(5)]
# The i=i is necessary because of late binding
# Without it, all lambdas would use i=4
```

## Scope and The LEGB Rule

Python's scope resolution follows LEGB: Local, Enclosing, Global, Built-in. It's simple until it isn't.

```python
# Global scope
global_var = "I'm global"

def outer():
    # Enclosing scope
    enclosing_var = "I'm enclosing"
    
    def inner():
        # Local scope
        local_var = "I'm local"
        
        # LEGB resolution
        print(local_var)      # Local
        print(enclosing_var)  # Enclosing
        print(global_var)     # Global
        print(len)            # Built-in
    
    inner()

# The confusion starts
x = "global"

def confusing():
    print(x)  # UnboundLocalError!
    x = "local"

# Python sees x is assigned in the function
# So it's local for the ENTIRE function
# Even before the assignment

# The global keyword (admission of defeat)
def modify_global():
    global x
    x = "modified"

# The nonlocal keyword (for nested functions)
def outer():
    x = "outer"
    
    def inner():
        nonlocal x
        x = "modified outer"
    
    inner()
    print(x)  # "modified outer"

# Without nonlocal, inner would create its own local x
```

## Closures (Functions That Remember Things)

```python
# Basic closure
def make_multiplier(n):
    def multiplier(x):
        return x * n  # n is captured from enclosing scope
    return multiplier

times_two = make_multiplier(2)
times_three = make_multiplier(3)

print(times_two(5))    # 10
print(times_three(5))  # 15

# Closures capture variables, not values
def make_functions():
    funcs = []
    for i in range(5):
        funcs.append(lambda x: x + i)
    return funcs

functions = make_functions()
print(functions[0](10))  # 14 (not 10!)
print(functions[1](10))  # 14 (not 11!)
# All functions use i=4 (last value)

# Fix with default argument
def make_functions_fixed():
    funcs = []
    for i in range(5):
        funcs.append(lambda x, i=i: x + i)  # Capture current i
    return funcs

# Or use a closure properly
def make_functions_proper():
    def make_func(i):
        return lambda x: x + i
    return [make_func(i) for i in range(5)]
```

## Functions Are Objects (Everything Is An Object)

```python
# Functions have attributes
def my_function():
    """I'm a docstring"""
    pass

print(my_function.__name__)  # "my_function"
print(my_function.__doc__)   # "I'm a docstring"

# You can add attributes (but why?)
my_function.custom = "Why is this allowed?"
print(my_function.custom)

# Functions can be passed around
def apply_twice(func, arg):
    return func(func(arg))

def add_three(x):
    return x + 3

result = apply_twice(add_three, 10)  # 16

# Functions in data structures
operations = {
    "add": lambda x, y: x + y,
    "subtract": lambda x, y: x - y,
    "multiply": lambda x, y: x * y,
}

result = operations["add"](5, 3)  # 8

# Function annotations (type hints that do nothing)
def annotated(x: int, y: int) -> int:
    return x + y

print(annotated.__annotations__)
# {'x': <class 'int'>, 'y': <class 'int'>, 'return': <class 'int'>}

# But you can still do this:
result = annotated("hello", "world")  # "helloworld"
# Annotations are just decoration
```

## Recursion (Stack Overflow Waiting to Happen)

```python
# Basic recursion
def factorial(n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)

# Python has a recursion limit
import sys
print(sys.getrecursionlimit())  # Usually 1000

# You can change it (but shouldn't)
# sys.setrecursionlimit(10000)

# Tail recursion isn't optimized
def factorial_tail(n, acc=1):
    if n <= 1:
        return acc
    return factorial_tail(n - 1, n * acc)
# Still uses O(n) stack space
# Python doesn't do tail call optimization
# Because Guido doesn't like it

# Mutual recursion (for fun)
def is_even(n):
    if n == 0:
        return True
    return is_odd(n - 1)

def is_odd(n):
    if n == 0:
        return False
    return is_even(n - 1)

# This works but is terrible for performance
```

## The Weird Parts

```python
# Functions can have the same name
def func():
    return 1

def func():  # Overwrites the first one
    return 2

print(func())  # 2

# Late binding strikes again
funcs = []
for i in range(3):
    def func():
        return i
    funcs.append(func)

for f in funcs:
    print(f())  # 2, 2, 2 (all return 2!)

# Functions inside conditions (yes, this works)
if True:
    def conditional_func():
        return "I exist"
else:
    def conditional_func():
        return "I don't"

print(conditional_func())  # "I exist"

# Modifying function internals (don't)
def normal():
    return 42

# Functions have a __code__ attribute
import types
normal.__code__ = types.CodeType(
    0, 0, 0, 0, 0, b'd\x00S\x00', (100,), 
    (), (), "", "", 1, b''
)
# I've just broken the function
# Please don't do this
```

## Exercises: Try Not to Cry

1. **Mutable Default Madness**: Create a function with a mutable default that actually makes sense to have. (Hint: caching)

2. **Scope Puzzle**: Predict the output without running:
   ```python
   x = 1
   def f():
       x = 2
       def g():
           x = 3
           def h():
               nonlocal x
               x = 4
           h()
           return x
       return g()
   print(f(), x)
   ```

3. **Lambda Labyrinth**: Write a single lambda that takes two arguments and returns a lambda that takes one argument and adds all three together.

4. **Closure Chaos**: Create a list of 10 functions where function i returns i*x. Make sure they actually work correctly.

5. **Args and Kwargs Kata**: Write a function that accepts any arguments and prints them in a formatted way, handling both positional and keyword arguments.

6. **Recursion Limit**: Write a recursive function that finds the actual recursion limit on your system by catching the RecursionError.

## Summary: What We've Learned

1. **Default arguments are evaluated once** (gotcha #1)
2. **Functions are objects** (with attributes and everything)
3. ***args and **kwargs** handle variable arguments (goodbye, type safety)
4. **Lambda functions exist** (but are limited to expressions)
5. **LEGB scope resolution** (Local, Enclosing, Global, Built-in)
6. **Closures capture variables** (not values, gotcha #2)
7. **global and nonlocal** keywords exist (because scope is hard)
8. **Functions can be passed around** (first-class objects)
9. **Recursion has a limit** (usually 1000)
10. **Late binding** will hurt you (gotcha #3)

Next chapter: Decorators, where we'll learn how to wrap functions in other functions in other functions until nobody knows what's happening anymore.

---

*"Python functions are like opinions: everyone has them, they're often wrong, and default arguments make them dangerous."* - Wisdom from production debugging