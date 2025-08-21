# Chapter 7: Decorators (The @thing That Confuses Everyone)

*"Decorators are like makeup for functions: they can enhance beauty, hide flaws, or make everything unrecognizable."* - A developer lost in decorator hell

Decorators are Python's way of modifying functions using other functions, wrapped in syntactic sugar that looks like someone's email address got lost in your code. They're powerful, confusing, and everywhere in modern Python.

## What Even Is a Decorator?

A decorator is a function that takes a function and returns a function. If that sounds like a function, that's because it's a function.

```python
# The simplest decorator (that does nothing useful)
def do_nothing_decorator(func):
    return func

@do_nothing_decorator
def greet():
    print("Hello!")

# The @ syntax is sugar for:
# greet = do_nothing_decorator(greet)

# A slightly useful decorator
def shout_decorator(func):
    def wrapper():
        result = func()
        return result.upper() if result else result
    return wrapper

@shout_decorator
def greet():
    return "hello"

print(greet())  # "HELLO"

# What actually happened:
# 1. Python defined greet
# 2. Passed greet to shout_decorator
# 3. shout_decorator returned wrapper
# 4. Python assigned wrapper to the name greet
# 5. Original greet is now trapped inside wrapper
```

## The Basic Decorator Pattern

```python
# The standard decorator template
def my_decorator(func):
    def wrapper(*args, **kwargs):
        # Do something before
        print(f"Calling {func.__name__}")
        
        # Call the original function
        result = func(*args, **kwargs)
        
        # Do something after
        print(f"Done calling {func.__name__}")
        
        return result
    return wrapper

@my_decorator
def add(x, y):
    return x + y

result = add(5, 3)
# Calling add
# Done calling add
# result = 8

# The problem: we lost the original function's metadata
print(add.__name__)  # "wrapper" (not "add"!)
```

## Functools.wraps (Fixing What We Broke)

```python
from functools import wraps

def better_decorator(func):
    @wraps(func)  # Preserves func's metadata
    def wrapper(*args, **kwargs):
        """This is the wrapper"""
        return func(*args, **kwargs)
    return wrapper

@better_decorator
def greet(name):
    """Greet someone"""
    return f"Hello, {name}!"

print(greet.__name__)  # "greet" (correct!)
print(greet.__doc__)   # "Greet someone" (preserved!)

# Without @wraps, we'd get wrapper's metadata
# Always use @wraps. Always.
```

## Decorators With Arguments (Inception Mode)

```python
# Decorator that takes arguments needs another level
def repeat(times):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(times=3)
def greet():
    print("Hello!")

greet()
# Hello!
# Hello!
# Hello!

# What's happening:
# 1. repeat(times=3) returns decorator
# 2. decorator(greet) returns wrapper
# 3. wrapper replaces greet

# Without sugar:
# greet = repeat(times=3)(greet)

# Even more parameters
def advanced_decorator(prefix="", suffix=""):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            print(prefix)
            result = func(*args, **kwargs)
            print(suffix)
            return result
        return wrapper
    return decorator

@advanced_decorator(prefix="Starting...", suffix="Done!")
def process():
    print("Processing")
```

## Class Decorators (Objects Decorating Functions)

```python
# Decorators can be classes too
class CountCalls:
    def __init__(self, func):
        self.func = func
        self.count = 0
        wraps(func)(self)  # Preserve metadata
    
    def __call__(self, *args, **kwargs):
        self.count += 1
        print(f"Call {self.count} to {self.func.__name__}")
        return self.func(*args, **kwargs)

@CountCalls
def greet(name):
    return f"Hello, {name}!"

greet("Alice")  # Call 1 to greet
greet("Bob")    # Call 2 to greet
print(greet.count)  # 2

# Class with parameters
class Retry:
    def __init__(self, max_attempts=3):
        self.max_attempts = max_attempts
    
    def __call__(self, func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(self.max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == self.max_attempts - 1:
                        raise
                    print(f"Attempt {attempt + 1} failed: {e}")
            return None
        return wrapper

@Retry(max_attempts=3)
def unreliable_function():
    import random
    if random.random() < 0.5:
        raise ValueError("Random failure!")
    return "Success!"
```

## Common Decorators That Actually Matter

```python
# @property - Makes methods look like attributes
class Temperature:
    def __init__(self, celsius=0):
        self._celsius = celsius
    
    @property
    def fahrenheit(self):
        return self._celsius * 9/5 + 32
    
    @fahrenheit.setter
    def fahrenheit(self, value):
        self._celsius = (value - 32) * 5/9

temp = Temperature(25)
print(temp.fahrenheit)  # 77.0 (no parentheses!)
temp.fahrenheit = 86
print(temp._celsius)  # 30.0

# @staticmethod - Method that doesn't need self
class MathUtils:
    @staticmethod
    def add(x, y):
        return x + y

print(MathUtils.add(5, 3))  # 8 (no instance needed)

# @classmethod - Method that gets class, not instance
class Person:
    count = 0
    
    def __init__(self, name):
        self.name = name
        Person.count += 1
    
    @classmethod
    def get_count(cls):
        return cls.count

Person("Alice")
Person("Bob")
print(Person.get_count())  # 2

# @lru_cache - Memoization (actually useful!)
from functools import lru_cache

@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

# First call: calculates
print(fibonacci(100))  # Fast!
# Second call: returns cached result
print(fibonacci(100))  # Instant!

# @dataclass - Generates boilerplate (Python 3.7+)
from dataclasses import dataclass

@dataclass
class Point:
    x: float
    y: float
    
    # __init__, __repr__, __eq__ generated automatically!

p = Point(10, 20)
print(p)  # Point(x=10, y=20)
```

