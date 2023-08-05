```python
import asyncio


async def fetch_data(data: str) -> dict:
    print(f"Start fetching {data}")
    await asyncio.sleep(2)
    print(f"Done fetching {data}")
    return {"data": data}


async def counter():
    for i in range(10):
        await asyncio.sleep(0.5)
        print(i)


async def main():
    task1 = asyncio.create_task(fetch_data('data'))
    task2 = asyncio.create_task(counter())

    data: dict = await task1
    print(data)


asyncio.run(main())
# Start fetching data
# 0
# 1
# 2
# Done fetching data
# {'data': 'data'}
```

```python3
import asyncio


async def fetch_data(data: str) -> dict:
    print(f"Start fetching {data}")
    await asyncio.sleep(2)
    print(f"Done fetching {data}")
    return {"data": data}


async def counter():
    for i in range(10):
        await asyncio.sleep(0.5)
        print(i)


async def main():
    task1 = asyncio.create_task(fetch_data('data'))
    task2 = asyncio.create_task(counter())

    data: dict = await task1
    print(data)
    await task2


asyncio.run(main())
# Start fetching data
# 0
# 1
# 2
# Done fetching data
# {'data': 'data'}
# 3
# 4
# 5
# 6
# 7
# 8
# 9
```

```python3
import asyncio


async def fetch_data(data: str) -> dict:
    print(f"Start fetching {data}")
    await asyncio.sleep(5)
    print(f"Done fetching {data}")
    return {"data": data}


async def main():
    task = asyncio.create_task(fetch_data("data"))
    await asyncio.sleep(2)

    task.cancel()
    try:
        data: dict = await task
        print(data)
    except asyncio.CancelledError:
        print("Task was cancelled")


asyncio.run(main())
# Start fetching data (after 2 seconds)
# Task was cancelled
```

`asyncio.sleep()` is used as a way to yield control back to the event loop, allowing other tasks (coroutines) to run.
The asyncio event loop maintains a queue of tasks (coroutines) that are ready to run. When a task awaits on an operation (like `asyncio.sleep()`), it's suspended and control is returned to the event loop. The event loop then runs the next task in its queue. When the awaited operation is complete (in this case, when the sleep time is over), the task is added back to the event loop's queue and will be run again when its turn comes up.
