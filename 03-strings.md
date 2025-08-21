# Chapter 3: Strings (At Least These Are Pretty Good)

*"Python's string handling is like finding a designer outfit at a thrift store: unexpectedly good, but you're still shopping at a thrift store."* - A pleasantly surprised developer

Alright, I'll admit it. Python strings are actually... good? I know, I'm as shocked as you are. After two chapters of complaining, we finally found something Python does well. Don't worry, we'll still find things to complain about.

## String Basics (Where Python Doesn't Completely Fail)

```python
# Multiple quote styles (actually convenient)
single = 'Single quotes'
double = "Double quotes"
triple_single = '''Triple single quotes'''
triple_double = """Triple double quotes"""

# They're all the same (unlike JavaScript's template literals)
print(single == double)  # True

# Triple quotes preserve formatting
poem = """
    Roses are red,
    Violets are blue,
    Python has strings,
    That don't completely screw you.
"""

# Raw strings (r means raw, not regex)
path = r"C:\Users\name\Documents"  # Backslashes aren't escaped
regex = r"\d{3}-\d{3}-\d{4}"  # Actually useful

# But you can't end a raw string with a backslash
# invalid = r"path\"  # SyntaxError
# Because even good things in Python have gotchas
```

## F-Strings: Python's One Good Idea

F-strings (formatted string literals) were introduced in Python 3.6, and they're actually fantastic. I hate that I love them.

```python
name = "Python"
version = 3.11


# The old ways (all terrible)
old1 = "Hello, %s %s" % (name, version)  # C-style, gross
old2 = "Hello, {} {}".format(name, version)  # Verbose
old3 = "Hello, {0} {1}".format(name, version)  # Indexed
old4 = "Hello, {n} {v}".format(n=name, v=version)  # Named

# The f-string way (actually good)
good = f"Hello, {name} {version}"

# F-strings can do expressions
print(f"2 + 2 = {2 + 2}")
print(f"Upper: {name.upper()}")
print(f"Length: {len(name)}")

# Formatting specifications
pi = 3.14159
print(f"Pi: {pi:.2f}")  # 3.14
print(f"Binary: {42:b}")  # 101010
print(f"Hex: {255:x}")  # ff
print(f"Padded: {42:05d}")  # 00042
print(f"Percent: {0.123:.1%}")  # 12.3%

# Alignment (actually useful)
for item, price in [("Apple", 0.50), ("Banana", 0.30), ("Cherry", 2.00)]:
    print(f"{item:<10} ${price:>5.2f}")
# Apple      $ 0.50
# Banana     $ 0.30
# Cherry     $ 2.00

# Debug mode (Python 3.8+)
value = 42
print(f"{value=}")  # value=42
print(f"{2 + 2=}")  # 2 + 2=4
# The = adds the expression before the result
# Surprisingly handy for debugging

# Nested f-strings (because we can't have nice things)
width = 10
precision = 2
value = 3.14159
print(f"{value:{width}.{precision}f}")  # '      3.14'
# This works but makes my brain hurt

# Multiline f-strings
message = f"""
    User: {name}
    Version: {version}
    Status: {'Active' if version > 3 else 'Deprecated'}
"""

# The only problem: f-strings are evaluated immediately
# So you can't store them as templates
# template = f"Hello {name}"  # name must exist now
# Can't pass this around and fill in later
```

## String Methods That Almost Make Sense

Python has approximately 47 string methods. Here are the ones you'll actually use:

```python
text = "  Python Programming  "

# The useful ones
text.strip()       # "Python Programming" (removes whitespace)
text.lstrip()      # "Python Programming  "
text.rstrip()      # "  Python Programming"
text.lower()       # "  python programming  "
text.upper()       # "  PYTHON PROGRAMMING  "
text.title()       # "  Python Programming  "
text.capitalize()  # "  python programming  " (only first char)
text.swapcase()    # "  pYTHON pROGRAMMING  " (why does this exist?)

# Searching
"Python" in text   # True
text.startswith(" ")  # True
text.endswith("ing  ")  # True
text.find("Prog")  # 9 (index or -1 if not found)
text.index("Prog") # 9 (raises ValueError if not found)
text.count("o")    # 2

# Replacing
text.replace("Python", "Ruby")  # "  Ruby Programming  "
text.replace(" ", "")  # "PythonProgramming"

# Splitting and joining
words = text.split()  # ["Python", "Programming"]
" ".join(words)  # "Python Programming"
"a,b,c".split(",")  # ["a", "b", "c"]

# The weird ones
"42".isdigit()   # True
"abc".isalpha()  # True
"abc123".isalnum()  # True
"".isspace()     # False (empty string isn't space)
" ".isspace()    # True
"Title Case".istitle()  # True

# The confusing ones
"HELLO".isupper()  # True
"hello".islower()  # True
"Hello".isupper()  # False (mixed case)
"".isupper()       # False (empty has no case)
"123".isupper()    # False (digits have no case)
"ABC123".isupper() # True (ignores non-letters)

# The why-does-this-exist ones
"hello".center(10, "*")  # "**hello***"
"hello".ljust(10, "-")   # "hello-----"
"hello".rjust(10, "=")   # "=====hello"
"42".zfill(5)            # "00042" (zero fill)
"hello\nworld".splitlines()  # ["hello", "world"]
"Hello".encode("rot13")  # Not in Python 3! They removed the fun stuff

# String translation (for when replace isn't confusing enough)
trans_table = str.maketrans("aeiou", "12345")
"hello world".translate(trans_table)  # "h2ll4 w4rld"
# Because sometimes you need to replace vowels with numbers?
```

