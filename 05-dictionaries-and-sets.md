# Chapter 5: Dictionaries and Sets (Hash Tables in Disguise)

*"Python dictionaries are like that friend who got their life together after college: they used to be unpredictable, but now they're ordered and responsible. We miss the chaos."* - Developer reminiscing about Python 3.6

Dictionaries are Python's hash tables, and they're actually pretty good. Sets are like dictionaries that forgot their values. Both are built on hash tables, which means they're fast for lookups but will occasionally surprise you with unhashable type errors at the worst possible moment.

## Dictionaries: At Least These Are Useful

Dictionaries map keys to values. In other languages, they're called maps, hashes, associative arrays, or "that thing that's better than a list for lookups."

```python
# Creating dictionaries (pick your favorite syntax)
empty = {}
also_empty = dict()

# Literal syntax
person = {"name": "Alice", "age": 30, "city": "NYC"}

# Constructor with keyword arguments
person = dict(name="Alice", age=30, city="NYC")

# From sequences of pairs
pairs = [("name", "Alice"), ("age", 30)]
person = dict(pairs)

# From two lists (zip magic)
keys = ["name", "age", "city"]
values = ["Alice", 30, "NYC"]
person = dict(zip(keys, values))

# Dictionary comprehension (because of course)
squares = {x: x**2 for x in range(10)}

# From keys with same value
keys = ["a", "b", "c"]
same_value = dict.fromkeys(keys, 0)  # {"a": 0, "b": 0, "c": 0}

# Warning: mutable default value
danger = dict.fromkeys(keys, [])  # All keys share THE SAME list
danger["a"].append(1)
print(danger)  # {"a": [1], "b": [1], "c": [1]}
# Python's favorite gotcha strikes again!
```

### Dictionary Operations (The Reasonable Ones)

```python
person = {"name": "Alice", "age": 30}

# Accessing values
name = person["name"]  # "Alice"
# missing = person["height"]  # KeyError!

# Safe access with get()
height = person.get("height")  # None
height = person.get("height", 0)  # 0 (default value)

# Adding/updating
person["email"] = "alice@example.com"  # Add
person["age"] = 31  # Update

# Checking existence
if "name" in person:  # Key existence
    print("Has name")

# Note: 'in' checks keys, not values
if "Alice" in person:  # False! "Alice" is not a key
    print("This won't print")

# Removing items
del person["age"]  # Remove by key
# del person["height"]  # KeyError if missing

# Safe removal
age = person.pop("age", None)  # Returns value or default
item = person.popitem()  # Removes and returns arbitrary item (LIFO since 3.7)

# Clearing
person.clear()  # Now it's {}
```

### Dictionary Methods (Too Many, As Usual)

```python
person = {"name": "Alice", "age": 30}
other = {"age": 31, "city": "NYC"}

# Getting all keys, values, items
person.keys()    # dict_keys(['name', 'age']) - not a list!
person.values()  # dict_values(['Alice', 30])
person.items()   # dict_items([('name', 'Alice'), ('age', 30)])

# These are views, not lists
keys = person.keys()
person["email"] = "alice@example.com"
print(keys)  # dict_keys(['name', 'age', 'email']) - updated!

# Convert to list if needed
list(person.keys())  # ['name', 'age']

# Updating dictionaries
person.update(other)  # Merges other into person
person.update(age=32, city="Boston")  # Keyword arguments
person.update([("age", 33)])  # Sequence of pairs

# Python 3.9+ merge operators (finally!)
merged = person | other  # New dictionary
person |= other  # Update in place

# setdefault (the confusing one)
# Gets value if key exists, otherwise sets and returns default
email = person.setdefault("email", "unknown@example.com")
# If email exists: returns existing value
# If email doesn't exist: sets it to default and returns default

# Why not just use:
# email = person.get("email", "unknown@example.com")
# Because get() doesn't modify the dictionary
# setdefault() does. Because Python needs both.
```

### The Ordering Surprise (Python 3.7+)