## Stacking Decorators (Wrapper Inception)

```python
# Multiple decorators (order matters!)
def bold(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        return f"<b>{result}</b>"
    return wrapper

def italic(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        return f"<i>{result}</i>"
    return wrapper

@bold
@italic
def greet(name):
    return f"Hello, {name}"

print(greet("World"))  # <b><i>Hello, World</i></b>

# Applied bottom to top:
# greet = bold(italic(greet))

# Reverse the order:
@italic
@bold
def greet2(name):
    return f"Hello, {name}"

print(greet2("World"))  # <i><b>Hello, World</b></i>

# With arguments
@lru_cache(maxsize=256)
@repeat(times=2)
@timer
def complex_function(x):
    # This function is:
    # 1. Timed
    # 2. Repeated twice
    # 3. Cached
    return x ** 2
```

## Writing Your Own Decorators (Don't)

```python
# Timer decorator
import time
from functools import wraps

def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        end = time.perf_counter()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)

slow_function()  # slow_function took 1.0012 seconds

# Debug decorator
def debug(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        args_repr = [repr(a) for a in args]
        kwargs_repr = [f"{k}={v!r}" for k, v in kwargs.items()]
        signature = ", ".join(args_repr + kwargs_repr)
        print(f"Calling {func.__name__}({signature})")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result!r}")
        return result
    return wrapper

@debug
def add(x, y):
    return x + y

add(5, 3)
# Calling add(5, 3)
# add returned 8

# Validation decorator
def validate_positive(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        for arg in args:
            if isinstance(arg, (int, float)) and arg < 0:
                raise ValueError(f"Negative value not allowed: {arg}")
        return func(*args, **kwargs)
    return wrapper

@validate_positive
def calculate_area(width, height):
    return width * height

# calculate_area(-5, 10)  # ValueError!
```

## The Dark Side of Decorators

```python
# Decorators can make debugging hell
@cache_result
@retry_on_failure
@log_calls
@validate_inputs
@measure_time
@authenticate_user
@rate_limit
def do_something(data):
    # Good luck figuring out what's actually happening
    return process(data)

# Method decorators with self
class MyClass:
    @property
    def expensive_property(self):
        # This is called every time!
        return sum(range(1000000))
    
    @cached_property  # Python 3.8+
    def better_property(self):
        # This is cached after first access
        return sum(range(1000000))

# Decorators that modify signature
def add_argument(func):
    @wraps(func)
    def wrapper(extra_arg, *args, **kwargs):
        print(f"Extra arg: {extra_arg}")
        return func(*args, **kwargs)
    return wrapper

@add_argument
def greet(name):
    return f"Hello, {name}"

# Now requires extra argument!
greet("ignored", "Alice")  # Confusing!

# Decorators on classes (yes, this exists)
def add_repr(cls):
    def __repr__(self):
        return f"{cls.__name__}({self.__dict__})"
    cls.__repr__ = __repr__
    return cls

@add_repr
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(10, 20)
print(p)  # Point({'x': 10, 'y': 20})
```

## Parametrized Decorators (The Final Boss)

```python
# The ultimate decorator pattern
def parametrized(decorator):
    """Decorator that allows decorators to be used with or without arguments"""
    @wraps(decorator)
    def wrapper(*args, **kwargs):
        if len(args) == 1 and callable(args[0]) and not kwargs:
            # Called without arguments: @decorator
            return decorator()(args[0])
        else:
            # Called with arguments: @decorator(...)
            return decorator(*args, **kwargs)
    return wrapper

@parametrized
def repeat(func=None, times=2):
    def decorator(f):
        @wraps(f)
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = f(*args, **kwargs)
            return result
        return wrapper
    
    if func is None:
        return decorator
    else:
        return decorator(func)

# Now works both ways!
@repeat
def greet1():
    print("Hello!")

@repeat(times=3)
def greet2():
    print("Hi!")

# This is too clever. Please don't do this.
```

## Exercises: Try Not to Cry

1. **Decorator That Decorates**: Write a decorator that counts how many times a function is called and stores it in a class variable.

2. **Order Matters**: Create three decorators that modify a string. Apply them in different orders and predict the output.

3. **Cache With Expiry**: Write an @cache decorator that expires after N seconds.

4. **Method Decorator**: Create a decorator that only works on methods and accesses self.

5. **Decorator Factory**: Write a decorator that takes a list of validators and validates all arguments against them.

6. **Stack Trace**: Create a function with 5 stacked decorators. Cause an error and try to understand the stack trace.

## Summary: What We've Learned

1. **Decorators are functions that modify functions** (or classes)
2. **@ syntax is sugar** for func = decorator(func)
3. **Always use @wraps** to preserve metadata
4. **Decorators with arguments** need another level of nesting
5. **Multiple decorators** apply bottom to top
6. **Common decorators** include @property, @staticmethod, @lru_cache
7. **Classes can be decorators** using __call__
8. **Debugging decorated functions** is painful
9. **Decorators can stack** infinitely (but shouldn't)
10. **Just because you can** doesn't mean you should

Next: Classes, where we'll learn why everything needs self, what dunder methods are, and why Python's OOP feels like Java's awkward cousin who studied philosophy.

---

*"Decorators are like recursion: powerful, elegant, and a great way to confuse yourself six months later."* - Every Python developer's git blame