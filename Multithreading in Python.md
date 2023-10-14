# Multithreading
**Multithreading** is a technique in programming where a single set of code can be used by multiple threads, which are scheduled to run in an interleaved manner by the operating system. This allows programs to perform multiple tasks concurrently, which can be especially beneficial for I/O-bound tasks.

In Python, due to the Global Interpreter Lock (GIL), only one thread can execute at a time in a single process. This means that even though you might have multiple threads, they're not truly running in parallel on multiple cores; instead, they're taking turns using the same core. However, this is not a problem for I/O-bound tasks, which spend most of their time waiting for input and output operations to complete (like reading from or writing to a disk, or sending a request to a server and waiting for a response). During the time that one thread is waiting for an I/O operation to complete, another thread can use the CPU.

In languages or environments without a GIL (like Java or C++), multithreading can indeed allow a single set of code to be used by several processors at different stages of execution, leading to true parallel execution. However, this is not the case in Python due to the GIL.
```python
import concurrent.futures
import time


def print_cube(num):
    time.sleep(3)
    print("Cube: {}".format(num * num * num))


def print_square(num):
    time.sleep(3)
    print("Square: {}".format(num * num))


if __name__ == "__main__":
    # We can use a with statement to ensure threads are cleaned up promptly
    with concurrent.futures.ThreadPoolExecutor() as executor:
        # Start the load operations and mark each future with its URL
        future1 = executor.submit(print_square, 10)
        future2 = executor.submit(print_cube, 10)
```

In the above example, calling each funtion sequentially will have estimated accumulated time of 6 seconds. However, since the 2 functions spend most of their times sleeping. We can utilize multithreading technique to shorten execution time.

We can achieve the same result with python's `threading` library.
```python
import threading
import time


def print_cube(num):
    time.sleep(3)
    print("Cube: {}".format(num * num * num))


def print_square(num):
    time.sleep(3)
    print("Square: {}".format(num * num))


if __name__ == "__main__":
    t1 = threading.Thread(target=print_square, args=(10,))
    t2 = threading.Thread(target=print_cube, args=(10,))

    t1.start()
    t2.start()

    t1.join()
    t2.join()
```