```python
# Before Python 3.7: Dictionaries were unordered
# Python 3.6: Dictionaries happened to be ordered (implementation detail)
# Python 3.7+: Dictionaries are officially ordered!

d = {}
d["first"] = 1
d["second"] = 2
d["third"] = 3

# Guaranteed to iterate in insertion order
for key in d:
    print(key)  # first, second, third

# This was a big deal
# Broke a lot of assumptions
# Made OrderedDict mostly obsolete

from collections import OrderedDict
# OrderedDict still exists for:
# 1. Backward compatibility
# 2. Additional methods like move_to_end()
# 3. Different equality comparison (order matters)

od1 = OrderedDict([("a", 1), ("b", 2)])
od2 = OrderedDict([("b", 2), ("a", 1)])
print(od1 == od2)  # False (order matters)

d1 = {"a": 1, "b": 2}
d2 = {"b": 2, "a": 1}
print(d1 == d2)  # True (order doesn't matter for equality)
```

### Dictionary Comprehensions (Because Why Not)

```python
# Basic comprehension
squares = {x: x**2 for x in range(10)}

# With condition
even_squares = {x: x**2 for x in range(10) if x % 2 == 0}

# Transforming dictionaries
person = {"name": "alice", "city": "nyc"}
upper = {k: v.upper() for k, v in person.items()}

# Swapping keys and values
inverted = {v: k for k, v in person.items()}
# Careful: duplicate values will overwrite!

# Nested dictionaries (here be dragons)
matrix = {
    i: {j: i*j for j in range(3)}
    for i in range(3)
}
# {0: {0: 0, 1: 0, 2: 0}, 1: {0: 0, 1: 1, 2: 2}, ...}

# Filtering
scores = {"Alice": 85, "Bob": 92, "Charlie": 78}
passed = {name: score for name, score in scores.items() if score >= 80}
```

## Sets: Dictionaries That Lost Their Values

Sets are unordered collections of unique elements. They're like dictionaries that forgot their values and only remember keys.

```python
# Creating sets
empty = set()  # {} creates empty dict, not set!
numbers = {1, 2, 3, 4, 5}
from_list = set([1, 2, 2, 3, 3, 3])  # {1, 2, 3}

# Set comprehension
squares = {x**2 for x in range(10)}

# From string
letters = set("hello")  # {'h', 'e', 'l', 'o'} - no duplicates!

# Mixed types (but must be hashable)
mixed = {1, "two", 3.0, (4, 5)}
# invalid = {1, [2, 3]}  # TypeError! Lists aren't hashable

# Frozen sets (immutable sets)
frozen = frozenset([1, 2, 3])
# frozen.add(4)  # AttributeError!
# But frozen sets can be in sets!
set_of_sets = {frozenset([1, 2]), frozenset([3, 4])}
```

### Set Operations (Math Class Flashbacks)

```python
a = {1, 2, 3, 4, 5}
b = {4, 5, 6, 7, 8}

# Union (all elements from both)
a | b  # {1, 2, 3, 4, 5, 6, 7, 8}
a.union(b)  # Same thing

# Intersection (common elements)
a & b  # {4, 5}
a.intersection(b)  # Same thing

# Difference (in a but not in b)
a - b  # {1, 2, 3}
a.difference(b)  # Same thing

# Symmetric difference (in either but not both)
a ^ b  # {1, 2, 3, 6, 7, 8}
a.symmetric_difference(b)  # Same thing

# Subset and superset
{1, 2} <= {1, 2, 3}  # True (subset)
{1, 2, 3} >= {1, 2}  # True (superset)
{1, 2} < {1, 2, 3}  # True (proper subset)

# Disjoint (no common elements)
{1, 2}.isdisjoint({3, 4})  # True
{1, 2}.isdisjoint({2, 3})  # False
```

### Set Methods (Mutating and Non-Mutating)

