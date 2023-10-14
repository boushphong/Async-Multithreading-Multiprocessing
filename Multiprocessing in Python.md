# Multiprocessing
**Multiprocessing** refers to the ability of a system to support more than one processor at the same time. In Python, the multiprocessing module allows us to create processes, and offers both local and remote concurrency. It side-steps the Global Interpreter Lock by using subprocesses instead of threads and lets us leverage multiple processors to perform computations.

The multiprocessing module provides features like process pools, interprocess communication, shared state and robustness, which are not available with threading.

## Difference between a Process and a Thread
A **process** is an instance of a program that is being executed. It contains the program code and its current activity. Each process has a separate memory space, which means that a process runs independently and is isolated from other processes. It cannot directly access shared data in other processes. Switching from one process to another requires some time (relatively) for saving and loading registers, memory maps, and other administrative information.

A **thread**, on the other hand, is a subset of a process. A thread is a sequence of instructions within a process. It can be thought of as a lightweight process. Threads within the same process share the same data space with the main process and can therefore share information more easily than if they were separate processes.

**Some key differences:**
- Threads will share the same address space of a process, they will have their own instruction sets. Each thread will have a specific task to execute, hence they will have their own stack memory, instruction pointer,. In Python, if you have a global variable in your program, this variable can be accessed by another threads. Heap memory can be access be threads as well.

- Processes operate within their own distinct address spaces. If they need to share information, they must use interprocess communication (IPC) mechanisms. These mechanisms can include techniques like reading and writing to a shared file on disk, utilizing shared memory, or employing message passing via pipes.

- A memory leak within a process typically remains confined to that specific process, given that there's no interprocess communication involved. This means it doesn't impact other processes. However, a memory leak within a thread is more problematic as it can influence the entire program that the thread belongs to, potentially affecting other threads within the same program.

## Overcoming the GIL with Multiprocessing
The Global Interpreter Lock (GIL) is a mechanism used in Python's CPython interpreter to synchronize access to Python objects, preventing multiple threads from executing Python bytecodes at once. This lock is necessary because CPython's memory management is not thread-safe.

However, the GIL is not a problem with multiprocessing because each process has its own Python interpreter and memory space, so the GIL won’t be a bottleneck. The processes do not share memory and as a result, they can run on different cores of your CPU in true parallel fashion. This makes multiprocessing useful in scenarios where the program is CPU intensive and doesn't have to do any IO or user interaction.

It's important to note that while multiprocessing overcomes the GIL, it introduces a new set of complexities. Inter-process communication is more complex and slower than inter-thread communication because processes do not share memory space. Also, starting a new process has a larger overhead than starting a new thread.


# Spawning multiple processes
```python
import multiprocessing

def calc_square(numbers):
    for n in numbers:
        print("square:", n * n)


def calc_cube(numbers):
    for n in numbers:
        print("cube:", n * n * n)


if __name__ == "__main__":
    arr = range(4)
    p1 = multiprocessing.Process(target=calc_square, args=(arr,))
    p2 = multiprocessing.Process(target=calc_cube, args=(arr,))

    p1.start()
    p2.start()

    p1.join()
    p2.join()

# Print Result
"""
square: 0
square: 1
square: 4
square: 9
cube: 0
cube: 1
cube: 8
cube: 27
"""
```
There will be a total of 3 processes in the code example.
1. The `main` process: This is the process that runs your script. When it reaches the `multiprocessing.Process` calls, it creates new processes.

2. `p1` process: This is the new process created by the `multiprocessing.Process(target=calc_square, args=(arr,))` call. It runs the `calc_square` function in its own Python interpreter.

3. `p2` process: This is the new process created by the `multiprocessing.Process(target=calc_cube, args=(arr,))` call. It runs the `calc_cube` function in its own Python interpreter.

Each of these processes has its own Python interpreter and memory space. They run independently and the operating system schedules them to run concurrently. This allows the `calc_square` and `calc_cube` functions to run in parallel, potentially on different cores of your CPU.

