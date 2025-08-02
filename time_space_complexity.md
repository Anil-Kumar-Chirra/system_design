# Time and Space Complexity Analysis

## Table of Contents
- [Introduction](#introduction)
- [Big O Notation](#big-o-notation)
- [Time Complexity](#time-complexity)
- [Space Complexity](#space-complexity)
- [Common Complexity Classes](#common-complexity-classes)
- [Data Structure Complexities](#data-structure-complexities)
- [Algorithm Analysis Examples](#algorithm-analysis-examples)
- [Optimization Techniques](#optimization-techniques)
- [Best and Worst Case Scenarios](#best-and-worst-case-scenarios)
- [Practical Applications](#practical-applications)

## Introduction

Time and space complexity analysis helps us understand how algorithms perform as input size grows. This knowledge is crucial for writing efficient code and making informed decisions about algorithm selection.

### Why Complexity Analysis Matters
- **Performance Prediction**: Estimate how algorithms scale with larger inputs
- **Resource Planning**: Understand memory and processing requirements
- **Algorithm Comparison**: Choose the best approach for specific constraints
- **System Design**: Build scalable applications

## Big O Notation

Big O notation describes the upper bound of algorithm complexity in terms of input size `n`.

### Common Big O Classes (Best to Worst)
```python
# O(1) - Constant Time
def get_first_element(arr):
    return arr[0] if arr else None

# O(log n) - Logarithmic Time
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1

# O(n) - Linear Time
def find_max(arr):
    if not arr:
        return None
    max_val = arr[0]
    for num in arr:
        if num > max_val:
            max_val = num
    return max_val

# O(n log n) - Linearithmic Time
def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    
    return merge(left, right)

def merge(left, right):
    result = []
    i = j = 0
    
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    
    result.extend(left[i:])
    result.extend(right[j:])
    return result

# O(n²) - Quadratic Time
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        for j in range(0, n - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
    return arr

# O(2^n) - Exponential Time
def fibonacci_naive(n):
    if n <= 1:
        return n
    return fibonacci_naive(n - 1) + fibonacci_naive(n - 2)

# O(n!) - Factorial Time
def generate_permutations(arr):
    if len(arr) <= 1:
        return [arr]
    
    result = []
    for i in range(len(arr)):
        element = arr[i]
        remaining = arr[:i] + arr[i+1:]
        for perm in generate_permutations(remaining):
            result.append([element] + perm)
    return result
```

## Time Complexity

Time complexity measures how execution time grows with input size.

### Basic Operations Analysis
```python
def analyze_basic_operations():
    """
    Demonstrating different time complexities
    """
    # O(1) - Constant time operations
    def constant_operations(arr):
        # Array access: O(1)
        first = arr[0]
        
        # Dictionary access: O(1) average
        d = {'key': 'value'}
        value = d['key']
        
        # Arithmetic operations: O(1)
        result = 10 + 20 * 30
        
        return first, value, result
    
    # O(n) - Linear time operations
    def linear_operations(arr):
        # Single loop: O(n)
        total = 0
        for num in arr:
            total += num
        
        # List creation: O(n)
        doubled = [x * 2 for x in arr]
        
        # String operations: O(n)
        text = "hello"
        reversed_text = text[::-1]
        
        return total, doubled, reversed_text
    
    # O(n²) - Quadratic time operations
    def quadratic_operations(arr):
        # Nested loops: O(n²)
        pairs = []
        for i in range(len(arr)):
            for j in range(len(arr)):
                if i != j:
                    pairs.append((arr[i], arr[j]))
        
        return pairs

# Example usage and timing
import time

def measure_time_complexity():
    sizes = [100, 1000, 10000]
    
    for n in sizes:
        arr = list(range(n))
        
        # O(n) operation
        start = time.time()
        sum(arr)
        linear_time = time.time() - start
        
        # O(n²) operation
        start = time.time()
        bubble_sort(arr.copy())
        quadratic_time = time.time() - start
        
        print(f"n={n}: Linear={linear_time:.6f}s, Quadratic={quadratic_time:.6f}s")
```

### Loop Analysis
```python
def analyze_loops():
    """
    Different types of loops and their complexities
    """
    
    # Single loop: O(n)
    def single_loop(n):
        total = 0
        for i in range(n):
            total += i
        return total
    
    # Nested loops: O(n²)
    def nested_loops(n):
        total = 0
        for i in range(n):
            for j in range(n):
                total += i * j
        return total
    
    # Triple nested loops: O(n³)
    def triple_nested_loops(n):
        total = 0
        for i in range(n):
            for j in range(n):
                for k in range(n):
                    total += i * j * k
        return total
    
    # Loop with logarithmic reduction: O(log n)
    def logarithmic_loop(n):
        total = 0
        i = n
        while i > 0:
            total += i
            i //= 2
        return total
    
    # Loop with linear and logarithmic: O(n log n)
    def n_log_n_loop(arr):
        result = []
        for i in range(len(arr)):  # O(n)
            # Binary search on each element: O(log n)
            index = binary_search(sorted(arr), arr[i])
            result.append(index)
        return result

# Example analysis
def demonstrate_loop_complexities():
    n = 1000
    
    print("Time Complexity Demonstrations:")
    print(f"O(n): {single_loop(n)}")
    print(f"O(log n): {logarithmic_loop(n)}")
    print(f"O(n²): First few from nested loops")
```

### Recursive Algorithm Analysis
```python
def analyze_recursive_algorithms():
    """
    Analyzing time complexity of recursive algorithms
    """
    
    # T(n) = T(n-1) + O(1) = O(n)
    def factorial_recursive(n):
        if n <= 1:
            return 1
        return n * factorial_recursive(n - 1)
    
    # T(n) = 2T(n-1) + O(1) = O(2^n)
    def fibonacci_exponential(n):
        if n <= 1:
            return n
        return fibonacci_exponential(n - 1) + fibonacci_exponential(n - 2)
    
    # T(n) = T(n-1) + O(1) = O(n) with memoization
    def fibonacci_memoized(n, memo={}):
        if n in memo:
            return memo[n]
        if n <= 1:
            return n
        memo[n] = fibonacci_memoized(n - 1, memo) + fibonacci_memoized(n - 2, memo)
        return memo[n]
    
    # T(n) = 2T(n/2) + O(n) = O(n log n)
    def merge_sort_analysis(arr):
        if len(arr) <= 1:
            return arr
        
        mid = len(arr) // 2
        left = merge_sort_analysis(arr[:mid])
        right = merge_sort_analysis(arr[mid:])
        
        # Merge operation: O(n)
        return merge_two_arrays(left, right)
    
    def merge_two_arrays(left, right):
        result = []
        i = j = 0
        
        while i < len(left) and j < len(right):
            if left[i] <= right[j]:
                result.append(left[i])
                i += 1
            else:
                result.append(right[j])
                j += 1
        
        result.extend(left[i:])
        result.extend(right[j:])
        return result
    
    # T(n) = T(n/2) + O(1) = O(log n)
    def binary_search_recursive(arr, target, left, right):
        if left > right:
            return -1
        
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            return binary_search_recursive(arr, target, mid + 1, right)
        else:
            return binary_search_recursive(arr, target, left, mid - 1)

# Demonstration of recursive complexities
def compare_fibonacci_implementations():
    """Compare different fibonacci implementations"""
    
    def time_function(func, n):
        start = time.time()
        result = func(n)
        end = time.time()
        return result, end - start
    
    n = 30
    
    # O(2^n) - Exponential
    result1, time1 = time_function(fibonacci_exponential, n)
    print(f"Exponential Fibonacci({n}): {result1}, Time: {time1:.4f}s")
    
    # O(n) - Linear with memoization
    result2, time2 = time_function(fibonacci_memoized, n)
    print(f"Memoized Fibonacci({n}): {result2}, Time: {time2:.6f}s")
```

## Space Complexity

Space complexity measures how memory usage grows with input size.

### Types of Space Usage
```python
def analyze_space_complexity():
    """
    Different types of space complexity patterns
    """
    
    # O(1) - Constant space
    def constant_space_sum(arr):
        total = 0  # O(1) space
        for num in arr:
            total += num
        return total
    
    # O(n) - Linear space
    def linear_space_copy(arr):
        copy = []  # O(n) space
        for item in arr:
            copy.append(item)
        return copy
    
    # O(n) - Linear space with recursion stack
    def recursive_sum(arr, index=0):
        if index >= len(arr):
            return 0
        # Each recursive call uses O(1) space
        # Total depth is n, so O(n) space
        return arr[index] + recursive_sum(arr, index + 1)
    
    # O(log n) - Logarithmic space
    def binary_search_space(arr, target, left=0, right=None):
        if right is None:
            right = len(arr) - 1
        
        if left > right:
            return -1
        
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            # Recursion depth is log n
            return binary_search_space(arr, target, mid + 1, right)
        else:
            return binary_search_space(arr, target, left, mid - 1)
    
    # O(n²) - Quadratic space
    def create_multiplication_table(n):
        table = []  # O(n²) space
        for i in range(n):
            row = []
            for j in range(n):
                row.append(i * j)
            table.append(row)
        return table
    
    # O(2^n) - Exponential space (worst case)
    def generate_all_subsets(arr):
        if not arr:
            return [[]]
        
        first = arr[0]
        rest_subsets = generate_all_subsets(arr[1:])
        
        # Double the number of subsets
        new_subsets = []
        for subset in rest_subsets:
            new_subsets.append(subset)
            new_subsets.append([first] + subset)
        
        return new_subsets

# Auxiliary space vs Total space
def demonstrate_space_types():
    """
    Demonstrate different types of space usage
    """
    
    # Auxiliary space: O(1), Total space: O(n)
    def in_place_reverse(arr):
        left, right = 0, len(arr) - 1
        while left < right:
            arr[left], arr[right] = arr[right], arr[left]
            left += 1
            right -= 1
        return arr
    
    # Auxiliary space: O(n), Total space: O(n)
    def out_of_place_reverse(arr):
        return arr[::-1]
    
    # Space-time tradeoff example
    def fibonacci_space_optimized(n):
        if n <= 1:
            return n
        
        # O(1) space instead of O(n) with memoization
        prev2, prev1 = 0, 1
        for i in range(2, n + 1):
            current = prev1 + prev2
            prev2, prev1 = prev1, current
        
        return prev1

# Memory usage demonstration
def track_memory_usage():
    """
    Practical memory usage tracking
    """
    import sys
    
    # Different data structures and their memory usage
    def compare_data_structures():
        n = 10000
        
        # List: O(n) space
        list_data = list(range(n))
        list_size = sys.getsizeof(list_data)
        
        # Set: O(n) space
        set_data = set(range(n))
        set_size = sys.getsizeof(set_data)
        
        # Dictionary: O(n) space
        dict_data = {i: i for i in range(n)}
        dict_size = sys.getsizeof(dict_data)
        
        print(f"Memory usage for {n} elements:")
        print(f"List: {list_size} bytes")
        print(f"Set: {set_size} bytes")
        print(f"Dictionary: {dict_size} bytes")
        
        return {
            'list': list_size,
            'set': set_size,
            'dict': dict_size
        }
```

## Common Complexity Classes

### Detailed Analysis with Examples
```python
def complexity_class_examples():
    """
    Comprehensive examples for each complexity class
    """
    
    # O(1) - Constant Time Examples
    class ConstantTimeOperations:
        def __init__(self):
            self.data = {}
            self.stack = []
        
        def hash_table_access(self, key):
            """Hash table access: O(1)"""
            return self.data.get(key)
        
        def stack_operations(self, value=None):
            """Stack push/pop: O(1)"""
            if value is not None:
                self.stack.append(value)  # Push
            return self.stack.pop() if self.stack else None  # Pop
        
        def array_access(self, arr, index):
            """Array element access: O(1)"""
            return arr[index] if 0 <= index < len(arr) else None
    
    # O(log n) - Logarithmic Time Examples
    class LogarithmicOperations:
        
        def binary_search_iterative(self, arr, target):
            """Binary search: O(log n)"""
            left, right = 0, len(arr) - 1
            
            while left <= right:
                mid = (left + right) // 2
                if arr[mid] == target:
                    return mid
                elif arr[mid] < target:
                    left = mid + 1
                else:
                    right = mid - 1
            return -1
        
        def find_height_binary_tree(self, root):
            """Binary tree height: O(log n) for balanced tree"""
            if not root:
                return 0
            return 1 + max(
                self.find_height_binary_tree(root.left),
                self.find_height_binary_tree(root.right)
            )
        
        def power_calculation(self, base, exp):
            """Fast exponentiation: O(log n)"""
            if exp == 0:
                return 1
            if exp % 2 == 0:
                half_power = self.power_calculation(base, exp // 2)
                return half_power * half_power
            else:
                return base * self.power_calculation(base, exp - 1)
    
    # O(n) - Linear Time Examples
    class LinearOperations:
        
        def linear_search(self, arr, target):
            """Linear search: O(n)"""
            for i, value in enumerate(arr):
                if value == target:
                    return i
            return -1
        
        def find_min_max(self, arr):
            """Find min and max: O(n)"""
            if not arr:
                return None, None
            
            min_val = max_val = arr[0]
            for num in arr[1:]:
                if num < min_val:
                    min_val = num
                if num > max_val:
                    max_val = num
            
            return min_val, max_val
        
        def count_frequency(self, arr):
            """Count element frequencies: O(n)"""
            frequency = {}
            for item in arr:
                frequency[item] = frequency.get(item, 0) + 1
            return frequency
    
    # O(n log n) - Linearithmic Time Examples
    class LinearithmicOperations:
        
        def merge_sort_implementation(self, arr):
            """Merge sort: O(n log n)"""
            if len(arr) <= 1:
                return arr
            
            mid = len(arr) // 2
            left = self.merge_sort_implementation(arr[:mid])
            right = self.merge_sort_implementation(arr[mid:])
            
            return self._merge(left, right)
        
        def _merge(self, left, right):
            result = []
            i = j = 0
            
            while i < len(left) and j < len(right):
                if left[i] <= right[j]:
                    result.append(left[i])
                    i += 1
                else:
                    result.append(right[j])
                    j += 1
            
            result.extend(left[i:])
            result.extend(right[j:])
            return result
        
        def heap_sort(self, arr):
            """Heap sort: O(n log n)"""
            import heapq
            return [heapq.heappop(arr) for _ in range(len(arr))]
    
    # O(n²) - Quadratic Time Examples
    class QuadraticOperations:
        
        def bubble_sort_implementation(self, arr):
            """Bubble sort: O(n²)"""
            n = len(arr)
            for i in range(n):
                for j in range(0, n - i - 1):
                    if arr[j] > arr[j + 1]:
                        arr[j], arr[j + 1] = arr[j + 1], arr[j]
            return arr
        
        def selection_sort(self, arr):
            """Selection sort: O(n²)"""
            for i in range(len(arr)):
                min_idx = i
                for j in range(i + 1, len(arr)):
                    if arr[j] < arr[min_idx]:
                        min_idx = j
                arr[i], arr[min_idx] = arr[min_idx], arr[i]
            return arr
        
        def find_all_pairs(self, arr):
            """Find all pairs: O(n²)"""
            pairs = []
            for i in range(len(arr)):
                for j in range(i + 1, len(arr)):
                    pairs.append((arr[i], arr[j]))
            return pairs
    
    # O(2^n) - Exponential Time Examples
    class ExponentialOperations:
        
        def fibonacci_recursive_naive(self, n):
            """Naive fibonacci: O(2^n)"""
            if n <= 1:
                return n
            return self.fibonacci_recursive_naive(n - 1) + self.fibonacci_recursive_naive(n - 2)
        
        def power_set(self, arr):
            """Generate power set: O(2^n)"""
            if not arr:
                return [[]]
            
            first = arr[0]
            rest_power_set = self.power_set(arr[1:])
            
            result = []
            for subset in rest_power_set:
                result.append(subset)
                result.append([first] + subset)
            
            return result
        
        def tower_of_hanoi(self, n, source, destination, auxiliary):
            """Tower of Hanoi: O(2^n)"""
            if n == 1:
                return [(source, destination)]
            
            moves = []
            moves.extend(self.tower_of_hanoi(n - 1, source, auxiliary, destination))
            moves.append((source, destination))
            moves.extend(self.tower_of_hanoi(n - 1, auxiliary, destination, source))
            
            return moves

# Performance comparison demonstration
def demonstrate_complexity_differences():
    """
    Compare different algorithms with same functionality
    """
    import time
    import random
    
    def time_algorithm(func, *args):
        start = time.time()
        result = func(*args)
        end = time.time()
        return result, end - start
    
    # Generate test data
    sizes = [100, 1000, 5000]
    
    for n in sizes:
        arr = [random.randint(1, 1000) for _ in range(n)]
        target = arr[n // 2]  # Target exists in array
        
        # Linear search vs Binary search (on sorted array)
        sorted_arr = sorted(arr)
        
        linear_ops = LinearOperations()
        log_ops = LogarithmicOperations()
        
        _, linear_time = time_algorithm(linear_ops.linear_search, arr, target)
        _, binary_time = time_algorithm(log_ops.binary_search_iterative, sorted_arr, target)
        
        print(f"n={n}: Linear Search={linear_time:.6f}s, Binary Search={binary_time:.6f}s")
```

## Data Structure Complexities

### Comprehensive Data Structure Analysis
```python
def analyze_data_structure_complexities():
    """
    Time and space complexities for common data structures
    """
    
    # Array/List Operations
    class ArrayOperations:
        def __init__(self):
            self.arr = []
        
        def access(self, index):
            """Access by index: O(1)"""
            return self.arr[index] if 0 <= index < len(self.arr) else None
        
        def search(self, value):
            """Search value: O(n)"""
            for i, val in enumerate(self.arr):
                if val == value:
                    return i
            return -1
        
        def insert_end(self, value):
            """Insert at end: O(1) amortized"""
            self.arr.append(value)
        
        def insert_beginning(self, value):
            """Insert at beginning: O(n)"""
            self.arr.insert(0, value)
        
        def delete_end(self):
            """Delete from end: O(1)"""
            return self.arr.pop() if self.arr else None
        
        def delete_beginning(self):
            """Delete from beginning: O(n)"""
            return self.arr.pop(0) if self.arr else None
    
    # Linked List Operations
    class LinkedListNode:
        def __init__(self, data):
            self.data = data
            self.next = None
    
    class LinkedListOperations:
        def __init__(self):
            self.head = None
            self.size = 0
        
        def insert_beginning(self, data):
            """Insert at beginning: O(1)"""
            new_node = LinkedListNode(data)
            new_node.next = self.head
            self.head = new_node
            self.size += 1
        
        def insert_end(self, data):
            """Insert at end: O(n)"""
            new_node = LinkedListNode(data)
            if not self.head:
                self.head = new_node
            else:
                current = self.head
                while current.next:
                    current = current.next
                current.next = new_node
            self.size += 1
        
        def search(self, data):
            """Search: O(n)"""
            current = self.head
            index = 0
            while current:
                if current.data == data:
                    return index
                current = current.next
                index += 1
            return -1
        
        def delete(self, data):
            """Delete: O(n)"""
            if not self.head:
                return False
            
            if self.head.data == data:
                self.head = self.head.next
                self.size -= 1
                return True
            
            current = self.head
            while current.next:
                if current.next.data == data:
                    current.next = current.next.next
                    self.size -= 1
                    return True
                current = current.next
            return False
    
    # Hash Table Operations
    class HashTableOperations:
        def __init__(self, size=10):
            self.size = size
            self.table = [[] for _ in range(size)]
        
        def _hash(self, key):
            return hash(key) % self.size
        
        def insert(self, key, value):
            """Insert: O(1) average, O(n) worst case"""
            index = self._hash(key)
            bucket = self.table[index]
            
            for i, (k, v) in enumerate(bucket):
                if k == key:
                    bucket[i] = (key, value)
                    return
            
            bucket.append((key, value))
        
        def search(self, key):
            """Search: O(1) average, O(n) worst case"""
            index = self._hash(key)
            bucket = self.table[index]
            
            for k, v in bucket:
                if k == key:
                    return v
            return None
        
        def delete(self, key):
            """Delete: O(1) average, O(n) worst case"""
            index = self._hash(key)
            bucket = self.table[index]
            
            for i, (k, v) in enumerate(bucket):
                if k == key:
                    del bucket[i]
                    return True
            return False
    
    # Binary Search Tree Operations
    class BSTNode:
        def __init__(self, data):
            self.data = data
            self.left = None
            self.right = None
    
    class BinarySearchTreeOperations:
        def __init__(self):
            self.root = None
        
        def insert(self, data):
            """Insert: O(log n) average, O(n) worst case"""
            self.root = self._insert_recursive(self.root, data)
        
        def _insert_recursive(self, node, data):
            if not node:
                return BSTNode(data)
            
            if data < node.data:
                node.left = self._insert_recursive(node.left, data)
            else:
                node.right = self._insert_recursive(node.right, data)
            
            return node
        
        def search(self, data):
            """Search: O(log n) average, O(n) worst case"""
            return self._search_recursive(self.root, data)
        
        def _search_recursive(self, node, data):
            if not node or node.data == data:
                return node
            
            if data < node.data:
                return self._search_recursive(node.left, data)
            else:
                return self._search_recursive(node.right, data)
        
        def delete(self, data):
            """Delete: O(log n) average, O(n) worst case"""
            self.root = self._delete_recursive(self.root, data)
        
        def _delete_recursive(self, node, data):
            if not node:
                return node
            
            if data < node.data:
                node.left = self._delete_recursive(node.left, data)
            elif data > node.data:
                node.right = self._delete_recursive(node.right, data)
            else:
                # Node to be deleted found
                if not node.left:
                    return node.right
                elif not node.right:
                    return node.left
                
                # Node with two children
                min_node = self._find_min(node.right)
                node.data = min_node.data
                node.right = self._delete_recursive(node.right, min_node.data)
            
            return node
        
        def _find_min(self, node):
            while node.left:
                node = node.left
            return node
    
    # Heap Operations
    class MinHeapOperations:
        def __init__(self):
            self.heap = []
        
        def insert(self, value):
            """Insert: O(log n)"""
            self.heap.append(value)
            self._heapify_up(len(self.heap) - 1)
        
        def extract_min(self):
            """Extract minimum: O(log n)"""
            if not self.heap:
                return None
            
            if len(self.heap) == 1:
                return self.heap.pop()
            
            min_val = self.heap[0]
            self.heap[0] = self.heap.pop()
            self._heapify_down(0)
            return min_val
        
        def peek(self):
            """Peek minimum: O(1)"""
            return self.heap[0] if self.heap else None
        
        def _heapify_up(self, index):
            parent = (index - 1) // 2
            if index > 0 and self.heap[index] < self.heap[parent]:
                self.heap[index], self.heap[parent] = self.heap[parent], self.heap[index]
                self._heapify_up(parent)
        
        def _heapify_down(self, index):
            left_child = 2 * index + 1
            right_child = 2 * index + 2
            smallest = index
            
            if left_child < len(self.heap) and self.heap[left_child] < self.heap[smallest]:
                smallest = left_child
            
            if right_child < len(self.heap) and self.heap[right_child] < self.heap[smallest]:
                smallest = right_child
            
            if smallest != index:
                self.heap[index], self.heap[smallest] = self.heap[smallest], self.heap[index]
                self._heapify_down(smallest)

# Data Structure Complexity Summary
def data_structure_complexity_summary():
    """
    Comprehensive complexity summary for all data structures
    """
    
    complexity_table = {
        "Array/List": {
            "Access": "O(1)",
            "Search": "O(n)",
            "Insertion": "O(1) end, O(n) beginning",
            "Deletion": "O(1) end, O(n) beginning",
            "Space": "O(n)"
        },
        "Linked List": {
            "Access": "O(n)",
            "Search": "O(n)",
            "Insertion": "O(1) beginning, O(n) end",
            "Deletion": "O(1) if node known, O(n) search",
            "Space": "O(n)"
        },
        "Hash Table": {
            "Access": "N/A",
            "Search": "O(1) average, O(n) worst",
            "Insertion": "O(1) average, O(n) worst",
            "Deletion": "O(1) average, O(n) worst",
            "Space": "O(n)"
        },
        "Binary Search Tree": {
            "Access": "O(log n) average, O(n) worst",
            "Search": "O(log n) average, O(n) worst",
            "Insertion": "O(log n) average, O(n) worst",
            "Deletion": "O(log n) average, O(n) worst",
            "Space": "O(n)"
        },
        "Heap": {
            "Access": "O(1) min/max",
            "Search": "O(n)",
            "Insertion": "O(log n)",
            "Deletion": "O(log n) extract min/max",
            "Space": "O(n)"
        }
    }
    
    return complexity_table

## Algorithm Analysis Examples

### Sorting Algorithms Comparison
```python
def sorting_algorithms_analysis():
    """
    Compare different sorting algorithms and their complexities
    """
    
    # Bubble Sort: O(n²) time, O(1) space
    def bubble_sort(arr):
        n = len(arr)
        for i in range(n):
            swapped = False
            for j in range(0, n - i - 1):
                if arr[j] > arr[j + 1]:
                    arr[j], arr[j + 1] = arr[j + 1], arr[j]
                    swapped = True
            if not swapped:  # Early termination optimization
                break
        return arr
    
    # Selection Sort: O(n²) time, O(1) space
    def selection_sort(arr):
        for i in range(len(arr)):
            min_idx = i
            for j in range(i + 1, len(arr)):
                if arr[j] < arr[min_idx]:
                    min_idx = j
            arr[i], arr[min_idx] = arr[min_idx], arr[i]
        return arr
    
    # Insertion Sort: O(n²) worst, O(n) best, O(1) space
    def insertion_sort(arr):
        for i in range(1, len(arr)):
            key = arr[i]
            j = i - 1
            while j >= 0 and arr[j] > key:
                arr[j + 1] = arr[j]
                j -= 1
            arr[j + 1] = key
        return arr
    
    # Merge Sort: O(n log n) time, O(n) space
    def merge_sort(arr):
        if len(arr) <= 1:
            return arr
        
        mid = len(arr) // 2
        left = merge_sort(arr[:mid])
        right = merge_sort(arr[mid:])
        
        return merge_sorted_arrays(left, right)
    
    def merge_sorted_arrays(left, right):
        result = []
        i = j = 0
        
        while i < len(left) and j < len(right):
            if left[i] <= right[j]:
                result.append(left[i])
                i += 1
            else:
                result.append(right[j])
                j += 1
        
        result.extend(left[i:])
        result.extend(right[j:])
        return result
    
    # Quick Sort: O(n log n) average, O(n²) worst, O(log n) space
    def quick_sort(arr):
        if len(arr) <= 1:
            return arr
        
        pivot = arr[len(arr) // 2]
        left = [x for x in arr if x < pivot]
        middle = [x for x in arr if x == pivot]
        right = [x for x in arr if x > pivot]
        
        return quick_sort(left) + middle + quick_sort(right)
    
    # Heap Sort: O(n log n) time, O(1) space
    def heap_sort(arr):
        def heapify(arr, n, i):
            largest = i
            left = 2 * i + 1
            right = 2 * i + 2
            
            if left < n and arr[left] > arr[largest]:
                largest = left
            
            if right < n and arr[right] > arr[largest]:
                largest = right
            
            if largest != i:
                arr[i], arr[largest] = arr[largest], arr[i]
                heapify(arr, n, largest)
        
        n = len(arr)
        
        # Build max heap
        for i in range(n // 2 - 1, -1, -1):
            heapify(arr, n, i)
        
        # Extract elements from heap
        for i in range(n - 1, 0, -1):
            arr[0], arr[i] = arr[i], arr[0]
            heapify(arr, i, 0)
        
        return arr
    
    # Counting Sort: O(n + k) time, O(k) space where k is range
    def counting_sort(arr, k=None):
        if not arr:
            return arr
        
        if k is None:
            k = max(arr) + 1
        
        count = [0] * k
        output = [0] * len(arr)
        
        # Count occurrences
        for num in arr:
            count[num] += 1
        
        # Transform count array
        for i in range(1, k):
            count[i] += count[i - 1]
        
        # Build output array
        for i in range(len(arr) - 1, -1, -1):
            output[count[arr[i]] - 1] = arr[i]
            count[arr[i]] -= 1
        
        return output

# Sorting Algorithm Complexity Summary
sorting_complexities = {
    "Bubble Sort": {"Time": "O(n²)", "Space": "O(1)", "Stable": "Yes"},
    "Selection Sort": {"Time": "O(n²)", "Space": "O(1)", "Stable": "No"},
    "Insertion Sort": {"Time": "O(n²)", "Space": "O(1)", "Stable": "Yes"},
    "Merge Sort": {"Time": "O(n log n)", "Space": "O(n)", "Stable": "Yes"},
    "Quick Sort": {"Time": "O(n log n) avg, O(n²) worst", "Space": "O(log n)", "Stable": "No"},
    "Heap Sort": {"Time": "O(n log n)", "Space": "O(1)", "Stable": "No"},
    "Counting Sort": {"Time": "O(n + k)", "Space": "O(k)", "Stable": "Yes"}
}
```

### Search Algorithms Analysis
```python
def search_algorithms_analysis():
    """
    Compare different search algorithms and their complexities
    """
    
    # Linear Search: O(n) time, O(1) space
    def linear_search(arr, target):
        for i, value in enumerate(arr):
            if value == target:
                return i
        return -1
    
    # Binary Search: O(log n) time, O(1) space iterative
    def binary_search_iterative(arr, target):
        left, right = 0, len(arr) - 1
        
        while left <= right:
            mid = (left + right) // 2
            if arr[mid] == target:
                return mid
            elif arr[mid] < target:
                left = mid + 1
            else:
                right = mid - 1
        
        return -1
    
    # Binary Search: O(log n) time, O(log n) space recursive
    def binary_search_recursive(arr, target, left=0, right=None):
        if right is None:
            right = len(arr) - 1
        
        if left > right:
            return -1
        
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            return binary_search_recursive(arr, target, mid + 1, right)
        else:
            return binary_search_recursive(arr, target, left, mid - 1)
    
    # Jump Search: O(√n) time, O(1) space
    def jump_search(arr, target):
        import math
        n = len(arr)
        step = int(math.sqrt(n))
        prev = 0
        
        # Jump to find the block where element may be present
        while arr[min(step, n) - 1] < target:
            prev = step
            step += int(math.sqrt(n))
            if prev >= n:
                return -1
        
        # Linear search in the identified block
        while arr[prev] < target:
            prev += 1
            if prev == min(step, n):
                return -1
        
        if arr[prev] == target:
            return prev
        
        return -1
    
    # Interpolation Search: O(log log n) average, O(n) worst
    def interpolation_search(arr, target):
        left, right = 0, len(arr) - 1
        
        while left <= right and target >= arr[left] and target <= arr[right]:
            if left == right:
                if arr[left] == target:
                    return left
                return -1
            
            # Calculate position using interpolation formula
            pos = left + ((target - arr[left]) * (right - left)) // (arr[right] - arr[left])
            
            if arr[pos] == target:
                return pos
            elif arr[pos] < target:
                left = pos + 1
            else:
                right = pos - 1
        
        return -1
    
    # Exponential Search: O(log n) time, O(1) space
    def exponential_search(arr, target):
        if arr[0] == target:
            return 0
        
        # Find range for binary search
        i = 1
        while i < len(arr) and arr[i] <= target:
            i *= 2
        
        # Binary search in found range
        return binary_search_iterative(arr[i//2:min(i, len(arr))], target)

# Search Algorithm Complexity Summary
search_complexities = {
    "Linear Search": {"Time": "O(n)", "Space": "O(1)", "Requirement": "None"},
    "Binary Search": {"Time": "O(log n)", "Space": "O(1) iterative, O(log n) recursive", "Requirement": "Sorted array"},
    "Jump Search": {"Time": "O(√n)", "Space": "O(1)", "Requirement": "Sorted array"},
    "Interpolation Search": {"Time": "O(log log n) avg, O(n) worst", "Space": "O(1)", "Requirement": "Sorted, uniformly distributed"},
    "Exponential Search": {"Time": "O(log n)", "Space": "O(1)", "Requirement": "Sorted array"}
}
```

## Optimization Techniques

### Common Optimization Strategies
```python
def optimization_techniques():
    """
    Demonstrate various optimization techniques and their impact on complexity
    """
    
    # 1. Memoization - Trading Space for Time
    def fibonacci_optimizations():
        # O(2^n) - Exponential time
        def fibonacci_naive(n):
            if n <= 1:
                return n
            return fibonacci_naive(n - 1) + fibonacci_naive(n - 2)
        
        # O(n) time, O(n) space - Memoization
        def fibonacci_memoized(n, memo={}):
            if n in memo:
                return memo[n]
            if n <= 1:
                return n
            memo[n] = fibonacci_memoized(n - 1, memo) + fibonacci_memoized(n - 2, memo)
            return memo[n]
        
        # O(n) time, O(1) space - Space optimization
        def fibonacci_optimized(n):
            if n <= 1:
                return n
            prev2, prev1 = 0, 1
            for i in range(2, n + 1):
                current = prev1 + prev2
                prev2, prev1 = prev1, current
            return prev1
        
        return fibonacci_naive, fibonacci_memoized, fibonacci_optimized
    
    # 2. Two Pointers Technique
    def two_pointers_examples():
        # Two Sum in sorted array: O(n) instead of O(n²)
        def two_sum_sorted(arr, target):
            left, right = 0, len(arr) - 1
            
            while left < right:
                current_sum = arr[left] + arr[right]
                if current_sum == target:
                    return [left, right]
                elif current_sum < target:
                    left += 1
                else:
                    right -= 1
            
            return [-1, -1]
        
        # Remove duplicates from sorted array: O(n) time, O(1) space
        def remove_duplicates(arr):
            if not arr:
                return 0
            
            write_index = 1
            for read_index in range(1, len(arr)):
                if arr[read_index] != arr[read_index - 1]:
                    arr[write_index] = arr[read_index]
                    write_index += 1
            
            return write_index
        
        # Palindrome check: O(n) time, O(1) space
        def is_palindrome(s):
            left, right = 0, len(s) - 1
            
            while left < right:
                if s[left] != s[right]:
                    return False
                left += 1
                right -= 1
            
            return True
        
        return two_sum_sorted, remove_duplicates, is_palindrome
    
    # 3. Sliding Window Technique
    def sliding_window_examples():
        # Maximum sum subarray of size k: O(n) instead of O(n*k)
        def max_sum_subarray(arr, k):
            if len(arr) < k:
                return None
            
            # Calculate sum of first window
            window_sum = sum(arr[:k])
            max_sum = window_sum
            
            # Slide the window
            for i in range(k, len(arr)):
                window_sum = window_sum - arr[i - k] + arr[i]
                max_sum = max(max_sum, window_sum)
            
            return max_sum
        
        # Longest substring without repeating characters: O(n)
        def longest_unique_substring(s):
            char_index = {}
            max_length = 0
            start = 0
            
            for end, char in enumerate(s):
                if char in char_index and char_index[char] >= start:
                    start = char_index[char] + 1
                
                char_index[char] = end
                max_length = max(max_length, end - start + 1)
            
            return max_length
        
        return max_sum_subarray, longest_unique_substring
    
    # 4. Hash Table for O(1) Lookups
    def hash_table_optimizations():
        # Two Sum: O(n) instead of O(n²)
        def two_sum_hash(arr, target):
            num_index = {}
            
            for i, num in enumerate(arr):
                complement = target - num
                if complement in num_index:
                    return [num_index[complement], i]
                num_index[num] = i
            
            return [-1, -1]
        
        # Find duplicates: O(n) time, O(n) space
        def find_duplicates(arr):
            seen = set()
            duplicates = set()
            
            for num in arr:
                if num in seen:
                    duplicates.add(num)
                seen.add(num)
            
            return list(duplicates)
        
        # Group anagrams: O(n*m log m) where m is average string length
        def group_anagrams(strs):
            from collections import defaultdict
            
            groups = defaultdict(list)
            
            for s in strs:
                # Sort characters to create key
                key = ''.join(sorted(s))
                groups[key].append(s)
            
            return list(groups.values())
        
        return two_sum_hash, find_duplicates, group_anagrams
    
    # 5. Divide and Conquer Optimization
    def divide_conquer_examples():
        # Maximum subarray sum: O(n log n) Divide & Conquer vs O(n) Kadane's
        def max_subarray_divide_conquer(arr, left=0, right=None):
            if right is None:
                right = len(arr) - 1
            
            if left == right:
                return arr[left]
            
            mid = (left + right) // 2
            
            # Maximum sum in left half
            left_sum = max_subarray_divide_conquer(arr, left, mid)
            
            # Maximum sum in right half
            right_sum = max_subarray_divide_conquer(arr, mid + 1, right)
            
            # Maximum sum crossing the middle
            cross_sum = max_crossing_sum(arr, left, mid, right)
            
            return max(left_sum, right_sum, cross_sum)
        
        def max_crossing_sum(arr, left, mid, right):
            left_sum = float('-inf')
            total = 0
            
            for i in range(mid, left - 1, -1):
                total += arr[i]
                left_sum = max(left_sum, total)
            
            right_sum = float('-inf')
            total = 0
            
            for i in range(mid + 1, right + 1):
                total += arr[i]
                right_sum = max(right_sum, total)
            
            return left_sum + right_sum
        
        # Kadane's Algorithm: O(n) - more efficient
        def max_subarray_kadane(arr):
            max_ending_here = max_so_far = arr[0]
            
            for i in range(1, len(arr)):
                max_ending_here = max(arr[i], max_ending_here + arr[i])
                max_so_far = max(max_so_far, max_ending_here)
            
            return max_so_far
        
        return max_subarray_divide_conquer, max_subarray_kadane

# Performance comparison of optimizations
def demonstrate_optimizations():
    """
    Show performance improvements from optimizations
    """
    import time
    
    def time_function(func, *args):
        start = time.time()
        result = func(*args)
        end = time.time()
        return result, end - start
    
    # Fibonacci comparison
    fib_naive, fib_memo, fib_opt = fibonacci_optimizations()
    
    n = 35
    print("Fibonacci Optimizations:")
    
    # Memoized version
    _, memo_time = time_function(fib_memo, n)
    print(f"Memoized Fibonacci({n}): {memo_time:.6f}s")
    
    # Optimized version
    _, opt_time = time_function(fib_opt, n)
    print(f"Optimized Fibonacci({n}): {opt_time:.6f}s")
    
    # Two Sum comparison
    arr = list(range(10000))
    target = 15000
    
    # Hash table version
    two_sum_hash, _, _ = hash_table_optimizations()
    _, hash_time = time_function(two_sum_hash, arr, target)
    print(f"Two Sum (Hash): {hash_time:.6f}s")
```

## Best and Worst Case Scenarios

### Comprehensive Scenario Analysis
```python
def best_worst_case_analysis():
    """
    Analyze best, average, and worst case scenarios for algorithms
    """
    
    # Quick Sort Analysis
    def quicksort_scenarios():
        def quicksort(arr, low=0, high=None):
            if high is None:
                high = len(arr) - 1
            
            if low < high:
                pivot_index = partition(arr, low, high)
                quicksort(arr, low, pivot_index - 1)
                quicksort(arr, pivot_index + 1, high)
            
            return arr
        
        def partition(arr, low, high):
            pivot = arr[high]
            i = low - 1
            
            for j in range(low, high):
                if arr[j] <= pivot:
                    i += 1
                    arr[i], arr[j] = arr[j], arr[i]
            
            arr[i + 1], arr[high] = arr[high], arr[i + 1]
            return i + 1
        
        # Best Case: O(n log n) - Pivot divides array into equal halves
        best_case = [4, 2, 6, 1, 3, 5, 7]  # Well-balanced partitions
        
        # Worst Case: O(n²) - Pivot is always smallest or largest
        worst_case = [1, 2, 3, 4, 5, 6, 7]  # Already sorted
        
        # Average Case: O(n log n) - Random partitions
        import random
        average_case = list(range(100))
        random.shuffle(average_case)
        
        return best_case, worst_case, average_case
    
    # Binary Search Analysis
    def binary_search_scenarios():
        def binary_search_with_comparisons(arr, target):
            left, right = 0, len(arr) - 1
            comparisons = 0
            
            while left <= right:
                comparisons += 1
                mid = (left + right) // 2
                
                if arr[mid] == target:
                    return mid, comparisons
                elif arr[mid] < target:
                    left = mid + 1
                else:
                    right = mid - 1
            
            return -1, comparisons
        
        arr = list(range(1, 17))  # [1, 2, 3, ..., 16]
        
        # Best Case: O(1) - Target is at middle
        best_target = arr[len(arr) // 2]
        
        # Worst Case: O(log n) - Target is at end or doesn't exist
        worst_target = arr[-1]  # Last element
        
        # Average Case: O(log n) - Random target
        average_target = arr[3]  # Some random element
        
        best_result = binary_search_with_comparisons(arr, best_target)
        worst_result = binary_search_with_comparisons(arr, worst_target)
        average_result = binary_search_with_comparisons(arr, average_target)
        
        return {
            'best': best_result,
            'worst': worst_result,
            'average': average_result
        }
    
    # Hash Table Analysis
    def hash_table_scenarios():
        class HashTableWithStats:
            def __init__(self, size=10):
                self.size = size
                self.table = [[] for _ in range(size)]
                self.collision_count = 0
            
            def _hash(self, key):
                return hash(key) % self.size
            
            def insert(self, key, value):
                index = self._hash(key)
                bucket = self.table[index]
                
                if bucket:  # Collision occurred
                    self.collision_count += 1
                
                bucket.append((key, value))
            
            def search(self, key):
                index = self._hash(key)
                bucket = self.table[index]
                
                comparisons = 0
                for k, v in bucket:
                    comparisons += 1
                    if k == key:
                        return v, comparisons
                
                return None, comparisons
        
        # Best Case: O(1) - No collisions, perfect hash function
        best_case_ht = HashTableWithStats(size=100)
        for i in range(10):
            best_case_ht.insert(i * 100, f"value_{i}")  # Well-distributed keys
        
        # Worst Case: O(n) - All keys hash to same bucket
        worst_case_ht = HashTableWithStats(size=10)
        for i in range(10):
            # All keys hash to same value (designed collision)
            key = i * worst_case_ht.size
            worst_case_ht.insert(key, f"value_{i}")
        
        return best_case_ht, worst_case_ht
    
    # Tree Operations Analysis
    def tree_scenarios():
        class TreeNode:
            def __init__(self, val=0, left=None, right=None):
                self.val = val
                self.left = left
                self.right = right
        
        # Best Case: Balanced Binary Tree - O(log n) operations
        def create_balanced_tree(values):
            if not values:
                return None
            
            mid = len(values) // 2
            root = TreeNode(values[mid])
            root.left = create_balanced_tree(values[:mid])
            root.right = create_balanced_tree(values[mid + 1:])
            return root
        
        # Worst Case: Skewed Tree - O(n) operations
        def create_skewed_tree(values):
            if not values:
                return None
            
            root = TreeNode(values[0])
            current = root
            
            for val in values[1:]:
                current.right = TreeNode(val)
                current = current.right
            
            return root
        
        def tree_height(root):
            if not root:
                return 0
            return 1 + max(tree_height(root.left), tree_height(root.right))
        
        values = list(range(1, 16))  # 15 nodes
        
        balanced_tree = create_balanced_tree(values)
        skewed_tree = create_skewed_tree(values)
        
        balanced_height = tree_height(balanced_tree)
        skewed_height = tree_height(skewed_tree)
        
        return {
            'balanced_height': balanced_height,  # ~log n
            'skewed_height': skewed_height       # ~n
        }
    
    return quicksort_scenarios(), binary_search_scenarios(), hash_table_scenarios(), tree_scenarios()

# Amortized Analysis Examples
def amortized_analysis():
    """
    Demonstrate amortized time complexity analysis
    """
    
    # Dynamic Array (Python List) - Amortized O(1) append
    class DynamicArray:
        def __init__(self):
            self.capacity = 1
            self.size = 0
            self.data = [None] * self.capacity
            self.resize_count = 0
        
        def append(self, value):
            if self.size == self.capacity:
                self._resize()
            
            self.data[self.size] = value
            self.size += 1
        
        def _resize(self):
            self.resize_count += 1
            old_capacity = self.capacity
            self.capacity *= 2
            
            new_data = [None] * self.capacity
            for i in range(self.size):
                new_data[i] = self.data[i]
            
            self.data = new_data
            print(f"Resized from {old_capacity} to {self.capacity}")
        
        def get_stats(self):
            return {
                'size': self.size,
                'capacity': self.capacity,
                'resize_count': self.resize_count,
                'load_factor': self.size / self.capacity
            }
    
    # Demonstrate amortized O(1) append
    def demonstrate_dynamic_array():
        arr = DynamicArray()
        
        # Add 100 elements and track resizes
        for i in range(100):
            arr.append(i)
            if i in [0, 1, 3, 7, 15, 31, 63, 99]:  # Powers of 2 - 1
                stats = arr.get_stats()
                print(f"After {i+1} insertions: {stats}")
    
    return demonstrate_dynamic_array
```

## Practical Applications

### Real-World Complexity Considerations
```python
def practical_applications():
    """
    Real-world applications and complexity considerations
    """
    
    # Database Query Optimization
    def database_query_analysis():
        """
        Simulate database query complexities
        """
        
        # Table scan: O(n)
        def full_table_scan(table, condition):
            results = []
            for row in table:
                if condition(row):
                    results.append(row)
            return results
        
        # Index lookup: O(log n) for B-tree index
        def index_lookup(index, key):
            # Simplified binary search on sorted index
            left, right = 0, len(index) - 1
            
            while