# Chapter 4: Lists and Tuples (Arrays With Identity Crisis)

*"Python has lists and tuples because sometimes you want to change your data, and sometimes you want your data to gaslight you into thinking it can't be changed."* - A confused developer

Why does Python have both lists and tuples? It's like having both a pen and a pencil, except the pencil can't be erased and sometimes Python uses it to store your function arguments. This is what passes for design decisions in Python.

## Lists: Mutable Arrays That Wish They Were Ruby Arrays

Lists are Python's arrays. They're mutable, dynamic, and can hold anything. They're almost as good as Ruby arrays, which is like saying your car is almost as good as a Ferrari.

```python
# Creating lists (so many ways to be wrong)
empty = []
also_empty = list()
numbers = [1, 2, 3, 4, 5]
mixed = [1, "two", 3.0, None, True]  # Type safety is for quitters
nested = [[1, 2], [3, 4], [5, 6]]  # Lists all the way down

# List from other iterables
from_string = list("hello")  # ['h', 'e', 'l', 'l', 'o']
from_range = list(range(5))  # [0, 1, 2, 3, 4]
from_dict = list({"a": 1, "b": 2})  # ['a', 'b'] (keys only, because Python)

# List repetition (because why not)
zeros = [0] * 10  # [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
# But watch out:
danger = [[]] * 3  # Looks like [[], [], []]
danger[0].append(1)
print(danger)  # [[1], [1], [1]] - They're all the same list!
# This is everyone's favorite Python gotcha #2
```

### List Operations (The Good, The Bad, and The Pythonic)

```python
fruits = ["apple", "banana", "cherry"]

# Accessing elements
first = fruits[0]  # "apple"
last = fruits[-1]  # "cherry" (negative indexing is actually nice)
# middle = fruits[10]  # IndexError! At least it tells you

# Slicing (Python's best feature that nobody understands)
fruits[1:3]  # ["banana", "cherry"] (end is exclusive)
fruits[:2]   # ["apple", "banana"] (from start)
fruits[1:]   # ["banana", "cherry"] (to end)
fruits[:]    # Full copy (shallow)
fruits[::2]  # ["apple", "cherry"] (every 2nd element)
fruits[::-1] # ["cherry", "banana", "apple"] (reverse)

# The slicing syntax: list[start:stop:step]
# If you can remember what order these go in, you're better than me

# Modifying lists
fruits.append("date")  # Adds to end
fruits.insert(1, "apricot")  # Insert at index
fruits.extend(["elderberry", "fig"])  # Add multiple
fruits += ["grape"]  # Same as extend but looks like addition

# Removing items (pick your poison)
fruits.remove("banana")  # Remove first occurrence by value
popped = fruits.pop()  # Remove and return last item
popped = fruits.pop(0)  # Remove and return item at index
del fruits[0]  # Remove by index (no return value)
del fruits[1:3]  # Remove slice
fruits.clear()  # Remove everything

# Why do we need 5 ways to remove things?
# Because Python.
```

### List Methods (There Are So Many)

```python
numbers = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5]

# Searching
numbers.index(5)  # 4 (first occurrence)
# numbers.index(10)  # ValueError if not found
numbers.count(5)  # 3 (frequency)
1 in numbers  # True (membership test)

# Sorting (mutates the list)
numbers.sort()  # Sorts in place
print(numbers)  # [1, 1, 2, 3, 3, 4, 5, 5, 5, 6, 9]

# Want a new sorted list instead?
original = [3, 1, 4, 1, 5]
sorted_copy = sorted(original)  # Returns new list
print(original)  # Still [3, 1, 4, 1, 5]

# Reverse sorting
numbers.sort(reverse=True)
# Or
numbers.reverse()  # Just reverses, doesn't sort

# Custom sorting
words = ["apple", "pie", "Zoo", "banana"]
words.sort()  # ['Zoo', 'apple', 'banana', 'pie'] (capitals first!)
words.sort(key=str.lower)  # ['apple', 'banana', 'pie', 'Zoo']
words.sort(key=len)  # ['pie', 'Zoo', 'apple', 'banana']

# Complex sorting
students = [("Alice", 85), ("Bob", 75), ("Charlie", 95)]
students.sort(key=lambda x: x[1])  # Sort by grade
# [('Bob', 75), ('Alice', 85), ('Charlie', 95)]
```

### List Comprehensions (Again, Because They're Everywhere)

