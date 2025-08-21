# Chapter 8: Classes (Where Self Goes to Die)

*"Python's OOP is like a vegetarian bacon: it looks like the real thing, tastes somewhat similar, but deep down everyone knows something's not quite right."* - A Java developer trying Python

Python has classes. They're like classes in other languages, except you have to pass `self` everywhere, there's no real privacy, and everything is held together by naming conventions and good intentions.

## The Eternal Mystery of `self`

```python
class Dog:
    def __init__(self, name):
        self.name = name  # self everywhere
    
    def bark(self):  # self again
        print(f"{self.name} says Woof!")  # self.name
    
    def play_with(self, other_dog):  # self even with other args
        print(f"{self.name} plays with {other_dog.name}")

# Why self?
# Because Python passes the instance explicitly
# dog.bark() is really Dog.bark(dog)

fido = Dog("Fido")
fido.bark()  # Fido says Woof!

# You can call it the ugly way
Dog.bark(fido)  # Same thing!

# You don't HAVE to call it self (but you're a monster if you don't)
class Cat:
    def __init__(this_thing_here, name):  # Works but WHY
        this_thing_here.name = name
    
    def meow(whatever):
        print(f"{whatever.name} says Meow!")

# Please don't do this
```

## `__init__` and Other Dunder Methods

Dunder (double underscore) methods are Python's magic methods. They're called "magic" because nobody understands why there are so many.

```python
class Person:
    def __init__(self, name, age):
        """Constructor (not really, __new__ is, but whatever)"""
        self.name = name
        self.age = age
    
    def __str__(self):
        """String representation for humans"""
        return f"{self.name} ({self.age})"
    
    def __repr__(self):
        """String representation for developers"""
        return f"Person(name='{self.name}', age={self.age})"
    
    def __eq__(self, other):
        """Equality comparison"""
        if not isinstance(other, Person):
            return False
        return self.name == other.name and self.age == other.age
    
    def __lt__(self, other):
        """Less than comparison"""
        return self.age < other.age
    
    def __len__(self):
        """Why would a person have length? Because Python"""
        return self.age
    
    def __bool__(self):
        """Truthiness"""
        return self.age >= 18  # Adults are truthy?
    
    def __call__(self):
        """Make instance callable like a function"""
        print(f"Why are you calling {self.name}?")

p = Person("Alice", 30)
print(str(p))    # Alice (30)
print(repr(p))   # Person(name='Alice', age=30)
print(len(p))    # 30 (weird but okay)
print(bool(p))   # True
p()              # Why are you calling Alice?

# There are like 50 more dunder methods
# __add__, __sub__, __mul__, __div__  # Math operators
# __getitem__, __setitem__  # Bracket notation
# __enter__, __exit__  # Context managers
# __iter__, __next__  # Iteration
# __getattr__, __setattr__  # Attribute access
# It never ends
```

## Why Everything Needs Double Underscores

```python
class BankAccount:
    def __init__(self, balance):
        self.__balance = balance  # "Private" attribute
    
    def deposit(self, amount):
        self.__balance += amount
    
    def __secret_method(self):  # "Private" method
        return "This is secret"

account = BankAccount(100)
# print(account.__balance)  # AttributeError!
# print(account.__secret_method())  # AttributeError!

# Just kidding, nothing is private
print(account._BankAccount__balance)  # 100
print(account._BankAccount__secret_method())  # "This is secret"

# Python mangles names starting with __ to _ClassName__name
# It's not security, it's a suggestion
# Like a "Please Don't Touch" sign on a button

# Single underscore is "internal use" (also just a suggestion)
class Example:
    def __init__(self):
        self._internal = "Please don't use this"
        self.public = "Use this"

# You can still access _internal
# Python trusts you
# Python shouldn't trust you
```

## Class vs Instance Variables (The Shared State Trap)

```python
class Student:
    school = "Python High"  # Class variable (shared)
    
    def __init__(self, name):
        self.name = name  # Instance variable

# All instances share class variables
alice = Student("Alice")
bob = Student("Bob")

print(alice.school)  # "Python High"
print(bob.school)    # "Python High"

# Change class variable
Student.school = "Python University"
print(alice.school)  # "Python University"
print(bob.school)    # "Python University"

# But watch this mess
alice.school = "Different School"  # Creates instance variable!
print(alice.school)  # "Different School"
print(bob.school)    # "Python University"
print(Student.school)  # "Python University"

# The mutable class variable disaster
class Classroom:
    students = []  # Shared mutable state!
    
    def add_student(self, name):
        self.students.append(name)

room1 = Classroom()
room2 = Classroom()

room1.add_student("Alice")
print(room2.students)  # ['Alice'] - SHARED!

# The fix
class FixedClassroom:
    def __init__(self):
        self.students = []  # Instance variable
    
    def add_student(self, name):
        self.students.append(name)
```

