# Chapter 10: Generators and Iterators (Lazy Evaluation for Lazy Programmers)

*"Generators are Python's way of procrastinating: why calculate everything now when you can put it off until someone actually needs it?"* - A developer who just discovered yield

Generators and iterators are Python's approach to lazy evaluation. They're like promises to do work later, which is perfect for programmers who are already experts at procrastination.

## Yield: Not Just for Traffic

The `yield` keyword turns a function into a generator. It's like `return`, but lazier.

```python
# Regular function (eager)
def get_numbers_eager():
    result = []
    for i in range(1000000):
        result.append(i * 2)
    return result  # Creates entire list in memory

# Generator function (lazy)
def get_numbers_lazy():
    for i in range(1000000):
        yield i * 2  # Produces values one at a time

# Using them
eager = get_numbers_eager()  # Uses ~40MB of memory immediately
lazy = get_numbers_lazy()     # Uses ~120 bytes, returns generator object

print(type(eager))  # <class 'list'>
print(type(lazy))   # <class 'generator'>

# Generators are iterators
for num in lazy:
    print(num)
    if num > 10:
        break  # Only calculated 6 values, not 1 million!

# The magic: yield pauses the function
def counting():
    print("Starting to count")
    yield 1
    print("After 1")
    yield 2
    print("After 2")
    yield 3
    print("Done counting")

gen = counting()
print(next(gen))  # "Starting to count" then 1
print(next(gen))  # "After 1" then 2
print(next(gen))  # "After 2" then 3
# print(next(gen))  # "Done counting" then StopIteration

# Generators remember their state between calls
# It's like the function is frozen in time
```

## The Iterator Protocol: Making Things Iterable

```python
# The iterator protocol requires __iter__ and __next__
class CountDown:
    def __init__(self, start):
        self.start = start
    
    def __iter__(self):
        return self  # Returns itself as the iterator
    
    def __next__(self):
        if self.start <= 0:
            raise StopIteration  # Signal we're done
        self.start -= 1
        return self.start + 1

# Using it
for i in CountDown(5):
    print(i)  # 5, 4, 3, 2, 1

# Manual iteration
counter = CountDown(3)
iterator = iter(counter)  # Calls __iter__
print(next(iterator))  # 3, calls __next__
print(next(iterator))  # 2
print(next(iterator))  # 1
# print(next(iterator))  # StopIteration!

# The separation of concerns that nobody asked for
class BetterCountDown:
    """Iterable that returns a fresh iterator each time"""
    def __init__(self, start):
        self.start = start
    
    def __iter__(self):
        return CountDownIterator(self.start)

class CountDownIterator:
    """The actual iterator"""
    def __init__(self, start):
        self.current = start
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        self.current -= 1
        return self.current + 1

# Now you can iterate multiple times
countdown = BetterCountDown(3)
print(list(countdown))  # [3, 2, 1]
print(list(countdown))  # [3, 2, 1] (fresh iterator!)

# Why separate iterable and iterator?
# So you can iterate multiple times
# Most people just use generators instead
```

## Generator Expressions: List Comprehensions' Lazy Cousin

```python
# List comprehension (eager)
squares_list = [x**2 for x in range(1000000)]  # Uses memory immediately

# Generator expression (lazy)
squares_gen = (x**2 for x in range(1000000))  # Just parentheses instead of brackets!

print(sys.getsizeof(squares_list))  # ~40MB
print(sys.getsizeof(squares_gen))   # ~120 bytes

# They look the same when iterating
for square in squares_gen:
    if square > 100:
        break  # Only calculated until we hit 100

# Generator expressions are perfect for chaining
numbers = range(1000000)
squared = (x**2 for x in numbers)
evens = (x for x in squared if x % 2 == 0)
strings = (str(x) for x in evens)

# Nothing calculated yet!
# Only when we iterate:
for s in strings:
    print(s)
    if int(s) > 100:
        break

# Common pattern: generator expression in function calls
total = sum(x**2 for x in range(100))  # No extra parentheses needed!
any_big = any(x > 1000 for x in numbers)  # Stops at first True
all_positive = all(x > 0 for x in numbers)  # Stops at first False
```

## Advanced Generator Features (Because Simple Wasn't Enough)

