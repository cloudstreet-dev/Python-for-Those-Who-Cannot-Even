# Introduction: Why We're All Here Against Our Will

*"Python is an experiment in how much freedom programmers need. Too much freedom and nobody can read another's code; too little and expressiveness is endangered."* - Guido van Rossum

Yeah, well, Guido, you know what else endangers expressiveness? Making whitespace syntactically significant. But here we are.

## The Great Indentation Disaster of 1991

In 1991, while the rest of us were enjoying music with actual instruments and phones with actual buttons, Guido van Rossum unleashed Python upon an unsuspecting world. His great innovation? What if we took all the things that make code readable—like braces, semicolons, and explicit block delimiters—and replaced them with... nothing? Just spaces. Or tabs. But not both. Never both.

The legend goes that Guido was inspired by ABC, a teaching language that also used indentation for structure. This is like being inspired by a tricycle to build a car. Sure, they both have wheels, but one of them is what adults use.

## Why Python Won (Spoiler: It's Not Because It's Good)

Let's be honest about why we're all here:

1. **Machine Learning Libraries**: Python has NumPy, TensorFlow, and PyTorch. What does Ruby have? Happiness and readable code. Guess which one pays better.

2. **"It's Easy to Learn"**: People say Python is easy to learn. These are the same people who think pineapple belongs on pizza. Sure, Python *looks* simple:
   ```python
   print("Hello, World!")
   ```
   But so does juggling. Until you try it with chainsaws.

3. **Everyone Else Is Using It**: The same reason we all got Facebook accounts in 2008. Peer pressure is real, even in programming.

4. **Google Uses It**: And if it's good enough for a company that kills half its products every year, it's good enough for us!

## What You're In For

This book will teach you Python. Not because Python is good, but because your mortgage isn't going to pay itself. We'll cover:

- **Syntax that will make you miss JavaScript** (and that's saying something)
- **Data structures that are almost as good as Ruby's** (but not quite)
- **Object-oriented programming that feels like Java's awkward cousin**
- **Functional programming features bolted on like a spoiler on a minivan**
- **Async programming that makes you wish for callback hell**

## Setting Up Your Therapy Session (Development Environment)

Before we begin this journey of suffering together, let's set up Python. You have several options, all of them wrong in different ways:

### Option 1: System Python (Don't)

Your system probably came with Python. This is like finding a half-eaten sandwich in your car. Sure, it's food, but do you really want to trust it?

```bash
python --version
# Python 2.7.16
```

Oh good, Python 2. The Python that Python doesn't even want you to use anymore.

### Option 2: Python.org (The Official Disappointment)

Go to python.org and download Python 3. Watch as it installs itself in seventeen different places on your system, none of which are in your PATH.

### Option 3: Homebrew/apt/yum (The Lesser Evil)

```bash
# macOS (assuming you hate yourself enough to use macOS)
brew install python3

# Ubuntu/Debian (for people who compile their own kernels for fun)
sudo apt-get install python3

# RHEL/CentOS (for enterprise suffering)
sudo yum install python3
```

### Option 4: Anaconda (For Data Scientists Who've Given Up)

Download a 3GB installer that includes Python and every library you'll never use. It's like buying a Swiss Army knife that's mostly spoons.

### Option 5: pyenv (The Least Worst Option)

```bash
# Install pyenv (the version manager nobody asked for)
curl https://pyenv.run | bash

# Install a Python version (pick your poison)
pyenv install 3.11.0
pyenv global 3.11.0
```

At least with pyenv you can switch between different versions of disappointment.

## Your First Python Program (Already Wrong)

Let's write our first Python program. Create a file called `misery.py`:

```python
#!/usr/bin/env python3

def main():
    name = input("What's your name? ")
    print(f"Hi {name}, welcome to Python.")
    print("I'm sorry.")
    
if __name__ == "__main__":
    main()
```

Look at that `if __name__ == "__main__":` nonsense. In Ruby, you'd just write your code. In Python, you need this magical incantation to make your script work as both a module and a script. Why? Because Python.

Run it:
```bash
python3 misery.py
```

Congratulations! You've written your first Python program. It's all downhill from here.

## The Python Philosophy (Or: The Zen of Python)

Type this in your Python interpreter:
```python
import this
```

You'll get "The Zen of Python," a collection of aphorisms that Python developers pretend to follow:

> Beautiful is better than ugly.

Unless you're talking about lambda syntax.

> Explicit is better than implicit.

Except for self, type coercion, and that whole "everything is truthy or falsy" thing.

> Simple is better than complex.

*Looks at Python's method resolution order (MRO) for multiple inheritance.* Sure, Jan.

> Complex is better than complicated.

This is just the previous one backwards. They're padding the list.

> Flat is better than nested.

Says the language that requires indentation for everything.

> Sparse is better than dense.

Yet list comprehensions exist.

> Readability counts.

`[x for x in range(10) if x % 2 == 0]` - So readable!

> Special cases aren't special enough to break the rules.

*Stares at `else` clause on loops.*

> Although practicality beats purity.

Translation: "We couldn't figure out how to make it consistent, so here we are."

> Errors should never pass silently.

```python
try:
    something_dangerous()
except:
    pass  # Look, I'm passing silently!
```

> Unless explicitly silenced.

Oh, okay then.

> In the face of ambiguity, refuse the temptation to guess.

Python 2 vs Python 3 string handling has entered the chat.

> There should be one-- and preferably only one --obvious way to do it.

String formatting alone has four different ways. FOUR.

> Although that way may not be obvious at first unless you're Dutch.

This is an actual line in the Zen of Python. I'm not making this up.

> Now is better than never.

Python 3 was released in 2008. Python 2 was sunset in 2020. "Now" is relative.

> Although never is often better than *right* now.

Make up your mind!

> If the implementation is hard to explain, it's a bad idea.

*Gestures at the Global Interpreter Lock (GIL).*

> If the implementation is easy to explain, it may be a good idea.

"May be." The confidence is overwhelming.

> Namespaces are one honking great idea -- let's do more of those!

This one's actually true. Credit where credit's due.

## What Makes Python Special (And By Special, I Mean Frustrating)

### 1. Significant Whitespace

In most languages, indentation is for humans. In Python, it's for the interpreter. Miss a space? Syntax error. Mix tabs and spaces? Believe it or not, straight to jail.

```python
if True:
    print("This is fine")
   print("This is not fine")  # IndentationError
```

### 2. Dynamic Typing (Or: Types Are For Quitters)

```python
x = 5
x = "now I'm a string"
x = ["now I'm a list"]
x = None  # Now I'm nothing, like my will to live
```

This is "flexibility." In Ruby, we call it "duck typing" and at least we're honest about the quacking.

### 3. The Global Interpreter Lock (GIL)

Python threads are like having multiple cashiers at a store but only one register. They can all work, just not at the same time. Efficient!

### 4. Everything Is An Object (But Not In A Good Way)

```python
(5).__add__(3)  # This works. Because reasons.
```

Even integers have methods. It's like finding out your toaster has opinions.

## A Note on Python 2 vs Python 3

Python 2 is dead. It ceased to be. It's expired and gone to meet its maker. It's a stiff. Bereft of life, it rests in peace. If you're still using Python 2 in production:

1. Why?
2. No, seriously, why?
3. Are you okay? Do you need help?

Python 3 has been around since 2008. If your codebase is still on Python 2, it can legally drive in most states.

## How This Book Works

Each chapter will:

1. **Introduce a Python concept** with maximum reluctance
2. **Show you the syntax** while lamenting the lack of curly braces
3. **Compare it to Ruby** (where it will lose)
4. **Provide working examples** (because we're professionals, damn it)
5. **Include exercises** to practice your suffering

## Exercises: Try Not to Cry

1. **Install Python 3** on your system using any method. Document how many different Python installations you now have.

2. **Write a program** that prints "I miss semicolons" 10 times. Notice how you can't end lines with semicolons without Python judging you.

3. **Try to mix tabs and spaces** in a Python file. Experience the error message. This is your life now.

4. **Open a Python interpreter** and type:
   ```python
   import antigravity
   ```
   At least Guido has a sense of humor.

5. **Create two Python files**:
   - One that uses 2-space indentation
   - One that uses 4-space indentation
   - Try to copy code between them
   - Cry

## What's Next?

In Chapter 1, we'll dive into variables and types, where you'll learn that Python's type system is less of a system and more of a suggestion. We'll explore how Python pretends types don't matter until they suddenly do, usually in production at 3 AM.

Remember: We're all in this together. Nobody's happy about it, but at least we're getting paid.

Welcome to Python. I'm sorry.

---

*"Python is the second-best language for everything."* - Someone who's never used Ruby

*"There should be one obvious way to do it. Unless you're formatting strings. Then there should be four."* - The Real Zen of Python