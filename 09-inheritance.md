# Chapter 9: Inheritance (Making Bad Decisions Hereditary)

*"Inheritance is when your code inherits all the problems from its parents, plus some new ones of its own."* - A developer debugging a 10-level inheritance hierarchy

Inheritance in Python is like inheritance in real life: you get a bunch of stuff you didn't ask for, some of it's broken, and by the time you figure out where it came from, it's too late to give it back.

## Single Inheritance: At Least It's Not Multiple

```python
# Basic inheritance (the gateway drug)
class Animal:
    def __init__(self, name, species):
        self.name = name
        self.species = species
    
    def speak(self):
        return f"{self.name} makes a sound"
    
    def move(self):
        return f"{self.name} moves somehow"

class Dog(Animal):
    def __init__(self, name, breed):
        # Call parent constructor
        super().__init__(name, "Canine")
        self.breed = breed
    
    def speak(self):
        # Override parent method
        return f"{self.name} says Woof!"
    
    def fetch(self):
        # New method not in parent
        return f"{self.name} fetches the ball"

fido = Dog("Fido", "Golden Retriever")
print(fido.speak())  # "Fido says Woof!" (overridden)
print(fido.move())   # "Fido moves somehow" (inherited)
print(fido.fetch())  # "Fido fetches the ball" (new)

# Check inheritance
print(isinstance(fido, Dog))     # True
print(isinstance(fido, Animal))  # True
print(issubclass(Dog, Animal))   # True
```

## Super() and the Method Resolution Order

`super()` is Python's way of calling parent methods, and it's more complicated than it needs to be.

```python
# Simple super() usage
class Parent:
    def __init__(self, value):
        self.value = value
        print(f"Parent init with {value}")

class Child(Parent):
    def __init__(self, value, extra):
        super().__init__(value)  # Calls Parent.__init__
        self.extra = extra
        print(f"Child init with {extra}")

# The old way (Python 2, don't do this)
class OldChild(Parent):
    def __init__(self, value, extra):
        Parent.__init__(self, value)  # Direct call
        self.extra = extra

# super() without arguments (Python 3)
class Modern(Parent):
    def __init__(self, value):
        super().__init__(value)  # Figures out the class and self

# super() with arguments (rarely needed)
class Explicit(Parent):
    def __init__(self, value):
        super(Explicit, self).__init__(value)  # Explicit class and instance

# super() in other methods
class ExtendedDog(Dog):
    def speak(self):
        parent_speak = super().speak()  # Calls Dog.speak()
        return f"{parent_speak} and also Howls!"

# The weird part: super() doesn't always call the parent
class A:
    def method(self):
        print("A.method")

class B(A):
    def method(self):
        print("B.method")
        super().method()

class C(A):
    def method(self):
        print("C.method")
        super().method()

class D(B, C):  # Multiple inheritance
    def method(self):
        print("D.method")
        super().method()

d = D()
d.method()
# Output:
# D.method
# B.method
# C.method  (Wait, what? B's super() called C!)
# A.method

# super() follows the MRO, not the inheritance tree!
print(D.__mro__)
# (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)
```

## Multiple Inheritance (Making Things Complicated Since Forever)

```python
# The classic diamond problem
class Vehicle:
    def __init__(self, brand):
        self.brand = brand
        print(f"Vehicle init: {brand}")
    
    def start(self):
        return "Vehicle starting"

class Car(Vehicle):
    def __init__(self, brand, doors):
        super().__init__(brand)
        self.doors = doors
        print(f"Car init: {doors} doors")
    
    def start(self):
        return "Car starting with key"

class Boat(Vehicle):
    def __init__(self, brand, length):
        super().__init__(brand)
        self.length = length
        print(f"Boat init: {length}m long")
    
    def start(self):
        return "Boat starting with button"

class AmphibiousVehicle(Car, Boat):
    def __init__(self, brand, doors, length):
        # This is where it gets messy
        super().__init__(brand, doors)  # Only calls Car.__init__!
        self.length = length  # Have to set this manually
        print("Amphibious init")
    
    def start(self):
        # Which start method do we want?
        car_start = Car.start(self)  # Explicit call
        boat_start = Boat.start(self)  # Explicit call
        return f"{car_start} and {boat_start}"

# Creating the monstrosity
amphib = AmphibiousVehicle("Tesla", 4, 5)
# Vehicle init: Tesla
# Car init: 4 doors
# Amphibious init

# Notice Boat.__init__ was never called!
# super() only follows MRO once

# The MRO
print(AmphibiousVehicle.__mro__)
# AmphibiousVehicle -> Car -> Boat -> Vehicle -> object

# Cooperative multiple inheritance (the "right" way)
class CooperativeVehicle:
    def __init__(self, brand, **kwargs):
        super().__init__(**kwargs)  # Pass remaining kwargs up
        self.brand = brand

class CooperativeCar(CooperativeVehicle):
    def __init__(self, doors, **kwargs):
        super().__init__(**kwargs)
        self.doors = doors

class CooperativeBoat(CooperativeVehicle):
    def __init__(self, length, **kwargs):
        super().__init__(**kwargs)
        self.length = length

class CooperativeAmphibious(CooperativeCar, CooperativeBoat):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)

# Now everyone gets initialized
amphib2 = CooperativeAmphibious(brand="Tesla", doors=4, length=5)
# But this requires all classes to cooperate
# Good luck with third-party classes
```

