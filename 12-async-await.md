# Chapter 12: Async/Await (Concurrent Disappointment)

*"Python's async is like a restaurant with one waiter who's really good at juggling: impressive to watch, but you're still waiting for your food."* - Developer who just learned about the GIL

Asyncio is Python's answer to concurrent programming. It's single-threaded asynchronous programming, which is like saying "I can multitask, but only one thing at a time." Welcome to cooperative multitasking, where your code has to be polite and yield control voluntarily.

## Asyncio: Making Simple Things Complicated Since 2014

```python
import asyncio

# Synchronous (simple, understandable)
def fetch_data():
    time.sleep(1)  # Simulate network delay
    return "data"

result = fetch_data()
print(result)

# Asynchronous (complicated, confusing)
async def fetch_data_async():
    await asyncio.sleep(1)  # Simulate network delay
    return "data"

# Can't just call it!
# result = fetch_data_async()  # Returns a coroutine object, not data!

# Need an event loop
async def main():
    result = await fetch_data_async()
    print(result)

asyncio.run(main())

# Or the old way (Python < 3.7)
loop = asyncio.get_event_loop()
result = loop.run_until_complete(fetch_data_async())
loop.close()

# Why is this better? It's not, unless you're doing many things
```

## Async/Await: JavaScript Refugees Will Feel at Home

```python
# Coroutines: Functions that can pause
async def my_coroutine():
    print("Starting")
    await asyncio.sleep(1)  # Pause here, let others run
    print("Resumed after 1 second")
    return "Done"

# await can only be used in async functions
# def regular_function():
#     await asyncio.sleep(1)  # SyntaxError!

# Async functions return coroutines
coro = my_coroutine()
print(type(coro))  # <class 'coroutine'>

# Coroutines do nothing until awaited
# coro just sits there, judging you

# Multiple coroutines
async def task1():
    await asyncio.sleep(2)
    return "Task 1 done"

async def task2():
    await asyncio.sleep(1)
    return "Task 2 done"

async def run_sequential():
    # Sequential (slow)
    result1 = await task1()  # Wait 2 seconds
    result2 = await task2()  # Then wait 1 second
    return result1, result2  # Total: 3 seconds

async def run_concurrent():
    # Concurrent (fast)
    results = await asyncio.gather(task1(), task2())
    return results  # Total: 2 seconds (max of both)

# Or use create_task
async def run_with_tasks():
    t1 = asyncio.create_task(task1())
    t2 = asyncio.create_task(task2())
    # They start immediately
    result1 = await t1
    result2 = await t2
    return result1, result2
```

## The Event Loop (The Puppet Master)

```python
# The event loop is what actually runs async code
import asyncio

# Get the current loop
async def show_loop():
    loop = asyncio.get_running_loop()
    print(f"Loop: {loop}")
    print(f"Is running: {loop.is_running()}")

# Low-level loop usage (don't do this)
loop = asyncio.new_event_loop()
asyncio.set_event_loop(loop)

# Schedule a callback
def callback(n):
    print(f"Callback {n} called")

loop.call_soon(callback, 1)
loop.call_later(2, callback, 2)  # Call after 2 seconds

# Run forever (until stopped)
# loop.run_forever()

# Schedule a coroutine
async def coro():
    await asyncio.sleep(1)
    return "result"

future = asyncio.ensure_future(coro())
result = loop.run_until_complete(future)

# The modern way: just use asyncio.run()
async def main():
    # asyncio.run() creates loop, runs code, closes loop
    await asyncio.sleep(1)

asyncio.run(main())

# Event loop is thread-specific
# Each thread can have its own loop
# But only one loop per thread
```

## Tasks, Futures, and Broken Promises

```python
# Tasks are futures that wrap coroutines
async def slow_operation(n):
    await asyncio.sleep(n)
    return n * 2

async def main():
    # Create task (starts immediately)
    task = asyncio.create_task(slow_operation(2))
    print(f"Task created: {task}")
    
    # Do other things while task runs
    await asyncio.sleep(1)
    print("Doing other work...")
    
    # Get result when ready
    result = await task
    print(f"Result: {result}")

# Futures (low-level, usually don't use directly)
async def with_future():
    future = asyncio.Future()
    
    # Set result from elsewhere
    async def setter():
        await asyncio.sleep(1)
        future.set_result("Future completed")
    
    asyncio.create_task(setter())
    result = await future
    return result

# Task cancellation
async def cancellable():
    try:
        await asyncio.sleep(10)
    except asyncio.CancelledError:
        print("I was cancelled!")
        raise  # Re-raise to properly cancel

async def cancel_example():
    task = asyncio.create_task(cancellable())
    await asyncio.sleep(1)
    task.cancel()  # Request cancellation
    
    try:
        await task
    except asyncio.CancelledError:
        print("Task was cancelled")

# Timeouts
async def with_timeout():
    try:
        async with asyncio.timeout(2):  # Python 3.11+
            await asyncio.sleep(10)
    except asyncio.TimeoutError:
        print("Timed out!")
    
    # Or older way
    try:
        await asyncio.wait_for(asyncio.sleep(10), timeout=2)
    except asyncio.TimeoutError:
        print("Also timed out!")
```