```python
import multiprocessing

square_result = []

def calc_square(numbers):
    for n in numbers:
        print(f"square: {n*n}")
        square_result.append(n*n)
    print(f"Within a process: Result: {square_result}")


if __name__ == '__main__':
    arr = range(4)
    p1 = multiprocessing.Process(target=calc_square, args=(arr,))

    p1.start()
    p1.join()

    print(f"Main process's result: {square_result}")

# Print Result
"""
square: 0
square: 1
square: 4
square: 9
Within a process: Result: [0, 1, 4, 9]
Main process's result: []
"""
```
Since every process has its own address space (virtual memory). Thus program variables are not shared between two processes. In this case, interprocess communication (IPC) techniques would have to be used to share data between processes.

The `square_result` in the code example is an empty list `[]` because when we append to `square_result` in the `calc_square` function, we're appending to the list in the child process's memory space, not the list in the main process's memory space. That's why the square_result list is still empty when we print it in the `main` process.


## Forking in Python
The `multiprocessing` module in Python uses a technique called **"forking"** to create new processes. **Forking** is a way to create a new process by duplicating the existing process. The new process, called the child, is an exact copy of the calling process, called the parent, except for a few values that get changed, such as the process ID.

When you call `multiprocessing.Process()`, Python forks the current process. The child process that's created runs the function you passed in, while the parent process continues to the next line of code.

## Interprocess Communication (IPC) techniques
1. **Shared Memory**: Shared memory is one of the fastest methods of Interprocess Communication (IPC). In this method, a memory area is shared between multiple processes. This means that all these processes can access and modify this shared memory directly. It's a very efficient way of passing data between processes, but it requires careful synchronization to avoid race conditions, where two processes attempt to read or modify the same piece of memory at the same time.

The `multiprocessing` module provides a `Value` or `Array` class for storing data in shared memory.
```python
from multiprocessing import Process, Value, Array

def func(n, a):
    n.value = 3.1415927
    for i in range(len(a)):
        a[i] *= -1

if __name__ == '__main__':
    # 'd' stands for double precision floating point number. Value('d', 0.0) creates a shared memory space 
    num = Value('d', 0.0)  that can store a double precision floating point number.
    # 'i' stands for integer. Array('i', range(10)) creates a shared memory space that store an array of integers.
    arr = Array('i', range(10))

    p = Process(target=func, args=(num, arr))
    p.start()
    p.join()

    print(num.value)
    print(arr[:])

# Print Result
"""
3.1415927
[0, -1, -2, -3, -4, -5, -6, -7, -8, -9]
"""
```

2. **Message Pipe (Pipes)**: Pipes provide a one-way communication channel between processes. A pipe has a write end and a read end. One process writes data to the pipe, and another process reads that data. The data is handled in a FIFO (First In, First Out) manner. Pipes are particularly useful for setting up communication between parent and child processes or between "chained" processes, such as in a pipeline of shell commands.

**Message Pipe (Pipes)**: The `multiprocessing` module provides a `Pipe` function which returns a pair of connection objects connected by a pipe.
```python
from multiprocessing import Process, Pipe

def func(conn):
    conn.send(['hello world'])
    conn.close()

if __name__ == '__main__':
    parent_conn, child_conn = Pipe()
    p = Process(target=func, args=(child_conn,))
    p.start()
    print(parent_conn.recv())   # prints "[‘hello world’]"
    p.join()
```

3. **File-based Communication**: This is a simple and common method of IPC. In this method, one process writes data to a file, and another process reads from that file to get the data. This method is easy to implement and understand, but it's slower than other methods because it involves disk I/O operations. It's also less secure because any process (or even an external user) with the right permissions can read from or write to the file.

Standard file I/O operations in Python can be used to achieved File-based IPC. However, you need to be careful to avoid race conditions by using some form of locking or synchronization. The `multiprocessing` module provides a `Lock` class that you can use to ensure that only one process writes to the file at a time.

```python
from multiprocessing import Process, Lock

def func(lock, i):
    lock.acquire()
    try:
        with open('file.txt', 'a') as f:
            f.write('Hello world {}\n'.format(i))
    finally:
        lock.release()

if __name__ == '__main__':
    lock = Lock()

    for num in range(10):
        Process(target=func, args=(lock, num)).start()
```