## String Encoding: Unicode and The Pain It Brings

Python 3 strings are Unicode by default. This is good! Until it isn't.

```python
# Python 3 strings are Unicode
emoji = "😀🐍"  # This just works
print(len(emoji))  # 2 (two characters)

# But wait
emoji = "😀"
print(len(emoji))  # 1
print(len(emoji.encode('utf-8')))  # 4 bytes
print(len(emoji.encode('utf-16')))  # 4 bytes (plus BOM)
print(len(emoji.encode('utf-32')))  # 4 bytes (plus BOM)

# Encoding and decoding
text = "Hello, 世界"
bytes_utf8 = text.encode('utf-8')  # b'Hello, \xe4\xb8\x96\xe7\x95\x8c'
bytes_utf16 = text.encode('utf-16')  # b'\xff\xfeH\x00e\x00l\x00l\x00o\x00,\x00 \x00\x16N\x8c\x95'

# Decoding
bytes_utf8.decode('utf-8')  # Back to "Hello, 世界"
# bytes_utf8.decode('ascii')  # UnicodeDecodeError! Fun!

# The nightmare scenario
text = "café"
print(len(text))  # 4

# But what about normalization?
import unicodedata
text1 = "café"  # é as one character
text2 = "café"  # e + combining accent
print(text1 == text2)  # Might be False!
print(len(text1), len(text2))  # Could be 4, 5

# Normalize to fix
text1_norm = unicodedata.normalize('NFC', text1)
text2_norm = unicodedata.normalize('NFC', text2)
print(text1_norm == text2_norm)  # True

# This is why we can't have nice things
```

## String Formatting: Four Ways to Do the Same Thing

Because "There should be one-- and preferably only one --obvious way to do it" is just a suggestion.

```python
name = "Alice"
age = 30

# Method 1: % formatting (C-style, deprecated but everywhere)
old_style = "Name: %s, Age: %d" % (name, age)
old_style2 = "Name: %(name)s, Age: %(age)d" % {"name": name, "age": age}

# Method 2: str.format() (Python 2.6+, verbose)
format_style = "Name: {}, Age: {}".format(name, age)
format_style2 = "Name: {0}, Age: {1}".format(name, age)
format_style3 = "Name: {n}, Age: {a}".format(n=name, a=age)

# Method 3: f-strings (Python 3.6+, actually good)
f_string = f"Name: {name}, Age: {age}"

# Method 4: Template strings (nobody uses these)
from string import Template
t = Template("Name: $name, Age: $age")
template_style = t.substitute(name=name, age=age)

# They all do the same thing
# f-strings are best
# Everyone still uses % formatting
# This is Python
```

## String Operations That Make You Think

```python
# String multiplication (actually useful)
"=" * 50  # Creates a line of 50 equals signs
"Hello" * 3  # "HelloHelloHello"
"Ha" * 0  # "" (multiplication by zero gives empty string)

# String comparison (alphabetical)
"apple" < "banana"  # True
"Apple" < "apple"  # True (capitals come first)
"10" < "2"  # True (string comparison, not numeric!)

# String containment
"Hello" in "Hello, World!"  # True
"HELLO" in "Hello, World!"  # False (case sensitive)

# String identity vs equality
a = "Hello"
b = "Hello"
a == b  # True (equal values)
a is b  # Probably True (string interning)

# But watch this
a = "Hello World!"
b = "Hello World!"
a is b  # Maybe True, maybe False (depends on Python's mood)

# And this
a = "Hello" + " World!"
b = "Hello World!"
a == b  # True
a is b  # False (constructed differently)

# String interning is weird
import sys
a = sys.intern("Hello World!")
b = sys.intern("Hello World!")
a is b  # True (forced interning)
```

## Regular Expressions (Now Things Get Ugly)

Python's regex is... adequate. It's not Perl, but what is?

