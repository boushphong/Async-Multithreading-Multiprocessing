# Multiprocessing
**Multiprocessing** refers to the ability of a system to support more than one processor at the same time. In Python, the multiprocessing module allows us to create processes, and offers both local and remote concurrency. It side-steps the Global Interpreter Lock by using subprocesses instead of threads and lets us leverage multiple processors to perform computations.

The multiprocessing module provides features like process pools, interprocess communication, shared state and robustness, which are not available with threading.

## Overcoming the GIL with Multiprocessing
The Global Interpreter Lock (GIL) is a mechanism used in Python's CPython interpreter to synchronize access to Python objects, preventing multiple threads from executing Python bytecodes at once. This lock is necessary because CPython's memory management is not thread-safe.

However, the GIL is not a problem with multiprocessing because each process has its own Python interpreter and memory space, so the GIL won’t be a bottleneck. The processes do not share memory and as a result, they can run on different cores of your CPU in true parallel fashion. This makes multiprocessing useful in scenarios where the program is CPU intensive and doesn't have to do any IO or user interaction.

It's important to note that while multiprocessing overcomes the GIL, it introduces a new set of complexities. Inter-process communication is more complex and slower than inter-thread communication because processes do not share memory space. Also, starting a new process has a larger overhead than starting a new thread.


# Spawning multiple processes
```python
import multiprocessing


def calc_square(numbers):
    for n in numbers:
        print('square:', n * n)


def calc_cube(numbers):
    for n in numbers:
        print('cube:', n * n * n)


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
        print("square " + str(n*n))
        square_result.append(n*n)
    print("Within a process: Result: " + str(square_result))


if __name__ == '__main__':
    arr = range(4)
    p1 = multiprocessing.Process(target=calc_square, args=(arr,))

    p1.start()
    p1.join()

    print("Main process's result: " + str(square_result))

# Print Result
"""
square 0
square 1
square 4
square 9
Within a process: Result: [0, 1, 4, 9]
Main process's result: []
"""
```
Since every process has its own address space (virtual memory). Thus program variables are not shared between two processes. In this case, interprocess communication (IPC) techniques would have to be used to share data between processes.

The `square_result` in the code example is an empty list `[]` because when we append to `square_result` in the `calc_square` function, we're appending to the list in the child process's memory space, not the list in the main process's memory space. That's why the square_result list is still empty when we print it in the `main` process.


## Forking in Python
The `multiprocessing` module in Python uses a technique called **"forking"** to create new processes. **Forking** is a way to create a new process by duplicating the existing process. The new process, called the child, is an exact copy of the calling process, called the parent, except for a few values that get changed, such as the process ID.

When you call `multiprocessing.Process()`, Python forks the current process. The child process that's created runs the function you passed in, while the parent process continues to the next line of code.

