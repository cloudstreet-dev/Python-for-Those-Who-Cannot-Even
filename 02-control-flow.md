# Chapter 2: Control Flow (Now With 100% More Whitespace)

*"Significant whitespace is like significant others: sounds important until they start controlling your life."* - A Python developer who misses braces

Remember when you could tell where a block of code ended by looking for a closing brace? Those were simpler times. Now we have Python, where a single misplaced space can turn your if statement into an existential crisis.

## If Statements: The Art of Significant Indentation

In most languages, if statements look like this:
```c
if (condition) {
    // do something
} else {
    // do something else
}
```

Clear. Obvious. You can see where it ends. Python looked at this and said, "What if we made it harder?"

```python
if condition:
    # do something
else:
    # do something else
# Is this still part of the else? No? How can you tell?
# You count spaces. Like a barbarian.
```

### The Basic If (Now With Less Clarity)

```python
age = 18

if age >= 18:
    print("You can vote")
    print("You can also be disappointed in your choices")
print("This is outside the if statement")  # Or is it?

# The number of spaces matters
if age >= 18:
    print("Indented with 4 spaces")
   print("Indented with 3 spaces")  # IndentationError!
```

That's right. One space off and Python throws a tantrum. It's like having a compiler that's also your most pedantic coworker.

### The Elif (Because ElseIf Was Too Many Letters)

```python
score = 85

if score >= 90:
    grade = "A"
elif score >= 80:  # elif, not elseif or else if
    grade = "B"
elif score >= 70:
    grade = "C"
elif score >= 60:
    grade = "D"
else:
    grade = "F"  # For "Fantastic job using Python"

# In Ruby, we'd have elsif (also weird but at least consistent)
# In Python, we saved two whole characters! Efficiency!
```

### The Ternary Operator (Or: How to Make One Line Unreadable)

Most languages: `condition ? true_value : false_value`
Python: "Hold my whitespace"

```python
# Python's ternary (they call it "conditional expression")
status = "adult" if age >= 18 else "minor"

# It reads like Yoda wrote it
# "Adult if age greater than 18 is, else minor"

# You can chain them (but shouldn't)
score = 85
grade = "A" if score >= 90 else "B" if score >= 80 else "C" if score >= 70 else "F"
# This is technically one line. It's also technically terrible.

# Compare to Ruby's beautiful:
# grade = score >= 90 ? "A" : score >= 80 ? "B" : "C"
# Still bad, but at least it flows left to right
```

### The Match Statement (Switch's Awkward Cousin)

After 30 years, Python finally got a switch statement in version 3.10. They called it "match" because calling it "switch" would be admitting other languages were right.

```python
# Python 3.10+ only (released in 2021, fashionably late)
command = "quit"

match command:
    case "start":
        print("Starting...")
    case "stop":
        print("Stopping...")
    case "quit" | "exit":  # Multiple patterns
        print("Goodbye!")
    case _:  # Default case (underscore, because 'default' was too obvious)
        print("Unknown command")

# But wait, it gets weird:
point = (0, 1)

match point:
    case (0, 0):
        print("Origin")
    case (0, y):  # Pattern matching with variable binding
        print(f"On Y-axis at {y}")
    case (x, 0):
        print(f"On X-axis at {x}")
    case (x, y) if x == y:  # Guards (conditional patterns)
        print(f"On diagonal at {x}")
    case (x, y):
        print(f"Point at {x}, {y}")

# It's pattern matching, not just a switch
# It's actually powerful, but arriving 30 years late to the party
```

## Loops: While You Question Your Life Choices

### The While Loop (At Least This Is Normal)

```python
count = 0
while count < 5:
    print(f"Count: {count}")
    count += 1  # No ++ operator because Python hates convenience

# Python doesn't have do-while loops
# The Pythonic way to simulate one:
while True:
    response = input("Continue? (y/n): ")
    if response.lower() == 'n':
        break
# "Pythonic" here means "working around missing features"
```

### The For Loop (It's Actually a Foreach)

Python doesn't have traditional for loops. It has foreach loops that it calls for loops because accuracy is optional.