## Mixins (Multiple Inheritance That Pretends to Be Useful)

```python
# Mixins are classes that provide functionality but aren't meant to stand alone
class TimestampMixin:
    """Adds timestamp functionality to any class"""
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        from datetime import datetime
        self.created_at = datetime.now()
        self.updated_at = datetime.now()
    
    def touch(self):
        from datetime import datetime
        self.updated_at = datetime.now()

class SerializableMixin:
    """Adds JSON serialization to any class"""
    def to_dict(self):
        return {
            key: value
            for key, value in self.__dict__.items()
            if not key.startswith('_')
        }
    
    def to_json(self):
        import json
        return json.dumps(self.to_dict(), default=str)

class LoggingMixin:
    """Adds logging to any class"""
    def log(self, message):
        print(f"[{self.__class__.__name__}] {message}")

# Using mixins
class User(TimestampMixin, SerializableMixin, LoggingMixin):
    def __init__(self, name, email):
        super().__init__()  # Initializes mixins
        self.name = name
        self.email = email
        self.log(f"Created user {name}")

user = User("Alice", "alice@example.com")
print(user.to_json())  # From SerializableMixin
user.touch()  # From TimestampMixin
user.log("Did something")  # From LoggingMixin

# The order matters!
class Wrong(LoggingMixin, TimestampMixin, SerializableMixin):
    # Different MRO, might behave differently
    pass

# Mixins are just multiple inheritance with good intentions
```

## Abstract Base Classes: Python Pretends to Have Interfaces

```python
from abc import ABC, abstractmethod

# Abstract base class
class DatabaseInterface(ABC):
    @abstractmethod
    def connect(self, connection_string):
        """Subclasses must implement connect"""
        pass
    
    @abstractmethod
    def execute(self, query):
        """Subclasses must implement execute"""
        pass
    
    def disconnect(self):
        """Concrete method (optional to override)"""
        print("Disconnecting...")

# Can't instantiate abstract class
# db = DatabaseInterface()  # TypeError!

class MySQLDatabase(DatabaseInterface):
    def connect(self, connection_string):
        print(f"Connecting to MySQL: {connection_string}")
    
    def execute(self, query):
        print(f"Executing MySQL query: {query}")
        return ["result1", "result2"]

class PostgreSQLDatabase(DatabaseInterface):
    def connect(self, connection_string):
        print(f"Connecting to PostgreSQL: {connection_string}")
    
    def execute(self, query):
        print(f"Executing PostgreSQL query: {query}")
        return ["result1", "result2"]

# Virtual subclasses (duck typing with ABC)
from collections.abc import Sized

class MySized:
    def __len__(self):
        return 42

# Register as virtual subclass
Sized.register(MySized)
print(isinstance(MySized(), Sized))  # True
print(issubclass(MySized, Sized))  # True

# But MySized doesn't actually inherit from Sized!
print(MySized.__mro__)  # Just (<class 'MySized'>, <class 'object'>)

# This is Python pretending to have interfaces while maintaining duck typing
```

## Composition vs Inheritance (The Eternal Debate)

```python
# Inheritance approach (IS-A relationship)
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary
    
    def work(self):
        return f"{self.name} is working"

class Manager(Employee):
    def __init__(self, name, salary, department):
        super().__init__(name, salary)
        self.department = department
    
    def manage(self):
        return f"{self.name} is managing {self.department}"

# Problems with inheritance:
# What if we need a ContractEmployee who isn't full-time?
# What if we need a TechnicalManager who codes and manages?
# Multiple inheritance gets messy fast

# Composition approach (HAS-A relationship)
class WorkRole:
    def work(self):
        return "Working"

class ManagementRole:
    def __init__(self, department):
        self.department = department
    
    def manage(self):
        return f"Managing {self.department}"

class CodingRole:
    def code(self):
        return "Writing code"

class Employee:
    def __init__(self, name, salary, roles=None):
        self.name = name
        self.salary = salary
        self.roles = roles or []
    
    def add_role(self, role):
        self.roles.append(role)
    
    def perform_duties(self):
        duties = []
        for role in self.roles:
            if hasattr(role, 'work'):
                duties.append(role.work())
            if hasattr(role, 'manage'):
                duties.append(role.manage())
            if hasattr(role, 'code'):
                duties.append(role.code())
        return duties

# More flexible!
tech_manager = Employee("Bob", 120000)
tech_manager.add_role(CodingRole())
tech_manager.add_role(ManagementRole("Engineering"))

# "Favor composition over inheritance" - Gang of Four
# "But Python developers will use inheritance anyway" - Reality
```