## Properties (Getters and Setters, But Pythonic)

```python
class Temperature:
    def __init__(self, celsius=0):
        self._celsius = celsius
    
    @property
    def celsius(self):
        """Getter"""
        return self._celsius
    
    @celsius.setter
    def celsius(self, value):
        """Setter with validation"""
        if value < -273.15:
            raise ValueError("Below absolute zero!")
        self._celsius = value
    
    @celsius.deleter
    def celsius(self):
        """Deleter (why would you delete temperature?)"""
        print("Deleting temperature")
        del self._celsius
    
    @property
    def fahrenheit(self):
        return self._celsius * 9/5 + 32
    
    @fahrenheit.setter
    def fahrenheit(self, value):
        self.celsius = (value - 32) * 5/9

temp = Temperature()
temp.celsius = 25  # Uses setter
print(temp.celsius)  # Uses getter
print(temp.fahrenheit)  # Computed property

# The old way (don't do this)
class OldWay:
    def get_value(self):
        return self._value
    
    def set_value(self, value):
        self._value = value

# This isn't Java!
```

## Class Methods and Static Methods (The Confused Siblings)

```python
class Pizza:
    def __init__(self, ingredients):
        self.ingredients = ingredients
    
    @classmethod
    def margherita(cls):
        """Factory method - gets class, not instance"""
        return cls(['mozzarella', 'tomatoes'])
    
    @classmethod
    def pepperoni(cls):
        return cls(['mozzarella', 'tomatoes', 'pepperoni'])
    
    @staticmethod
    def is_vegetarian(ingredients):
        """Doesn't need class or instance"""
        meat = ['pepperoni', 'ham', 'beef', 'chicken']
        return not any(item in meat for item in ingredients)
    
    def add_ingredient(self, ingredient):
        """Regular instance method"""
        self.ingredients.append(ingredient)

# Using class methods
pizza1 = Pizza.margherita()  # Don't need to pass ingredients
pizza2 = Pizza.pepperoni()

# Using static method
print(Pizza.is_vegetarian(['cheese', 'tomatoes']))  # True

# Why have both @classmethod and @staticmethod?
# @classmethod can access/modify class state
# @staticmethod can't access anything
# Just use regular functions? No, that would be too simple
```

## The MRO (Method Resolution Order) Drama

```python
# Simple inheritance
class Animal:
    def speak(self):
        return "Some sound"

class Dog(Animal):
    def speak(self):
        return "Woof!"

# Multiple inheritance (here comes trouble)
class A:
    def method(self):
        return "A"

class B:
    def method(self):
        return "B"

class C(A, B):  # Inherits from both
    pass

c = C()
print(c.method())  # "A" (first in inheritance list)

# The Diamond Problem
class Animal:
    def speak(self):
        return "Animal sound"

class Dog(Animal):
    def speak(self):
        return "Woof"

class Cat(Animal):
    def speak(self):
        return "Meow"

class Chimera(Dog, Cat):  # What happens now?
    pass

beast = Chimera()
print(beast.speak())  # "Woof" (Dog comes first)

# Check the MRO
print(Chimera.__mro__)
# (<class 'Chimera'>, <class 'Dog'>, <class 'Cat'>, <class 'Animal'>, <class 'object'>)

# C3 Linearization algorithm determines this
# It's complicated
# Just avoid multiple inheritance
```

## Metaclasses (Don't)

```python
# Metaclasses are classes that create classes
# If you think you need them, you don't

# The horror
class Meta(type):
    def __new__(cls, name, bases, attrs):
        # Modify class creation
        attrs['added_by_metaclass'] = True
        return super().__new__(cls, name, bases, attrs)

class MyClass(metaclass=Meta):
    pass

print(MyClass.added_by_metaclass)  # True

# "Metaclasses are deeper magic than 99% of users should ever worry about"
# - Tim Peters

# Seriously, just don't
```

## Slots (Memory Optimization Nobody Uses)