```python
# What you think a for loop looks like:
# for (int i = 0; i < 10; i++) { }

# What Python gives you:
for i in range(10):
    print(i)

# range() is a function that returns an iterator
# So this is really: "for each i in this sequence"

# Want to count backwards?
for i in range(10, 0, -1):  # start, stop, step
    print(i)
# Note: This prints 10 through 1, not 0
# Because range stops BEFORE the stop value
# Because Python

# Want the index and value?
fruits = ["apple", "banana", "orange"]
for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")

# Want to iterate over two lists?
names = ["Alice", "Bob", "Charlie"]
ages = [25, 30, 35]
for name, age in zip(names, ages):
    print(f"{name} is {age}")

# What if the lists are different lengths?
# zip stops at the shortest. Silent data loss! Fun!
```

### List Comprehensions (One-Liners That Make You Feel Smart)

Python's list comprehensions are like the smart kid in class who answers every question before anyone else can think.

```python
# Traditional way (readable)
squares = []
for i in range(10):
    squares.append(i ** 2)

# List comprehension (Pythonic™)
squares = [i ** 2 for i in range(10)]

# With conditions
evens = [i for i in range(20) if i % 2 == 0]

# Nested loops (this is where it gets fun)
matrix = [[i * j for j in range(3)] for i in range(3)]

# Now let's make it unreadable!
result = [
    x if x % 2 == 0 else x * 2
    for x in range(10)
    if x > 5
]
# This filters for x > 5, then doubles odd numbers
# Good luck figuring that out in six months

# Dictionary comprehension (because why not)
squares_dict = {i: i ** 2 for i in range(10)}

# Set comprehension (getting ridiculous)
unique_lengths = {len(word) for word in ["hi", "bye", "hello"]}

# Generator expressions (list comprehensions' lazy cousin)
squares_gen = (i ** 2 for i in range(10))
# Looks the same but with () instead of []
# It's lazy evaluation. It's also confusing.
```

## The Else Clause on Loops (Because Python)

This is where Python gets weird. Well, weirder.

```python
# for-else: The else runs if the loop completes without break
for item in items:
    if item == target:
        print("Found it!")
        break
else:
    print("Didn't find it")
    # This runs if we never hit break
    # "else" here means "no break" not "otherwise"
    # This makes total sense if you don't think about it

# while-else: Same confusing concept
attempts = 0
while attempts < 3:
    password = input("Password: ")
    if password == "secret":
        print("Access granted")
        break
    attempts += 1
else:
    print("Too many attempts")
    # Runs if while condition becomes false
    # Not if we break out

# Most developers' reaction: "Wait, loops can have else?"
# Yes. Should they? No. Will you use it? Also no.
# Will it appear in someone's code review? Absolutely.
```

## Break, Continue, and Pass (The Three Horsemen of Loop Control)

```python
# break: exits the loop
for i in range(10):
    if i == 5:
        break
    print(i)  # Prints 0-4

# continue: skips to next iteration
for i in range(10):
    if i % 2 == 0:
        continue
    print(i)  # Prints odd numbers only

# pass: does nothing (Python's way of saying "TODO")
for i in range(10):
    if i == 5:
        pass  # Placeholder for future code
    else:
        print(i)

# Why does pass exist?
# Because Python needs something in empty blocks:
if True:
    # This is a syntax error without something here
    pass  # Now it's valid

# In other languages, you'd just have empty braces {}
# In Python, you need to explicitly do nothing
# It's like being forced to say "I'm not talking to you"
```

## The Walrus in the Room (Assignment Expressions in Conditions)

Remember our friend the walrus operator from Chapter 1? It really shines in loops:

```python
# Without walrus (Python < 3.8)
while True:
    line = input("Enter command: ")
    if line == "quit":
        break
    process(line)

# With walrus (Python >= 3.8)
while (line := input("Enter command: ")) != "quit":
    process(line)

# Reading a file until empty line
while chunk := file.read(1024):
    process(chunk)

# It's actually useful here
# But it looks like a emoticon
# Professional code with emoticons. We've peaked.
```

## Exception Handling (Try Not to Fail)

Python's exception handling is like Java's, but with more ways to confuse yourself.

```python
# Basic try-except
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Math is hard")

# Multiple exceptions
try:
    risky_operation()
except (ValueError, TypeError) as e:
    print(f"Error: {e}")
except Exception as e:
    print(f"Unexpected error: {e}")
    # Catching all exceptions is bad practice
    # But you'll do it anyway

# The full monty
try:
    file = open("data.txt")
    data = file.read()
except FileNotFoundError:
    print("File not found")
except PermissionError:
    print("Permission denied")
else:
    # Runs if no exception occurred
    # Because apparently we needed another else
    print("File read successfully")
finally:
    # Always runs
    file.close()

# Oh wait, that's wrong. If open() fails, file doesn't exist
# So file.close() in finally will error
# Python: Making simple things complicated since 1991

# The Pythonic way (context managers)
with open("data.txt") as file:
    data = file.read()
# File automatically closed. Magic!
# We'll explain this in Chapter 11
# (Spoiler: It's not actually magic)
```

