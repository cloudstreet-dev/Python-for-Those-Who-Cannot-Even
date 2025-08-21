# Chapter 11: Context Managers (The 'with' Statement Nobody Understands)

*"Context managers are like bouncers at a club: they make sure everything gets cleaned up when the party's over, even if someone throws up."* - A developer who finally understood __exit__

The `with` statement and context managers are Python's way of ensuring cleanup happens. It's like try-finally, but with more dunder methods and confusion about when to use it.

## What's a Context Manager?

A context manager is an object that defines what happens when you enter and exit a `with` block. It's Python's answer to RAII (Resource Acquisition Is Initialization), except Python needed two methods instead of one destructor because Python.

```python
# The most common context manager
with open('file.txt', 'r') as f:
    content = f.read()
# File automatically closed here, even if an exception occurs

# What's really happening:
f = open('file.txt', 'r')
try:
    content = f.read()
finally:
    f.close()

# But with more protocol:
manager = open('file.txt', 'r')
exit_method = type(manager).__exit__
value = type(manager).__enter__(manager)
exc = True

try:
    try:
        f = value  # This is what 'as f' assigns
        content = f.read()
    except:
        exc = False
        if not exit_method(manager, *sys.exc_info()):
            raise
finally:
    if exc:
        exit_method(manager, None, None, None)

# Yeah, 'with' is doing a lot behind the scenes
```

## Writing Your Own (But Why?)

```python
# The protocol: __enter__ and __exit__
class MyContextManager:
    def __enter__(self):
        print("Entering context")
        return self  # This is what gets assigned to the 'as' variable
    
    def __exit__(self, exc_type, exc_value, traceback):
        print("Exiting context")
        if exc_type:
            print(f"Exception occurred: {exc_value}")
        # Return True to suppress exception, False/None to propagate
        return False

with MyContextManager() as manager:
    print("Inside with block")
    # raise ValueError("Oops!")

# Output:
# Entering context
# Inside with block
# Exiting context

# Real-world example: Database connection
class DatabaseConnection:
    def __init__(self, connection_string):
        self.connection_string = connection_string
        self.connection = None
    
    def __enter__(self):
        print(f"Connecting to {self.connection_string}")
        self.connection = self._connect()  # Imagine this connects
        return self.connection
    
    def __exit__(self, exc_type, exc_value, traceback):
        print("Closing connection")
        if self.connection:
            self.connection.close()
        if exc_type == DatabaseError:
            print(f"Database error occurred: {exc_value}")
            return True  # Suppress database errors
        return False  # Let other exceptions propagate
    
    def _connect(self):
        # Pretend this returns a connection
        return "connection"

with DatabaseConnection("localhost:5432") as conn:
    # Use connection
    result = conn.execute("SELECT * FROM users")
# Connection automatically closed
```

## The contextlib Module (Making Context Managers Bearable)

```python
from contextlib import contextmanager, closing, suppress, redirect_stdout

# @contextmanager decorator (the sane way)
@contextmanager
def my_context():
    print("Entering")
    resource = "some resource"
    try:
        yield resource  # This is where the with block runs
    finally:
        print("Exiting")

with my_context() as r:
    print(f"Using {r}")

# closing() - Ensures something with close() gets closed
from urllib.request import urlopen

with closing(urlopen('http://example.com')) as page:
    content = page.read()
# page.close() automatically called

# suppress() - Ignore specific exceptions
import os

with suppress(FileNotFoundError):
    os.remove('nonexistent.txt')  # No error if file doesn't exist

# Without suppress:
try:
    os.remove('nonexistent.txt')
except FileNotFoundError:
    pass

# redirect_stdout/stderr - Capture output
import io

f = io.StringIO()
with redirect_stdout(f):
    print("This goes to StringIO")
    help(int)  # This too

output = f.getvalue()  # Contains all printed output

# Multiple context managers (Python 3.1+)
with open('input.txt') as infile, open('output.txt', 'w') as outfile:
    outfile.write(infile.read())

# Equivalent to nested with:
with open('input.txt') as infile:
    with open('output.txt', 'w') as outfile:
        outfile.write(infile.read())
```

## ExitStack (When Things Get Complicated)