```python
# Basic comprehension
squares = [x**2 for x in range(10)]

# With condition
evens = [x for x in range(20) if x % 2 == 0]

# Multiple conditions
special = [x for x in range(100) if x % 2 == 0 if x % 3 == 0]
# Or
special = [x for x in range(100) if x % 2 == 0 and x % 3 == 0]

# Nested comprehensions (please don't)
matrix = [[i*j for j in range(3)] for i in range(3)]

# Flattening (this is actually useful)
nested = [[1, 2], [3, 4], [5, 6]]
flat = [item for sublist in nested for item in sublist]
# [1, 2, 3, 4, 5, 6]

# With conditions on nested
flat_evens = [item for sublist in nested for item in sublist if item % 2 == 0]

# This is getting unreadable
# "Pythonic" doesn't mean "good"
```

## Tuples: Immutable Lists That Nobody Asked For

Tuples are like lists, but immutable. They're Python's way of saying "you can look but you can't touch."

```python
# Creating tuples
empty = ()
also_empty = tuple()
single = (1,)  # Note the comma! (1) is just integer 1 in parentheses
numbers = (1, 2, 3, 4, 5)
mixed = (1, "two", 3.0, None)

# Parentheses are often optional (but please use them)
also_tuple = 1, 2, 3  # This is a tuple
x, y = 10, 20  # Tuple unpacking

# From other iterables
from_list = tuple([1, 2, 3])
from_string = tuple("hello")  # ('h', 'e', 'l', 'l', 'o')
```

### Why Tuples Exist (Nobody Really Knows)

```python
# Theory 1: Performance
import sys
list_version = [1, 2, 3, 4, 5]
tuple_version = (1, 2, 3, 4, 5)
print(sys.getsizeof(list_version))  # 88 bytes
print(sys.getsizeof(tuple_version))  # 80 bytes
# Wow, 8 whole bytes saved! Totally worth the confusion!

# Theory 2: Hashability
# Lists can't be dictionary keys
# dict_with_list = {[1, 2]: "value"}  # TypeError!

# Tuples can (if they contain only immutable items)
dict_with_tuple = {(1, 2): "value"}  # This works

# But wait
# dict_with_bad_tuple = {(1, [2]): "value"}  # TypeError!
# Tuples with mutable items aren't hashable either

# Theory 3: Semantic meaning
# Lists are for homogeneous data (same type)
# Tuples are for heterogeneous data (different types)
coordinates = (10.5, 20.3)  # x, y - different meaning
scores = [95, 87, 92, 88]  # All scores - same meaning
# But Python doesn't enforce this at all
# So it's just a suggestion that nobody follows
```

### Tuple Operations (Like Lists, But Sadder)

```python
numbers = (1, 2, 3, 4, 5)

# Reading works the same
first = numbers[0]
last = numbers[-1]
slice = numbers[1:3]

# But no modification
# numbers[0] = 10  # TypeError!
# numbers.append(6)  # AttributeError!

# The "immutable" lie
sneaky = ([1, 2], [3, 4])
sneaky[0].append(3)  # This works!
print(sneaky)  # ([1, 2, 3], [3, 4])
# The tuple is immutable, but its contents might not be
# This is fine and not confusing at all

# Tuple methods (both of them)
numbers.count(3)  # 1
numbers.index(3)  # 2
# That's it. That's all the methods.
```

### Tuple Unpacking (The One Good Thing)

```python
# Basic unpacking
point = (10, 20)
x, y = point  # x=10, y=20

# Multiple assignment
a, b = 1, 2  # Actually tuple unpacking

# Swapping variables
a, b = b, a  # The Pythonic way

# Extended unpacking (Python 3+)
first, *middle, last = [1, 2, 3, 4, 5]
# first=1, middle=[2, 3, 4], last=5

# Ignoring values
x, _, z = (1, 2, 3)  # _ is convention for "don't care"

# Function returns
def get_coordinates():
    return 10, 20  # Returns tuple

x, y = get_coordinates()

# Nested unpacking
data = ((1, 2), (3, 4))
(a, b), (c, d) = data

# Star expressions
first, *rest = [1, 2, 3, 4]  # first=1, rest=[2, 3, 4]
*start, last = [1, 2, 3, 4]  # start=[1, 2, 3], last=4
first, *middle, last = [1]  # first=1, middle=[], last=1??? No, ValueError!
```

## Named Tuples (Tuples With Aspirations)

```python
from collections import namedtuple

# Creating a named tuple class
Point = namedtuple('Point', ['x', 'y'])
# Or
Point = namedtuple('Point', 'x y')  # Space-separated string

# Using it
p = Point(10, 20)
print(p.x)  # 10 (attribute access!)
print(p[0])  # 10 (still works like a tuple)

# They're still immutable
# p.x = 30  # AttributeError!

# But you can create new ones
p2 = p._replace(x=30)  # Point(x=30, y=20)

# Other methods
print(p._fields)  # ('x', 'y')
print(p._asdict())  # {'x': 10, 'y': 20}

# Why not just use a class?
# Because... uh... performance?
```

