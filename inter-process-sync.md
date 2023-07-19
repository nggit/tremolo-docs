---
layout: page
title: Inter-process Synchronization
---

When Tremolo HTTP server is configured with [worker_num](configuration.html#worker_num) > 1, it can be dangerous e.g. when you have code that writes to the same file.

`server['lock']` can be used to avoid this potential problem.

It can synchronize all tasks between multiple workers/processes. In single process mode, it will only be like [asyncio.Lock](https://docs.python.org/3/library/asyncio-sync.html#asyncio.Lock).

Example of using the asynchronous [context manager](https://python.readthedocs.io/en/latest/glossary.html#term-context-manager) with `async with`:

```python
@app.route('/update')
async def update(**server):
    lock = server['lock']

    async with lock:
        # lock acquired, modify the file
        # other processes should be waiting
```

Or the traditional way:

```python
@app.route('/update')
async def update(**server):
    lock = server['lock']

    try:
        await lock.acquire()
        # lock acquired, modify the file
        # other processes should be waiting
    finally:
        lock.release()
```