```python
from contextlib import ExitStack

# Managing dynamic number of context managers
def process_files(filenames):
    with ExitStack() as stack:
        files = [
            stack.enter_context(open(fname))
            for fname in filenames
        ]
        # All files will be closed when we exit
        for f in files:
            print(f.read())

# Conditional context managers
def maybe_open(filename, use_file):
    with ExitStack() as stack:
        if use_file:
            f = stack.enter_context(open(filename))
            return f.read()
        else:
            return "No file used"

# Cleanup callbacks
def complex_operation():
    with ExitStack() as stack:
        # Register cleanup functions
        stack.callback(print, "Cleanup 1")
        stack.callback(print, "Cleanup 2")
        
        # They run in reverse order (LIFO)
        temp_file = create_temp_file()
        stack.callback(os.remove, temp_file)
        
        # Do work
        process(temp_file)
    # All callbacks run here

# Moving cleanup responsibility
def get_resource():
    stack = ExitStack()
    try:
        resource = stack.enter_context(open('file.txt'))
        # Transfer cleanup responsibility to caller
        stack.pop_all()
        return resource, stack
    except Exception:
        stack.close()  # Clean up on error
        raise

resource, cleanup = get_resource()
try:
    use(resource)
finally:
    cleanup.close()  # Caller responsible for cleanup
```

## Async Context Managers (Because Regular Ones Weren't Enough)

```python
# Async context managers for async code
class AsyncContextManager:
    async def __aenter__(self):
        print("Async entering")
        await asyncio.sleep(1)
        return self
    
    async def __aexit__(self, exc_type, exc_value, traceback):
        print("Async exiting")
        await asyncio.sleep(1)
        return False

async def use_async_context():
    async with AsyncContextManager() as manager:
        print("Inside async with")
        await asyncio.sleep(1)

# asyncio.run(use_async_context())

# contextlib supports async too
from contextlib import asynccontextmanager

@asynccontextmanager
async def async_resource():
    print("Acquiring async resource")
    await asyncio.sleep(1)
    try:
        yield "resource"
    finally:
        print("Releasing async resource")
        await asyncio.sleep(1)

# Async ExitStack
from contextlib import AsyncExitStack

async def process_async_files(filenames):
    async with AsyncExitStack() as stack:
        files = []
        for fname in filenames:
            f = await stack.enter_async_context(aiofiles.open(fname))
            files.append(f)
        # Process files
```

## Common Context Managers You'll Actually Use

```python
# File handling (the classic)
with open('data.txt', 'r') as f:
    data = f.read()

# Thread locks
import threading
lock = threading.Lock()

with lock:
    # Critical section
    shared_resource += 1

# Decimal precision
from decimal import localcontext, Decimal

with localcontext() as ctx:
    ctx.prec = 50  # 50 decimal places
    result = Decimal(1) / Decimal(7)
    print(result)  # 50 decimal places

# Outside the context, back to default precision

# Temporary directory
import tempfile

with tempfile.TemporaryDirectory() as tmpdir:
    # tmpdir is a path to a temporary directory
    temp_file = os.path.join(tmpdir, 'temp.txt')
    with open(temp_file, 'w') as f:
        f.write('Temporary data')
# Directory and contents deleted automatically

# Database transactions
with db.transaction():
    db.execute("INSERT INTO users VALUES (?)", user_data)
    db.execute("UPDATE accounts SET balance = ?", new_balance)
    # Commits on success, rolls back on exception

# Time measurement
import time
from contextlib import contextmanager

@contextmanager
def timer():
    start = time.perf_counter()
    try:
        yield
    finally:
        end = time.perf_counter()
        print(f"Elapsed: {end - start:.4f} seconds")

with timer():
    time.sleep(1)
    # Do expensive operation

# Mock patches in testing
from unittest.mock import patch

with patch('module.function') as mock_func:
    mock_func.return_value = 42
    result = module.function()  # Returns 42
```

## The Dark Side of Context Managers