## Async Context Managers and Iterators (More Async Everything)

```python
# Async context managers
class AsyncResource:
    async def __aenter__(self):
        print("Acquiring resource")
        await asyncio.sleep(1)
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        print("Releasing resource")
        await asyncio.sleep(1)

async def use_resource():
    async with AsyncResource() as resource:
        print("Using resource")
        await asyncio.sleep(1)

# Async iterators
class AsyncRange:
    def __init__(self, start, stop):
        self.start = start
        self.stop = stop
    
    def __aiter__(self):
        return self
    
    async def __anext__(self):
        if self.start >= self.stop:
            raise StopAsyncIteration
        await asyncio.sleep(0.1)  # Simulate async work
        value = self.start
        self.start += 1
        return value

async def use_async_iterator():
    async for i in AsyncRange(0, 5):
        print(i)

# Async generators (easier)
async def async_generator():
    for i in range(5):
        await asyncio.sleep(0.1)
        yield i

async def use_async_generator():
    async for value in async_generator():
        print(value)

# Async comprehensions (because why not)
async def async_comprehension():
    results = [i async for i in async_generator()]
    # Also dict and set comprehensions
    results_dict = {i: i**2 async for i in async_generator()}
    return results
```

## When to Use Async (Spoiler: Less Often Than You Think)

```python
# GOOD use case: I/O bound operations
async def fetch_multiple_urls(urls):
    async with aiohttp.ClientSession() as session:
        tasks = []
        for url in urls:
            tasks.append(fetch_url(session, url))
        results = await asyncio.gather(*tasks)
        return results

async def fetch_url(session, url):
    async with session.get(url) as response:
        return await response.text()

# BAD use case: CPU bound operations
async def calculate_prime(n):
    # This doesn't help at all!
    await asyncio.sleep(0)  # Pointless yield
    # CPU intensive work still blocks
    for i in range(2, n):
        if n % i == 0:
            return False
    return True

# For CPU bound, use multiprocessing
from concurrent.futures import ProcessPoolExecutor

async def calculate_primes_properly(numbers):
    loop = asyncio.get_running_loop()
    with ProcessPoolExecutor() as executor:
        tasks = []
        for n in numbers:
            tasks.append(loop.run_in_executor(executor, is_prime, n))
        results = await asyncio.gather(*tasks)
        return results

# When async makes sense:
# - Network requests
# - Database queries  
# - File I/O (with aiofiles)
# - Websockets
# - Multiple independent I/O operations

# When async doesn't help:
# - CPU intensive calculations
# - Simple scripts
# - When you need true parallelism (GIL!)
# - When synchronous is simpler and fast enough
```

## The GIL: Why Your Async Code Isn't Really Parallel

```python
# The Global Interpreter Lock (GIL) means only one thread executes Python at a time
import time
import asyncio
import threading

# CPU bound work
def cpu_bound(n):
    total = 0
    for i in range(n):
        total += i
    return total

# Async version (doesn't help!)
async def async_cpu_bound(n):
    return cpu_bound(n)

async def parallel_cpu_async():
    # These don't run in parallel!
    start = time.time()
    tasks = [async_cpu_bound(10_000_000) for _ in range(4)]
    await asyncio.gather(*tasks)
    print(f"Async time: {time.time() - start}")

# Threading version (also doesn't help!)
def parallel_cpu_threads():
    start = time.time()
    threads = []
    for _ in range(4):
        t = threading.Thread(target=cpu_bound, args=(10_000_000,))
        t.start()
        threads.append(t)
    for t in threads:
        t.join()
    print(f"Thread time: {time.time() - start}")

# Multiprocessing (actually parallel!)
from multiprocessing import Pool

def parallel_cpu_processes():
    start = time.time()
    with Pool(4) as p:
        p.map(cpu_bound, [10_000_000] * 4)
    print(f"Process time: {time.time() - start}")

# Async is concurrency, not parallelism
# Concurrency: dealing with multiple things at once
# Parallelism: doing multiple things at once
# Python async: concurrency without parallelism (thanks, GIL!)
```

## Common Async Pitfalls and Gotchas