```python
# Normal classes use __dict__ for attributes
class Normal:
    def __init__(self, x, y):
        self.x = x
        self.y = y

n = Normal(1, 2)
n.z = 3  # Can add attributes dynamically
print(n.__dict__)  # {'x': 1, 'y': 2, 'z': 3}

# __slots__ restricts attributes (and saves memory)
class Optimized:
    __slots__ = ['x', 'y']  # Only these attributes allowed
    
    def __init__(self, x, y):
        self.x = x
        self.y = y

o = Optimized(1, 2)
# o.z = 3  # AttributeError! Can't add new attributes

# Why use __slots__?
# 1. Memory savings (no __dict__)
# 2. Slightly faster attribute access
# 3. Prevents dynamic attributes

# Why not use __slots__?
# 1. Can't add attributes dynamically
# 2. No __dict__ (some code expects it)
# 3. Multiple inheritance gets weird
# 4. Nobody remembers it exists
```

## Abstract Base Classes (Python Pretends to Have Interfaces)

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        """Subclasses must implement this"""
        pass
    
    @abstractmethod
    def perimeter(self):
        pass

# Can't instantiate abstract class
# shape = Shape()  # TypeError!

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):
        return self.width * self.height
    
    def perimeter(self):
        return 2 * (self.width + self.height)

# Now it works
rect = Rectangle(5, 3)
print(rect.area())  # 15

# Forgot to implement a method?
class BadCircle(Shape):
    def area(self):
        return 3.14 * self.radius ** 2
    # Forgot perimeter!

# circle = BadCircle()  # TypeError! Can't instantiate
```

## Dataclasses (Classes Without Boilerplate)

```python
from dataclasses import dataclass, field
from typing import List

# Before dataclasses (so much typing)
class OldPerson:
    def __init__(self, name: str, age: int):
        self.name = name
        self.age = age
    
    def __repr__(self):
        return f"OldPerson(name='{self.name}', age={self.age})"
    
    def __eq__(self, other):
        if not isinstance(other, OldPerson):
            return False
        return self.name == other.name and self.age == other.age

# With dataclasses (Python 3.7+)
@dataclass
class Person:
    name: str
    age: int
    hobbies: List[str] = field(default_factory=list)
    
    # __init__, __repr__, __eq__ generated automatically!

p = Person("Alice", 30)
print(p)  # Person(name='Alice', age=30, hobbies=[])

# Frozen dataclass (immutable)
@dataclass(frozen=True)
class Point:
    x: float
    y: float

p = Point(10, 20)
# p.x = 30  # FrozenInstanceError!

# With post-init processing
@dataclass
class Rectangle:
    width: float
    height: float
    area: float = field(init=False)
    
    def __post_init__(self):
        self.area = self.width * self.height
```

## Exercises: Try Not to Cry

1. **The Self Challenge**: Write a class method that doesn't use self. Now write an instance method that doesn't use self. Which one works?

2. **MRO Madness**: Create a diamond inheritance hierarchy. Predict the MRO without checking. Now check. Were you right?

3. **Dunder Overload**: Create a class that implements `__add__`, `__mul__`, and `__getitem__`. Make it do something unexpected but legal.

4. **Property Problems**: Create a property that has a setter but no getter. What happens when you try to read it?

5. **Class Variable Chaos**: Create a class with a mutable class variable. Create two instances and modify it from one. Observe the shared state disaster.

6. **Metaclass Mistake**: Try to create a metaclass that adds a method to every class. Realize why you shouldn't.

## Summary: What We've Learned

1. **self must be passed explicitly** (because explicit is better, apparently)
2. **Dunder methods are everywhere** (and there are too many)
3. **Nothing is truly private** (__ is just name mangling)
4. **Class variables are shared** (and mutable ones are dangerous)
5. **Properties are pythonic** (getters/setters are not)
6. **Multiple inheritance exists** (but shouldn't be used)
7. **Metaclasses are dark magic** (avoid at all costs)
8. **__slots__ saves memory** (but nobody uses it)
9. **ABC provides interfaces** (sort of)
10. **Dataclasses reduce boilerplate** (finally, something good)

Next chapter: Inheritance, where we'll learn how to make bad decisions hereditary and why super() is more complicated than it needs to be.

---

*"Python's OOP is like a buffet where everything is labeled wrong, but you're hungry so you eat it anyway."* - Developer migrating from Java