```python
import re

# Basic matching
pattern = r"\d{3}-\d{3}-\d{4}"
text = "Call me at 555-123-4567"

if re.search(pattern, text):
    print("Found a phone number!")

# Finding all matches
emails = "Contact: john@example.com and jane@test.org"
pattern = r"\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b"
matches = re.findall(pattern, emails)
# ['john@example.com', 'jane@test.org']

# Groups (capturing parts)
pattern = r"(\d{3})-(\d{3})-(\d{4})"
match = re.search(pattern, "555-123-4567")
if match:
    print(match.group(0))  # "555-123-4567" (full match)
    print(match.group(1))  # "555" (first group)
    print(match.groups())  # ("555", "123", "4567")

# Named groups (slightly more readable)
pattern = r"(?P<area>\d{3})-(?P<exchange>\d{3})-(?P<number>\d{4})"
match = re.search(pattern, "555-123-4567")
if match:
    print(match.group('area'))  # "555"
    print(match.groupdict())  # {'area': '555', 'exchange': '123', 'number': '4567'}

# Substitution
text = "The year 2020 was weird"
new_text = re.sub(r"\d{4}", "2021", text)
# "The year 2021 was weird"

# Substitution with function
def replace_year(match):
    year = int(match.group())
    return str(year + 1)

new_text = re.sub(r"\d{4}", replace_year, text)
# "The year 2021 was weird"

# Splitting
text = "apple,banana;orange|grape"
fruits = re.split(r"[,;|]", text)
# ['apple', 'banana', 'orange', 'grape']

# Compiling (for performance, supposedly)
pattern = re.compile(r"\d+")
# Use pattern.search(), pattern.findall(), etc.
# Marginally faster if you use it multiple times

# Regex flags (because one way isn't enough)
re.search(r"hello", "HELLO", re.IGNORECASE)  # Case insensitive
re.search(r"^hello", "hello\nworld", re.MULTILINE)  # ^ matches line start
re.search(r"hello.world", "hello\nworld", re.DOTALL)  # . matches newline

# The horror: Verbose regex
pattern = re.compile(r"""
    \d{3}  # Area code
    -      # Separator
    \d{3}  # Exchange
    -      # Separator
    \d{4}  # Number
""", re.VERBOSE)
# You can add comments! In your regex! Why would you want this?
```

## String Performance Gotchas

```python
# String concatenation in loops (bad)
result = ""
for i in range(10000):
    result += str(i)  # Creates new string each time
# This is O(n²) because strings are immutable

# Better: Use list and join
parts = []
for i in range(10000):
    parts.append(str(i))
result = "".join(parts)
# This is O(n)

# Or use io.StringIO (even better for large strings)
from io import StringIO
buffer = StringIO()
for i in range(10000):
    buffer.write(str(i))
result = buffer.getvalue()

# String comparison with 'is' (don't)
user_input = input("Enter 'yes': ")
if user_input is "yes":  # BAD! Might work, might not
    print("You said yes")

if user_input == "yes":  # GOOD! Always works
    print("You said yes")
```

## The Weird Parts (Because It's Still Python)

```python
# Strings are sequences
text = "Hello"
print(text[0])  # 'H'
print(text[-1])  # 'o' (negative indexing)
print(text[1:4])  # 'ell' (slicing)
print(text[::-1])  # 'olleH' (reverse)

# But strings are immutable
# text[0] = 'h'  # TypeError!

# String slicing never errors
text = "Hello"
print(text[100:200])  # '' (empty string, not error)
print(text[2:100])  # 'llo' (stops at end)

# Triple quote weirdness
text = """Line 1
Line 2"""
print(repr(text))  # 'Line 1\nLine 2'

text = """\
Line 1
Line 2"""  # Backslash prevents initial newline
print(repr(text))  # 'Line 1\nLine 2'

# String module constants (from another era)
import string
print(string.ascii_letters)  # 'abcdefg...XYZ'
print(string.digits)  # '0123456789'
print(string.punctuation)  # '!"#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~'
print(string.whitespace)  # ' \t\n\r\x0b\x0c'

# These exist because... reasons?
```

## Exercises: Try Not to Cry

1. **F-String Olympics**: Write the most complex f-string you can. Include expressions, formatting, alignment, and nested braces. Make it unreadable.

2. **Encoding Hell**: Create a string with emojis, Chinese characters, and combining diacriticals. Encode it in different formats. Try to get back the original. Cry.

3. **Regex Nightmare**: Write a regex to validate email addresses. Really validate them. According to RFC 5322. (Hint: The full regex is over 6,000 characters)

4. **String Method Chain**: Chain as many string methods as possible in a single line. Example: `text.strip().lower().replace().split().join().title()`

5. **Format Fight**: Take the same data and format it using all four formatting methods (%, format, f-string, Template). Decide which one you hate least.

6. **Performance Test**: Concatenate 100,000 strings using += vs join(). Time the difference. Wonder why Python makes the wrong way so easy.

## Summary: What We've Learned

1. **Python strings are actually good** (I can't believe I'm saying this)
2. **F-strings are fantastic** (Python did something right!)
3. **There are 4 ways to format strings** (because one would be too simple)
4. **Everything is Unicode** (until it breaks)
5. **Regex exists** (and it's exactly as painful as everywhere else)
6. **Strings are immutable** (which is good, actually)
7. **String methods are plentiful** (perhaps too plentiful)
8. **Triple quotes are useful** (despite the edge cases)

This might be the most positive chapter in this book. Don't worry, normal service will resume when we get to lists and tuples, where Python decided that having two nearly identical data structures was a good idea.

---

*"Python strings are like that friend who's surprisingly good at karaoke: you didn't expect much, but you're pleasantly surprised."* - A developer who still misses Ruby's string interpolation