```python
s = {1, 2, 3}

# Adding elements
s.add(4)  # {1, 2, 3, 4}
s.add(3)  # Still {1, 2, 3, 4} - no duplicates

# Updating (adding multiple)
s.update([5, 6])  # {1, 2, 3, 4, 5, 6}
s.update({7, 8}, {9, 10})  # Multiple iterables

# Removing elements
s.remove(5)  # Removes 5
# s.remove(100)  # KeyError if not present!

s.discard(6)  # Removes 6
s.discard(100)  # No error if not present

item = s.pop()  # Removes and returns arbitrary element
# empty_set.pop()  # KeyError if empty

# Set operations that modify in place
a = {1, 2, 3}
b = {2, 3, 4}

a |= b  # Union update
a &= b  # Intersection update
a -= b  # Difference update
a ^= b  # Symmetric difference update

# Why both operators and methods?
# Operators are cleaner for simple cases
# Methods can take multiple arguments
a.union(b, c, d)  # Union of multiple sets
```

## Common Use Cases (Where These Actually Shine)

### Dictionaries for Mapping

```python
# Counting occurrences
words = ["apple", "banana", "apple", "cherry", "banana", "apple"]
count = {}
for word in words:
    count[word] = count.get(word, 0) + 1
# {'apple': 3, 'banana': 2, 'cherry': 1}

# Or use Counter (cheating)
from collections import Counter
count = Counter(words)

# Grouping data
students = [
    {"name": "Alice", "grade": "A"},
    {"name": "Bob", "grade": "B"},
    {"name": "Charlie", "grade": "A"},
]
by_grade = {}
for student in students:
    grade = student["grade"]
    if grade not in by_grade:
        by_grade[grade] = []
    by_grade[grade].append(student["name"])
# {'A': ['Alice', 'Charlie'], 'B': ['Bob']}

# Or use defaultdict (more cheating)
from collections import defaultdict
by_grade = defaultdict(list)
for student in students:
    by_grade[student["grade"]].append(student["name"])

# Caching/Memoization
cache = {}
def expensive_function(n):
    if n in cache:
        return cache[n]
    result = n ** 2  # Pretend this is expensive
    cache[n] = result
    return result
```

### Sets for Uniqueness and Membership

```python
# Removing duplicates
numbers = [1, 2, 2, 3, 3, 3, 4, 4, 4, 4]
unique = list(set(numbers))  # [1, 2, 3, 4]

# Finding common elements
list1 = [1, 2, 3, 4, 5]
list2 = [4, 5, 6, 7, 8]
common = list(set(list1) & set(list2))  # [4, 5]

# Fast membership testing
valid_users = {"alice", "bob", "charlie"}
if username in valid_users:  # O(1) lookup
    print("Valid user")

# Set operations on words
python_keywords = {"if", "else", "for", "while", "def", "class"}
my_variables = {"data", "result", "if", "class"}
conflicts = my_variables & python_keywords  # {"if", "class"}
```

## Advanced Dictionary Features

### Dictionary Views and Their Weirdness

```python
d = {"a": 1, "b": 2, "c": 3}

# Views are dynamic
keys = d.keys()
values = d.values()
items = d.items()

d["d"] = 4
print(keys)  # dict_keys(['a', 'b', 'c', 'd']) - updated!

# Views support set operations (keys and items only)
d1 = {"a": 1, "b": 2}
d2 = {"b": 2, "c": 3}

d1.keys() & d2.keys()  # {'b'} - common keys
d1.keys() - d2.keys()  # {'a'} - keys in d1 but not d2

# items() views work if values are hashable
d1.items() & d2.items()  # {('b', 2)} - common items

# But this fails if values aren't hashable
d3 = {"a": [1, 2]}
# d3.items() & d1.items()  # TypeError!
```

### ChainMap (Dictionaries All the Way Down)

```python
from collections import ChainMap

# Combining dictionaries without merging
defaults = {"color": "red", "size": "medium"}
user_prefs = {"color": "blue"}
final = ChainMap(user_prefs, defaults)

print(final["color"])  # "blue" (from user_prefs)
print(final["size"])   # "medium" (from defaults)

# Changes affect the first dictionary
final["color"] = "green"
print(user_prefs)  # {"color": "green"} - modified!
print(defaults)    # {"color": "red"} - unchanged

# Use case: Configuration with fallbacks
import os
cli_args = {"port": 8080}
env_vars = {"host": os.getenv("HOST", "localhost")}
defaults = {"host": "127.0.0.1", "port": 3000, "debug": False}
config = ChainMap(cli_args, env_vars, defaults)
```