```python
# Context managers that do too much
class OverEngineeredContextManager:
    def __enter__(self):
        # Don't do all this!
        self.connect_to_database()
        self.authenticate_user()
        self.load_configuration()
        self.start_background_threads()
        self.initialize_cache()
        return self
    
    def __exit__(self, *args):
        # This is a cleanup nightmare
        self.save_state()
        self.stop_threads()
        self.clear_cache()
        self.disconnect()

# Exception suppression confusion
class BadContextManager:
    def __exit__(self, exc_type, exc_value, traceback):
        return True  # Suppresses ALL exceptions silently!

with BadContextManager():
    1 / 0  # No error visible!
    undefined_variable  # This too!
    sys.exit(1)  # Even this!

# Resource leaks with exceptions in __enter__
class LeakyContextManager:
    def __enter__(self):
        self.resource = acquire_resource()
        raise ValueError("Oops!")  # __exit__ never called!
        return self.resource
    
    def __exit__(self, *args):
        release_resource(self.resource)  # Never happens

# Nested context manager hell
with open('file1') as f1:
    with open('file2') as f2:
        with open('file3') as f3:
            with lock1:
                with lock2:
                    with transaction():
                        # Good luck debugging this

# Context managers that modify global state
class GlobalStateManager:
    def __enter__(self):
        self.old_state = global_state
        global_state = "modified"  # Dangerous!
    
    def __exit__(self, *args):
        global_state = self.old_state

# Reentrant context managers (most aren't)
lock = threading.Lock()

with lock:
    with lock:  # Deadlock! Lock isn't reentrant
        pass

# Use threading.RLock() for reentrant locks
```

## Creating Reusable Context Managers

```python
# Context manager as a decorator
from contextlib import ContextDecorator

class log_execution(ContextDecorator):
    def __enter__(self):
        print("Starting execution")
        return self
    
    def __exit__(self, *exc):
        print("Finished execution")
        return False

@log_execution()
def my_function():
    print("Doing work")

my_function()
# Starting execution
# Doing work  
# Finished execution

# Or use it as context manager
with log_execution():
    print("Also doing work")

# Parameterized context managers
class timeout:
    def __init__(self, seconds):
        self.seconds = seconds
    
    def __enter__(self):
        signal.signal(signal.SIGALRM, self._timeout_handler)
        signal.alarm(self.seconds)
    
    def __exit__(self, *args):
        signal.alarm(0)  # Cancel alarm
    
    def _timeout_handler(self, signum, frame):
        raise TimeoutError(f"Operation exceeded {self.seconds} seconds")

with timeout(5):
    long_running_operation()  # Raises TimeoutError after 5 seconds

# Context manager factory
def make_context_manager(enter_msg, exit_msg):
    @contextmanager
    def manager():
        print(enter_msg)
        try:
            yield
        finally:
            print(exit_msg)
    return manager

custom_context = make_context_manager("Hello", "Goodbye")

with custom_context():
    print("In between")
```

## Exercises: Try Not to Cry

1. **Resource Pool**: Create a context manager that manages a pool of connections. It should acquire a connection on enter and return it to the pool on exit.

2. **Nested Transaction**: Write a context manager that supports nested transactions with rollback on any level's failure.

3. **Measurement Manager**: Create a context manager that measures time, memory usage, and counts function calls within its block.

4. **Exception Logger**: Build a context manager that logs all exceptions but only suppresses specific ones you configure.

5. **State Snapshot**: Make a context manager that takes a snapshot of an object's state and can restore it on exit (optionally based on success/failure).

6. **Async Rate Limiter**: Create an async context manager that rate-limits operations within its block.

## Summary: What We've Learned

1. **Context managers ensure cleanup** (even with exceptions)
2. **__enter__ and __exit__ define the protocol** (two methods for one job)
3. **@contextmanager makes it easier** (generators for context managers)
4. **with can handle multiple managers** (since Python 3.1)
5. **ExitStack handles dynamic contexts** (when you don't know how many)
6. **Exception suppression is dangerous** (returning True in __exit__)
7. **contextlib has useful utilities** (closing, suppress, redirect_stdout)
8. **Async context managers exist** (__aenter__ and __aexit__)
9. **Don't do too much in context managers** (keep them simple)
10. **Most things should just use try-finally** (but context managers look cooler)

Next chapter: Async/Await, where we'll learn how Python decided to make concurrent programming "easier" by adding new keywords and making everything more confusing.

---

*"Context managers are like automatic door closers: great when they work, annoying when you're trying to prop the door open, and nobody understands the spring mechanism."* - Janitor turned programmer