```python
# Sending values INTO a generator (wat)
def echo_generator():
    while True:
        value = yield  # yield can receive values!
        print(f"Received: {value}")

gen = echo_generator()
next(gen)  # Prime the generator (gets to first yield)
gen.send("Hello")  # Received: Hello
gen.send("World")  # Received: World

# Yield from (delegating to another generator)
def generator1():
    yield 1
    yield 2

def generator2():
    yield 3
    yield 4

def combined():
    yield from generator1()  # Delegates to generator1
    yield from generator2()  # Then to generator2

list(combined())  # [1, 2, 3, 4]

# Without yield from (the old way)
def combined_old():
    for value in generator1():
        yield value
    for value in generator2():
        yield value

# yield from also handles send() and exceptions
def accumulator():
    total = 0
    while True:
        value = yield total
        if value is not None:
            total += value

def wrapper():
    yield from accumulator()  # Transparent delegation

# Generator return values (Python 3.3+)
def generator_with_return():
    yield 1
    yield 2
    return "I'm done!"  # Generators can return values!

gen = generator_with_return()
print(next(gen))  # 1
print(next(gen))  # 2
try:
    next(gen)
except StopIteration as e:
    print(e.value)  # "I'm done!"

# The return value is in StopIteration.value
# Because Python
```

## Infinite Generators (Memory Efficient Infinity)

```python
# Infinite sequences
def fibonacci():
    a, b = 0, 1
    while True:  # Infinite loop!
        yield a
        a, b = b, a + b

# Use it carefully
fib = fibonacci()
for i, num in enumerate(fib):
    print(num)
    if i > 10:  # Must break manually
        break

# itertools has infinite generators
import itertools

# Count forever
for i in itertools.count(10, 2):  # Start at 10, step by 2
    print(i)
    if i > 20:
        break

# Cycle forever
for item in itertools.cycle(['A', 'B', 'C']):
    print(item)  # A, B, C, A, B, C, ...
    # Break or it runs forever!

# Repeat forever
for item in itertools.repeat('Hello', 3):  # Repeat 3 times
    print(item)

# Infinite generator with state
def stateful_generator():
    state = {"count": 0}
    while True:
        state["count"] += 1
        decision = yield state["count"]
        if decision == "reset":
            state["count"] = 0

gen = stateful_generator()
print(next(gen))  # 1
print(next(gen))  # 2
print(gen.send("reset"))  # 1 (reset!)
```

## The Dark Art of Generator Pipelines

```python
# Chaining generators for data processing
def read_lines(filename):
    """Generate lines from a file"""
    with open(filename) as f:
        for line in f:
            yield line.strip()

def filter_comments(lines):
    """Filter out comment lines"""
    for line in lines:
        if not line.startswith('#'):
            yield line

def parse_numbers(lines):
    """Parse lines into numbers"""
    for line in lines:
        try:
            yield float(line)
        except ValueError:
            pass  # Skip non-numbers

def running_average(numbers):
    """Calculate running average"""
    total = 0
    count = 0
    for number in numbers:
        total += number
        count += 1
        yield total / count

# Pipeline!
lines = read_lines('data.txt')
no_comments = filter_comments(lines)
numbers = parse_numbers(no_comments)
averages = running_average(numbers)

# Nothing happens until we iterate!
for avg in averages:
    print(f"Running average: {avg}")

# Memory efficient: only one line in memory at a time
# Even for huge files!

# Generator pipeline with itertools
import itertools

def process_data(data):
    # Take first 1000 items
    data = itertools.islice(data, 1000)
    # Group by a key
    data = itertools.groupby(data, key=lambda x: x % 10)
    # Chain groups
    data = itertools.chain.from_iterable(g for k, g in data)
    # Filter
    data = filter(lambda x: x > 5, data)
    # Map
    data = map(lambda x: x ** 2, data)
    return data

# All lazy! No computation until iteration
```

## Context Managers with Generators (contextlib Magic)

```python
from contextlib import contextmanager

# Traditional context manager (class-based)
class FileManager:
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode
        self.file = None
    
    def __enter__(self):
        self.file = open(self.filename, self.mode)
        return self.file
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.file.close()

# Generator-based context manager (so much simpler!)
@contextmanager
def file_manager(filename, mode):
    f = open(filename, mode)
    try:
        yield f  # This is where 'with' block runs
    finally:
        f.close()

# Usage is identical
with file_manager('test.txt', 'w') as f:
    f.write('Hello!')

# More complex example
@contextmanager
def timed_operation(name):
    import time
    print(f"Starting {name}")
    start = time.time()
    try:
        yield  # No value needed
    finally:
        end = time.time()
        print(f"{name} took {end - start:.2f} seconds")

with timed_operation("Processing"):
    time.sleep(1)
    # Do stuff
# Automatically prints timing!

# Exception handling
@contextmanager
def error_handler():
    try:
        yield
    except Exception as e:
        print(f"Caught error: {e}")
        # Suppress exception by not re-raising

with error_handler():
    raise ValueError("Oops!")
# Error caught and suppressed!
```