```python
# Forgetting await
async def forgot_await():
    asyncio.sleep(1)  # This does nothing! Returns coroutine object
    # Should be: await asyncio.sleep(1)

# Blocking the event loop
async def blocking_bad():
    time.sleep(1)  # BLOCKS EVERYTHING!
    # Should use: await asyncio.sleep(1)
    
    requests.get("http://example.com")  # BLOCKS!
    # Should use: aiohttp
    
    with open("file.txt") as f:  # BLOCKS!
        content = f.read()
    # Should use: aiofiles

# Mixing sync and async
def sync_function():
    # Can't await in sync function!
    # await async_function()  # SyntaxError
    
    # Have to use asyncio.run()
    result = asyncio.run(async_function())
    return result

# Creating tasks but not awaiting them
async def fire_and_forget_bad():
    asyncio.create_task(slow_operation())  # Task created but not awaited
    # Task might not complete before program ends!

async def fire_and_forget_good():
    task = asyncio.create_task(slow_operation())
    # ... do other stuff ...
    await task  # Make sure to await eventually

# Shared state without locks
counter = 0

async def increment():
    global counter
    temp = counter
    await asyncio.sleep(0)  # Yield control
    counter = temp + 1  # Race condition!

async def safe_increment():
    async with lock:  # Use asyncio.Lock()
        global counter
        temp = counter
        await asyncio.sleep(0)
        counter = temp + 1

# Exception handling
async def exception_example():
    tasks = [
        asyncio.create_task(may_fail()),
        asyncio.create_task(may_fail()),
    ]
    
    # gather with return_exceptions
    results = await asyncio.gather(*tasks, return_exceptions=True)
    for result in results:
        if isinstance(result, Exception):
            print(f"Task failed: {result}")
```

## Real-World Async Patterns

```python
# Rate limiting
class RateLimiter:
    def __init__(self, rate, per):
        self.rate = rate
        self.per = per
        self.allowance = rate
        self.last_check = time.monotonic()
    
    async def acquire(self):
        current = time.monotonic()
        time_passed = current - self.last_check
        self.last_check = current
        self.allowance += time_passed * (self.rate / self.per)
        
        if self.allowance > self.rate:
            self.allowance = self.rate
        
        if self.allowance < 1.0:
            sleep_time = (1.0 - self.allowance) * (self.per / self.rate)
            await asyncio.sleep(sleep_time)
            self.allowance = 0.0
        else:
            self.allowance -= 1.0

# Producer-consumer with queues
async def producer(queue):
    for i in range(10):
        await asyncio.sleep(1)
        await queue.put(f"item {i}")
        print(f"Produced item {i}")

async def consumer(queue, name):
    while True:
        item = await queue.get()
        print(f"{name} consumed {item}")
        await asyncio.sleep(2)
        queue.task_done()

async def main():
    queue = asyncio.Queue()
    
    # Start consumers
    consumers = [asyncio.create_task(consumer(queue, f"C{i}")) for i in range(3)]
    
    # Run producer
    await producer(queue)
    
    # Wait for all items to be processed
    await queue.join()
    
    # Cancel consumers
    for c in consumers:
        c.cancel()

# Semaphore for limiting concurrent operations
async def fetch_with_limit(url, semaphore):
    async with semaphore:  # Acquire semaphore
        async with aiohttp.ClientSession() as session:
            async with session.get(url) as response:
                return await response.text()

async def fetch_many(urls, max_concurrent=10):
    semaphore = asyncio.Semaphore(max_concurrent)
    tasks = [fetch_with_limit(url, semaphore) for url in urls]
    return await asyncio.gather(*tasks)
```

## Exercises: Try Not to Cry

1. **Async Web Scraper**: Write an async function that fetches 100 URLs concurrently but with a maximum of 10 concurrent requests.

2. **Timeout Decorator**: Create a decorator that adds a timeout to any async function.

3. **Async Rate Monitor**: Build an async context manager that monitors and reports the rate of operations within its block.

4. **Queue Pipeline**: Create a pipeline of async workers connected by queues. Each worker transforms data and passes it to the next.

5. **Async vs Sync Benchmark**: Compare the performance of fetching 50 URLs sequentially (sync), concurrently (async), and in parallel (threading/multiprocessing).

6. **Fix the Race Condition**: Given an async function with a race condition, identify and fix it using appropriate async primitives.

## Summary: What We've Learned

1. **async/await is single-threaded concurrency** (not parallelism)
2. **Coroutines must be awaited** (or they do nothing)
3. **The event loop runs everything** (one loop per thread)
4. **asyncio.run() is the modern way** (handles loop lifecycle)
5. **Don't block the event loop** (no time.sleep, use await)
6. **gather() for concurrent execution** (wait for multiple tasks)
7. **The GIL prevents true parallelism** (use multiprocessing for CPU-bound)
8. **Async is for I/O bound operations** (network, disk, etc.)
9. **Don't mix sync and async** (pick one and stick with it)
10. **Async makes simple things complicated** (but complex things possible)

Next chapter: Package Management, where we'll explore pip, virtual environments, and the dependency hell that is Python packaging.

---

*"Python's async is like a fancy coffee machine: lots of buttons, makes weird noises, and in the end, you're not sure if it's actually faster than just making regular coffee."* - Developer who went back to threading