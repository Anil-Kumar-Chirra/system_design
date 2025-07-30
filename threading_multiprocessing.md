# Complete Guide to Threading and Multiprocessing in Python

## Table of Contents
1. [Basic Concepts](#basic-concepts)
2. [Sequential vs Concurrent Execution](#sequential-vs-concurrent-execution)
3. [Threading Fundamentals](#threading-fundamentals)
4. [Multiprocessing Fundamentals](#multiprocessing-fundamentals)
5. [When to Use What](#when-to-use-what)
6. [Advanced Concepts](#advanced-concepts)
7. [Best Practices](#best-practices)

---

## Basic Concepts

### What is Concurrency?
Concurrency is about dealing with multiple tasks at once. Think of it like a chef cooking multiple dishes - they switch between tasks efficiently.

### Key Terms
- **Process**: An independent program in execution (like separate applications)
- **Thread**: A lightweight sub-process within a process (like workers in a factory)
- **GIL**: Global Interpreter Lock - Python's mechanism that allows only one thread to execute Python code at a time

### Visual Representation
```
Single Process, Single Thread:
Task A → Task B → Task C → Task D
[====][====][====][====]

Multiple Threads (Concurrent):
Task A [====]    [====]
Task B    [====][====]
Task C       [====]    [====]

Multiple Processes (Parallel):
Process 1: Task A [====]
Process 2: Task B [====]
Process 3: Task C [====]
```

---

## Sequential vs Concurrent Execution

### Sequential Execution (Default)
```python
import time

def task(name, duration):
    print(f"Starting {name}")
    time.sleep(duration)
    print(f"Finished {name}")

# Sequential execution
start_time = time.time()
task("Task 1", 2)
task("Task 2", 2)
task("Task 3", 2)
print(f"Total time: {time.time() - start_time:.2f} seconds")
# Output: Total time: 6.00 seconds
```

---

## Threading Fundamentals

### What are Threads?
Threads share the same memory space within a process. They're ideal for I/O-bound tasks like file reading, network requests, or waiting operations.

### Basic Threading Example
```python
import threading
import time

def worker(name, duration):
    print(f"Worker {name} starting")
    time.sleep(duration)
    print(f"Worker {name} finished")

# Create and start threads
threads = []
start_time = time.time()

for i in range(3):
    thread = threading.Thread(target=worker, args=(f"Thread-{i+1}", 2))
    threads.append(thread)
    thread.start()

# Wait for all threads to complete
for thread in threads:
    thread.join()

print(f"All threads completed in: {time.time() - start_time:.2f} seconds")
# Output: All threads completed in: 2.00 seconds
```

### Thread Communication - Shared Data
```python
import threading
import time

# Shared variable (be careful with this!)
counter = 0

def increment():
    global counter
    for _ in range(100000):
        counter += 1

# Without proper synchronization (may give incorrect results)
threads = []
for _ in range(2):
    thread = threading.Thread(target=increment)
    threads.append(thread)
    thread.start()

for thread in threads:
    thread.join()

print(f"Counter value: {counter}")  # May not be 200000!
```

### Thread Synchronization with Lock
```python
import threading

counter = 0
lock = threading.Lock()

def safe_increment():
    global counter
    for _ in range(100000):
        with lock:  # Acquire lock
            counter += 1
        # Lock automatically released

threads = []
for _ in range(2):
    thread = threading.Thread(target=safe_increment)
    threads.append(thread)
    thread.start()

for thread in threads:
    thread.join()

print(f"Safe counter value: {counter}")  # Always 200000
```

### Producer-Consumer Pattern with Queue
```python
import threading
import queue
import time
import random

def producer(q, producer_id):
    for i in range(5):
        item = f"Item-{producer_id}-{i}"
        q.put(item)
        print(f"Produced: {item}")
        time.sleep(random.uniform(0.1, 0.5))

def consumer(q, consumer_id):
    while True:
        try:
            item = q.get(timeout=1)
            print(f"Consumer-{consumer_id} consumed: {item}")
            q.task_done()
            time.sleep(random.uniform(0.1, 0.3))
        except queue.Empty:
            break

# Create queue and threads
q = queue.Queue()

# Start producer
producer_thread = threading.Thread(target=producer, args=(q, 1))
producer_thread.start()

# Start consumers
consumers = []
for i in range(2):
    consumer_thread = threading.Thread(target=consumer, args=(q, i+1))
    consumers.append(consumer_thread)
    consumer_thread.start()

# Wait for completion
producer_thread.join()
q.join()  # Wait for all items to be processed
```

---

## Multiprocessing Fundamentals

### What is Multiprocessing?
Each process has its own memory space and Python interpreter. Perfect for CPU-intensive tasks that can benefit from true parallelism.

### Basic Multiprocessing Example
```python
import multiprocessing
import time

def cpu_intensive_task(n):
    """Simulate CPU-intensive work"""
    result = 0
    for i in range(n):
        result += i ** 2
    return result

if __name__ == "__main__":
    # Sequential execution
    start_time = time.time()
    results_sequential = []
    for i in range(4):
        result = cpu_intensive_task(1000000)
        results_sequential.append(result)
    sequential_time = time.time() - start_time
    
    # Parallel execution
    start_time = time.time()
    with multiprocessing.Pool() as pool:
        results_parallel = pool.map(cpu_intensive_task, [1000000] * 4)
    parallel_time = time.time() - start_time
    
    print(f"Sequential time: {sequential_time:.2f} seconds")
    print(f"Parallel time: {parallel_time:.2f} seconds")
    print(f"Speedup: {sequential_time/parallel_time:.2f}x")
```

### Process Communication - Shared Memory
```python
import multiprocessing
import time

def worker(shared_list, shared_value, lock, worker_id):
    for i in range(5):
        time.sleep(0.1)
        
        with lock:
            shared_value.value += 1
            shared_list.append(f"Worker-{worker_id}-Item-{i}")
            print(f"Worker {worker_id}: {shared_value.value}")

if __name__ == "__main__":
    # Create shared objects
    manager = multiprocessing.Manager()
    shared_list = manager.list()
    shared_value = multiprocessing.Value('i', 0)
    lock = multiprocessing.Lock()
    
    # Create processes
    processes = []
    for i in range(3):
        p = multiprocessing.Process(
            target=worker, 
            args=(shared_list, shared_value, lock, i+1)
        )
        processes.append(p)
        p.start()
    
    # Wait for all processes
    for p in processes:
        p.join()
    
    print(f"Final shared list: {list(shared_list)}")
    print(f"Final shared value: {shared_value.value}")
```

### Inter-Process Communication with Queue
```python
import multiprocessing
import time
import random

def producer(queue, producer_id):
    for i in range(5):
        item = f"Data-{producer_id}-{i}"
        queue.put(item)
        print(f"Produced: {item}")
        time.sleep(random.uniform(0.1, 0.3))

def consumer(queue, consumer_id):
    while True:
        try:
            item = queue.get(timeout=1)
            print(f"Consumer-{consumer_id} processing: {item}")
            time.sleep(random.uniform(0.2, 0.5))
        except:
            break

if __name__ == "__main__":
    queue = multiprocessing.Queue()
    
    # Start producer process
    producer_process = multiprocessing.Process(
        target=producer, args=(queue, 1)
    )
    producer_process.start()
    
    # Start consumer processes
    consumers = []
    for i in range(2):
        consumer_process = multiprocessing.Process(
            target=consumer, args=(queue, i+1)
        )
        consumers.append(consumer_process)
        consumer_process.start()
    
    # Wait for completion
    producer_process.join()
    for consumer in consumers:
        consumer.join()
```

---

## When to Use What

### Use Threading When:
```python
# I/O-bound tasks
import threading
import requests

def fetch_url(url):
    response = requests.get(url)
    return len(response.content)

urls = ["http://example.com"] * 10

# Threading is perfect for I/O-bound tasks
with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
    results = list(executor.map(fetch_url, urls))
```

### Use Multiprocessing When:
```python
# CPU-bound tasks
import multiprocessing

def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

numbers = [35, 36, 37, 38]

# Multiprocessing is perfect for CPU-bound tasks
if __name__ == "__main__":
    with multiprocessing.Pool() as pool:
        results = pool.map(fibonacci, numbers)
    print(results)
```

### Decision Matrix
| Task Type | GIL Impact | Memory | Communication | Use |
|-----------|------------|---------|---------------|-----|
| I/O-bound | Low | Shared | Easy | Threading |
| CPU-bound | High | Separate | Complex | Multiprocessing |
| Mixed | Medium | Depends | Varies | Async/Threading |

---

## Advanced Concepts

### ThreadPoolExecutor vs ProcessPoolExecutor
```python
import concurrent.futures
import time

def task(n):
    # Simulate work
    time.sleep(0.1)
    return n * n

numbers = list(range(20))

# ThreadPoolExecutor
with concurrent.futures.ThreadPoolExecutor(max_workers=4) as executor:
    start_time = time.time()
    thread_results = list(executor.map(task, numbers))
    thread_time = time.time() - start_time

# ProcessPoolExecutor
if __name__ == "__main__":
    with concurrent.futures.ProcessPoolExecutor(max_workers=4) as executor:
        start_time = time.time()
        process_results = list(executor.map(task, numbers))
        process_time = time.time() - start_time
    
    print(f"Thread time: {thread_time:.2f}s")
    print(f"Process time: {process_time:.2f}s")
```

### Async Programming (Alternative Approach)
```python
import asyncio
import aiohttp
import time

async def fetch_url(session, url):
    async with session.get(url) as response:
        return await response.text()

async def main():
    urls = ["http://httpbin.org/delay/1"] * 5
    
    start_time = time.time()
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url(session, url) for url in urls]
        results = await asyncio.gather(*tasks)
    
    print(f"Async time: {time.time() - start_time:.2f}s")

# Run with: asyncio.run(main())
```

### Daemon Threads
```python
import threading
import time

def daemon_worker():
    while True:
        print("Daemon working...")
        time.sleep(1)

def regular_worker():
    for i in range(3):
        print(f"Regular worker: {i}")
        time.sleep(1)

# Daemon thread (dies when main program exits)
daemon_thread = threading.Thread(target=daemon_worker)
daemon_thread.daemon = True
daemon_thread.start()

# Regular thread
regular_thread = threading.Thread(target=regular_worker)
regular_thread.start()
regular_thread.join()

print("Main program ending...")
# Daemon thread automatically terminates
```

---

## Best Practices

### 1. Always Use Context Managers
```python
# Good
with concurrent.futures.ThreadPoolExecutor() as executor:
    results = executor.map(function, data)

# Good
with multiprocessing.Pool() as pool:
    results = pool.map(function, data)
```

### 2. Handle Exceptions Properly
```python
import concurrent.futures

def risky_task(n):
    if n == 5:
        raise ValueError("Number 5 not allowed!")
    return n * 2

numbers = [1, 2, 3, 4, 5, 6]

with concurrent.futures.ThreadPoolExecutor() as executor:
    future_to_number = {
        executor.submit(risky_task, num): num 
        for num in numbers
    }
    
    for future in concurrent.futures.as_completed(future_to_number):
        number = future_to_number[future]
        try:
            result = future.result()
            print(f"Number {number} -> {result}")
        except Exception as exc:
            print(f"Number {number} generated exception: {exc}")
```

### 3. Choose Appropriate Number of Workers
```python
import multiprocessing
import concurrent.futures

# For CPU-bound tasks
cpu_workers = multiprocessing.cpu_count()

# For I/O-bound tasks (can be higher)
io_workers = min(32, (multiprocessing.cpu_count() or 1) + 4)

print(f"CPU cores: {cpu_workers}")
print(f"Recommended I/O workers: {io_workers}")
```

### 4. Memory Management
```python
# Process pools automatically manage memory
# But be aware of memory usage in long-running processes

def memory_intensive_task(data):
    # Process large data
    result = process_data(data)
    # Explicitly delete large objects if needed
    del data
    return result
```

### 5. Testing Concurrent Code
```python
import threading
import time

def test_thread_safety():
    shared_resource = []
    lock = threading.Lock()
    
    def append_numbers(start, end):
        for i in range(start, end):
            with lock:
                shared_resource.append(i)
    
    threads = []
    for i in range(0, 100, 20):
        thread = threading.Thread(
            target=append_numbers, 
            args=(i, i+20)
        )
        threads.append(thread)
        thread.start()
    
    for thread in threads:
        thread.join()
    
    # Verify results
    assert len(shared_resource) == 100
    assert set(shared_resource) == set(range(100))
    print("Thread safety test passed!")

test_thread_safety()
```

---

## Common Pitfalls and Solutions

### 1. Race Conditions
```python
# Problem: Race condition
counter = 0
def increment():
    global counter
    temp = counter
    temp += 1
    counter = temp

# Solution: Use locks or atomic operations
import threading
counter = 0
lock = threading.Lock()

def safe_increment():
    global counter
    with lock:
        counter += 1
```

### 2. Deadlocks
```python
# Problem: Potential deadlock
import threading

lock1 = threading.Lock()
lock2 = threading.Lock()

def worker1():
    with lock1:
        time.sleep(0.1)
        with lock2:
            pass

def worker2():
    with lock2:
        time.sleep(0.1)
        with lock1:
            pass

# Solution: Always acquire locks in the same order
def safe_worker1():
    with lock1:
        with lock2:
            pass

def safe_worker2():
    with lock1:  # Same order as worker1
        with lock2:
            pass
```

### 3. Resource Cleanup
```python
# Always clean up resources
import multiprocessing

def main():
    try:
        pool = multiprocessing.Pool(4)
        results = pool.map(function, data)
    finally:
        pool.close()
        pool.join()

# Better: Use context managers
def main():
    with multiprocessing.Pool(4) as pool:
        results = pool.map(function, data)
    # Automatic cleanup
```

---

## Performance Comparison Example

```python
import time
import threading
import multiprocessing
import concurrent.futures

def cpu_task(n):
    return sum(i*i for i in range(n))

def io_task(duration):
    time.sleep(duration)
    return f"Completed after {duration}s"

def benchmark():
    # CPU-intensive benchmark
    numbers = [100000] * 8
    durations = [0.5] * 8
    
    # Sequential
    start = time.time()
    [cpu_task(n) for n in numbers]
    sequential_cpu = time.time() - start
    
    # Threading (CPU-bound - limited by GIL)
    start = time.time()
    with concurrent.futures.ThreadPoolExecutor(4) as executor:
        list(executor.map(cpu_task, numbers))
    threading_cpu = time.time() - start
    
    # Multiprocessing (CPU-bound - true parallelism)
    start = time.time()
    with multiprocessing.Pool(4) as pool:
        pool.map(cpu_task, numbers)
    multiprocessing_cpu = time.time() - start
    
    # I/O-bound with threading
    start = time.time()
    with concurrent.futures.ThreadPoolExecutor(4) as executor:
        list(executor.map(io_task, durations))
    threading_io = time.time() - start
    
    print(f"CPU Task Results:")
    print(f"Sequential: {sequential_cpu:.2f}s")
    print(f"Threading: {threading_cpu:.2f}s")
    print(f"Multiprocessing: {multiprocessing_cpu:.2f}s")
    print(f"\nI/O Task (Threading): {threading_io:.2f}s")

if __name__ == "__main__":
    benchmark()
```

## Summary

- **Threading**: Best for I/O-bound tasks, limited by GIL for CPU tasks
- **Multiprocessing**: Best for CPU-bound tasks, true parallelism
- **Async**: Best for many concurrent I/O operations
- **Always**: Handle exceptions, clean up resources, test thoroughly

Choose the right tool based on your specific use case, and remember that premature optimization can lead to unnecessary complexity. Start simple and optimize when you have identified actual performance bottlenecks.