## Generator Gotchas and Pitfalls

```python
# Generators are single-use
gen = (x for x in range(5))
list1 = list(gen)  # [0, 1, 2, 3, 4]
list2 = list(gen)  # [] - Empty! Generator exhausted

# Can't pickle generators
import pickle
gen = (x for x in range(5))
# pickle.dumps(gen)  # TypeError! Can't pickle generator objects

# Can't get length
gen = (x for x in range(5))
# len(gen)  # TypeError! Generator has no len()

# Can't index
gen = (x for x in range(5))
# gen[2]  # TypeError! Generator is not subscriptable

# Late binding strikes again
funcs = []
for i in range(5):
    funcs.append(lambda: (yield i))  # SyntaxError! Can't yield in lambda

# Generator expressions in class scope are weird
class Weird:
    items = [1, 2, 3]
    # This fails!
    # gen = (x for x in items)  # NameError: items not found
    # Generator expressions can't access class variables

# Memory leaks with generator references
def leaky():
    huge_list = [i for i in range(1000000)]
    for item in huge_list:
        yield item
    # huge_list stays in memory as long as generator exists!

# StopIteration can be confusing
def my_generator():
    yield 1
    raise StopIteration("Custom message")  # Don't do this!
    yield 2  # Never reached

# In Python 3.7+, this is actually a RuntimeError
# StopIteration shouldn't be raised manually in generators
```

## Real-World Generator Examples

```python
# Reading large files
def read_large_file(file_path, chunk_size=1024):
    """Read a file in chunks"""
    with open(file_path, 'rb') as file:
        while True:
            chunk = file.read(chunk_size)
            if not chunk:
                break
            yield chunk

# Process multi-GB file with minimal memory
for chunk in read_large_file('huge_file.dat'):
    process(chunk)

# Pagination
def paginate(items, page_size):
    """Split items into pages"""
    for i in range(0, len(items), page_size):
        yield items[i:i + page_size]

# Database query results
def fetch_all_users():
    """Fetch users in batches"""
    offset = 0
    batch_size = 100
    while True:
        users = db.query(f"SELECT * FROM users LIMIT {batch_size} OFFSET {offset}")
        if not users:
            break
        yield from users
        offset += batch_size

# Backpressure handling
def rate_limited_generator(source, max_per_second):
    """Limit rate of iteration"""
    import time
    interval = 1.0 / max_per_second
    for item in source:
        start = time.time()
        yield item
        elapsed = time.time() - start
        if elapsed < interval:
            time.sleep(interval - elapsed)

# Use it to avoid overwhelming APIs
for item in rate_limited_generator(data_source, max_per_second=10):
    api.process(item)
```

## Exercises: Try Not to Cry

1. **Infinite Prime Generator**: Write a generator that yields prime numbers forever. Use it to find the 1000th prime.

2. **Generator Pipeline**: Create a pipeline that:
   - Reads a file line by line
   - Filters lines containing a word
   - Converts to uppercase
   - Takes only first 10 results
   All using generators!

3. **Send and Yield**: Create a generator that maintains a running total. Use send() to add values and yield to get the current total.

4. **Memory Test**: Create a list of 10 million numbers and a generator for the same. Compare memory usage using `sys.getsizeof()`.

5. **Context Manager Generator**: Write a generator-based context manager that measures memory usage of a code block.

6. **Generator Exhaustion**: Create a generator and try to iterate it twice. Handle the exhaustion gracefully.

## Summary: What We've Learned

1. **yield makes generators** (functions that pause and resume)
2. **Generators are lazy** (compute on demand)
3. **Generators save memory** (one item at a time)
4. **Generator expressions exist** (like list comprehensions but lazy)
5. **Generators can receive values** (with send())
6. **yield from delegates** (to other generators)
7. **Generators are single-use** (iterate once and done)
8. **Iterator protocol needs __iter__ and __next__** (but just use generators)
9. **contextmanager decorator** (makes context managers from generators)
10. **Generators are everywhere** (range, file objects, etc.)

Next chapter: Context Managers, where we'll learn about the 'with' statement that nobody understands and why Python needs __enter__ and __exit__ instead of just using try-finally like normal people.

---

*"Generators are like teenagers: they only do work when absolutely necessary and forget everything once they're done."* - Parent of a teenage programmer