### Raising Exceptions (How to Fail Professionally)

```python
# Raising exceptions
def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

# Re-raising exceptions
try:
    dangerous_operation()
except Exception as e:
    print(f"Logging error: {e}")
    raise  # Re-raises the same exception

# Custom exceptions (because built-in ones aren't special enough)
class MySpecialError(Exception):
    """An exception that makes me feel special"""
    pass

raise MySpecialError("I'm different!")

# Exception chaining (Python 3+)
try:
    process_data()
except ValueError as e:
    raise RuntimeError("Data processing failed") from e
    # Links the original exception as the cause
    # It's like exception inception
```

## Assertions (Optimistic Error Handling)

```python
# Assertions: For when you're pretty sure but not really
def calculate_average(numbers):
    assert len(numbers) > 0, "Cannot calculate average of empty list"
    return sum(numbers) / len(numbers)

# Assertions can be disabled with -O flag
# python -O script.py  # Assertions ignored
# So never use them for actual validation
# They're like comments that can crash your program

# This is bad:
def transfer_money(amount):
    assert amount > 0  # Don't do this!
    # User runs with -O, negative transfers happen
    # Bank account goes negative
    # You get fired

# This is good:
def transfer_money(amount):
    if amount <= 0:
        raise ValueError("Amount must be positive")
    # Always validates, job saved
```

## The Iterator Protocol (Making Things Loopable)

Everything in Python is iterable if you squint hard enough.

```python
# Iterating over different types
for char in "string":
    print(char)  # Strings are iterable

for key in {"a": 1, "b": 2}:
    print(key)  # Dicts iterate over keys (not values, because Python)

for item in range(5):
    print(item)  # range is an iterator

# Making your own iterator (masochist mode)
class CountDown:
    def __init__(self, start):
        self.start = start
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.start <= 0:
            raise StopIteration
        self.start -= 1
        return self.start + 1

# Using it
for i in CountDown(5):
    print(i)  # 5, 4, 3, 2, 1

# Or just use a generator (sane mode)
def countdown(start):
    while start > 0:
        yield start
        start -= 1

for i in countdown(5):
    print(i)  # Same result, less code, less pain
```

## Exercises: Try Not to Cry

1. **Indentation Hell**: Write a function with at least 5 levels of nested if statements. Now change your editor from spaces to tabs. Watch the world burn.

2. **The Else Loop Challenge**: Write a for-else loop that actually makes sense. (Hint: It involves searching for something)

3. **List Comprehension Golf**: Convert this to a one-line list comprehension:
   ```python
   result = []
   for i in range(100):
       if i % 3 == 0 or i % 5 == 0:
           result.append(i ** 2)
   ```
   Now make it even less readable.

4. **Match Statement Madness** (Python 3.10+): Write a match statement that pattern matches on a complex data structure. Marvel at how it's both powerful and arrived 30 years late.

5. **Exception Inception**: Write a try-except block inside another try-except block inside a finally block. Question your life choices.

6. **The Walrus Challenge**: Find a legitimate use case for the walrus operator that doesn't make the code harder to read. (This is the hardest exercise)

## Summary: What We've Learned

1. **Indentation is syntax**, and Python will never let you forget it
2. **elif** is else-if but shorter and weirder
3. **Ternary operators** read like Yoda speaks
4. **match statements** exist now (only took 30 years)
5. **for loops** are actually foreach loops in disguise
6. **List comprehensions** make you feel smart until you have to debug them
7. **Loops can have else clauses** for maximum confusion
8. **pass** exists because empty blocks are illegal
9. **Exceptions have else and finally** because why not
10. **Everything is iterable** if you implement enough dunder methods

In the next chapter, we'll explore strings, the one thing Python actually does pretty well. We'll learn about f-strings, the only Python feature that's unambiguously better than its equivalents in other languages. It's all downhill from there.

---

*"Python's control flow is like a GPS that insists you can only turn at 90-degree angles."* - Developer who misses goto statements (just kidding, nobody misses those)