## The Perils of Deep Inheritance

```python
# When inheritance goes too far
class LivingThing:
    def exist(self):
        return "I exist"

class Organism(LivingThing):
    def metabolize(self):
        return "Processing energy"

class Animal(Organism):
    def move(self):
        return "Moving"

class Vertebrate(Animal):
    def has_spine(self):
        return True

class Mammal(Vertebrate):
    def feed_young_milk(self):
        return "Feeding milk"

class Primate(Mammal):
    def use_hands(self):
        return "Using hands"

class Human(Primate):
    def think(self):
        return "I think therefore I am"

class Programmer(Human):
    def code(self):
        return "Writing bugs"

class PythonProgrammer(Programmer):
    def complain_about_whitespace(self):
        return "Tabs vs spaces debate"

# 9 levels deep!
dev = PythonProgrammer()
print(PythonProgrammer.__mro__)  # 11 classes in the MRO!

# Problems:
# 1. Hard to understand what PythonProgrammer can do
# 2. Changes to LivingThing affect everything
# 3. What if we need a PythonProgrammer who's a Robot?
# 4. The stack traces will be enormous

# Better approach: Keep it shallow
class Developer:
    def __init__(self, name, language):
        self.name = name
        self.language = language
    
    def code(self):
        return f"Writing {self.language}"

# Simple, clear, maintainable
```

## Method Overriding Gotchas

```python
class Parent:
    def method(self, x, y):
        return x + y

class Child(Parent):
    def method(self, x, y, z):  # Different signature!
        return x + y + z

# This "works" but violates Liskov Substitution Principle
parent = Parent()
child = Child()

print(parent.method(1, 2))     # 3
print(child.method(1, 2, 3))   # 6
# print(child.method(1, 2))    # TypeError! Child needs 3 args

# Python doesn't enforce signature compatibility
# You can break everything and Python won't stop you

# Private method "overriding" (doesn't really override)
class Parent:
    def __private(self):
        return "Parent's private"
    
    def call_private(self):
        return self.__private()

class Child(Parent):
    def __private(self):  # This is a different method!
        return "Child's private"

child = Child()
print(child.call_private())  # "Parent's private"
# Name mangling means Child.__private is actually Child._Child__private
# Parent.__private is Parent._Parent__private
# They're different methods!
```

## The `property` Inheritance Nightmare

```python
class Parent:
    def __init__(self):
        self._value = 10
    
    @property
    def value(self):
        print("Parent getter")
        return self._value
    
    @value.setter
    def value(self, val):
        print("Parent setter")
        self._value = val

class Child(Parent):
    @property
    def value(self):
        print("Child getter")
        return super().value * 2  # Calls parent getter
    
    # Oops, forgot to override setter!
    # Child uses Parent's setter

child = Child()
print(child.value)  # Child getter, then Parent getter, returns 20
child.value = 5     # Parent setter (not what you might expect!)

# To properly override a property:
class ProperChild(Parent):
    @property
    def value(self):
        print("ProperChild getter")
        return super().value * 2
    
    @value.setter
    def value(self, val):
        print("ProperChild setter")
        # Call parent setter through super()
        Parent.value.fset(self, val)  # Ugly but necessary

# Properties + inheritance = confusion
```

## Exercises: Try Not to Cry

1. **MRO Maze**: Create a class that inherits from 5 different classes in a diamond pattern. Predict the MRO. Call a method that exists in all parents and trace its path.

2. **Super() Surprise**: Create a situation where `super()` doesn't call the immediate parent class. Explain why.

3. **Mixin Madness**: Create 3 mixins that all define `__init__`. Make them work together properly with super().

4. **Abstract Disaster**: Create an abstract base class with 10 abstract methods. Implement a subclass that "forgets" one. Find out when Python tells you.

5. **Composition Challenge**: Take a deep inheritance hierarchy and refactor it to use composition instead. Which is clearer?

6. **Property Problems**: Create a property in a parent class and override just the getter in a child class. Try to use the setter. What happens?

## Summary: What We've Learned

1. **Single inheritance is simple** (until you add super())
2. **super() follows MRO** (not the parent class)
3. **Multiple inheritance is confusing** (diamond problem is real)
4. **MRO uses C3 linearization** (nobody understands it)
5. **Mixins are just multiple inheritance** (with good intentions)
6. **Abstract base classes exist** (but aren't enforced until instantiation)
7. **Composition is often better** (but requires more setup)
8. **Deep inheritance is bad** (keep it shallow)
9. **Method signatures aren't enforced** (you can break LSP easily)
10. **Properties + inheritance = pain** (getters and setters don't inherit well)

Next up: Generators and Iterators, where we'll learn how to be lazy efficiently and why `yield` is not just about giving up.

---

*"Inheritance in Python is like a family reunion: everyone's related, nobody's quite sure how, and by the end, you wish you'd just stayed home with composition."* - Developer refactoring a legacy codebase