## Lists vs Tuples: The Eternal Confusion

```python
# When to use lists:
shopping = ["milk", "eggs", "bread"]  # Will add/remove items

# When to use tuples:
coordinates = (10.5, 20.3)  # Won't change
rgb = (255, 128, 0)  # Color values

# What people actually do:
everything_is_lists = ["because", "why", "bother"]

# Function arguments are tuples
def func(*args):
    print(type(args))  # <class 'tuple'>

func(1, 2, 3)

# But keyword arguments are dicts
def func2(**kwargs):
    print(type(kwargs))  # <class 'dict'>

func2(a=1, b=2)

# String formatting wants tuples
"Hello %s %s" % ("Python", "World")  # Tuple
# "Hello %s %s" % ["Python", "World"]  # TypeError!

# Unless you use format, then it wants arguments
"Hello {} {}".format("Python", "World")  # Not a tuple
```

## Advanced List Slicing (Because Simple Wasn't Enough)

```python
numbers = list(range(10))  # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# Assignment to slices
numbers[2:5] = [20, 30, 40]  # [0, 1, 20, 30, 40, 5, 6, 7, 8, 9]

# Different size replacement
numbers[2:5] = [100]  # [0, 1, 100, 5, 6, 7, 8, 9]

# Insert via slice
numbers[2:2] = [99, 98]  # Insert without removing

# Delete via slice
numbers[2:5] = []  # Deletes elements 2-4

# Step slicing
numbers[::2] = [10, 20, 30, 40, 50]  # Replace every 2nd element
# But the replacement must be the same length!

# Copy confusion
list1 = [1, 2, [3, 4]]
list2 = list1[:]  # Shallow copy
list2[0] = 10  # Doesn't affect list1
list2[2].append(5)  # DOES affect list1!

# Deep copy (when you really mean it)
import copy
list3 = copy.deepcopy(list1)
list3[2].append(6)  # Doesn't affect list1
```

## Performance Gotchas

```python
# List concatenation in loops (bad)
result = []
for i in range(10000):
    result = result + [i]  # Creates new list each time!

# Better: append
result = []
for i in range(10000):
    result.append(i)

# Even better: list comprehension
result = [i for i in range(10000)]

# Or just
result = list(range(10000))

# Checking membership
# Lists: O(n)
1000000 in list(range(1000001))  # Slow

# Sets: O(1)
1000000 in set(range(1000001))  # Fast

# But don't convert just for one check
# The conversion itself is O(n)
```

## Exercises: Try Not to Cry

1. **The Aliasing Trap**: Create a list of lists using `[[0] * 3] * 3`. Try to set `matrix[0][0] = 1`. Observe the chaos. Fix it.

2. **Tuple Mutability Madness**: Create a tuple containing a list. Modify the list. Explain to a rubber duck why this is "immutable."

3. **Slice Assignment Golf**: Start with `list(range(10))`. Using only slice assignment (no methods), transform it to `[0, 99, 2, 4, 6, 8]`.

4. **Unpacking Challenge**: Given `data = [1, [2, 3, [4, 5]], 6]`, unpack it in one line to get `a=1, b=2, c=3, d=[4,5], e=6`.

5. **Performance Test**: Time these operations on a list of 1 million numbers:
   - Finding the maximum using `max()`
   - Finding the maximum using a loop
   - Finding if 999999 is in the list
   - Converting to set and checking if 999999 is in it

6. **Named Tuple Nightmare**: Create a named tuple for a Person with 10 fields. Try to remember the order. Give up and use a dictionary.

## Summary: What We've Learned

1. **Lists are mutable arrays** that can hold anything
2. **Tuples are immutable lists** that exist for reasons
3. **Both use the same indexing and slicing** because consistency
4. **Lists have many methods, tuples have two** because mutability
5. **Slicing is powerful** but the syntax is forgettable
6. **List comprehensions are everywhere** whether you like it or not
7. **Tuple unpacking is actually useful** (finally, something good)
8. **Named tuples exist** when you want a struct but get a tuple
9. **Shallow vs deep copying** will bite you eventually
10. **Use lists for everything** unless you literally can't

Next up: dictionaries and sets, where Python almost gets hash tables right, except for that time they changed how dictionaries work in 3.7 and just didn't tell anyone.

---

*"Lists and tuples are like twins where one got all the functionality and the other got... hashability, I guess?"* - Developer trying to justify tuples' existence