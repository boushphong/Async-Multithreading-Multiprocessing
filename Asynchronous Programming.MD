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