## Performance Considerations

```python
# Dictionary/Set lookups: O(1) average, O(n) worst case
# List lookups: O(n) always

# When to use sets vs lists
large_list = list(range(1000000))
large_set = set(range(1000000))

# Membership testing
999999 in large_list  # Slow (O(n))
999999 in large_set   # Fast (O(1))

# But creation has overhead
small_list = [1, 2, 3]
small_set = {1, 2, 3}
# For small collections, lists might be faster overall

# Dictionary key requirements
# Keys must be hashable (immutable)
valid_dict = {
    42: "number",
    "string": "text",
    (1, 2): "tuple",
    frozenset([3, 4]): "frozen set",
}

# These won't work as keys
# invalid = {[1, 2]: "list"}  # TypeError
# invalid = {{1, 2}: "set"}   # TypeError
# invalid = {{"a": 1}: "dict"}  # TypeError
```

## The Weird Parts

```python
# Boolean keys
d = {True: "yes", 1: "one", 1.0: "float"}
print(d)  # {True: 'float'}
# True == 1 == 1.0 in Python, same hash!
# Last value wins

# Dictionary update order
d = {"a": 1}
d.update({"b": 2}, a=3)
print(d)  # {"a": 3, "b": 2}
# Keyword arguments are processed last!

# Set operations with different types
{1, 2, 3} == {1.0, 2.0, 3.0}  # True!
{1, 2} | [3, 4]  # TypeError - can't mix sets and lists
{1, 2}.union([3, 4])  # {1, 2, 3, 4} - method works!

# Empty set syntax sadness
empty_dict = {}  # This is a dict
empty_set = set()  # This is a set
# {} is always a dict because Python

# The __missing__ magic method
class DefaultDict(dict):
    def __missing__(self, key):
        return "default"

dd = DefaultDict({"a": 1})
print(dd["a"])  # 1
print(dd["b"])  # "default"
```

## Exercises: Try Not to Cry

1. **The Mutable Default Disaster**: Create a function that uses a dictionary as a default parameter. Call it multiple times. Watch the shared state chaos.

2. **Set Operation Golf**: Given two lists with duplicates, find:
   - Elements in both (intersection)
   - Elements in either (union)
   - Elements in first but not second (difference)
   Do it in one line each using sets.

3. **Dictionary Merge Madness**: You have three dictionaries. Merge them where later values override earlier ones. Do it three ways:
   - Using update()
   - Using {**d1, **d2, **d3}
   - Using | operator (Python 3.9+)

4. **View Confusion**: Create a dictionary, get its keys() view, modify the dictionary, then try to convert the view to a list. Explain why the list changes.

5. **Performance Challenge**: Create a list and set with 1 million numbers. Time:
   - Checking if 999999 is present
   - Checking if -1 is present
   - Adding an element
   - Removing an element

6. **The Hash Collision Special**: Create a dictionary where the keys are True, 1, and 1.0. Predict what happens. Now add False, 0, and 0.0.

## Summary: What We've Learned

1. **Dictionaries are ordered now** (since Python 3.7, breaking decades of assumptions)
2. **Sets are just dictionaries without values** (literally the implementation)
3. **Both require hashable keys/elements** (immutable, basically)
4. **Dictionary comprehensions exist** (because everything needs a comprehension)
5. **Set operations are actually useful** (math was good for something)
6. **Views are dynamic** (and support some set operations)
7. **get() and setdefault()** are different (confusingly so)
8. **{} is a dict, not a set** (Python's little joke)
9. **ChainMap exists** (for when you need dictionary inheritance)
10. **O(1) lookup is nice** (when you remember to use it)

Next chapter: Functions, where we'll learn about Python's bizarre scoping rules, why default arguments are evaluated at definition time (still not over this), and what the hell *args and **kwargs actually mean.

---

*"Python dictionaries are like that responsible friend who finally got their act together in version 3.7. Sets are their weird cousin who only shows up to math conventions."* - Developer who remembers the dark times