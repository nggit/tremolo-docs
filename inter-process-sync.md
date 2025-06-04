---
layout: page
title: Inter-process Synchronization
---

When Tremolo HTTP server is configured with [worker_num](configuration.html#worker_num) > 1, it can be dangerous e.g. when you have code that writes to the same file.

`server['lock']` can be used to tackle this problem.

It can synchronize all tasks across multiple workers/processes. In single process mode, it will only be like [asyncio.Lock](https://docs.python.org/3/library/asyncio-sync.html#asyncio.Lock), but thread-safe.

Example of using the asynchronous [context manager](https://python.readthedocs.io/en/latest/glossary.html#term-context-manager) with `async with`:

```python
@app.route('/update')
async def update(**server):
    lock = server['lock']

    async with lock:
        # lock acquired, modify the shared resource
        # other processes that are acquiring this lock should be waiting
```

Or the traditional way:

```python
@app.route('/update')
async def update(**server):
    lock = server['lock']

    await lock.acquire()
    try:
        # lock acquired, modify the shared resource
        # other processes that are acquiring this lock should be waiting
    finally:
        lock.release()
```

## Multiple Shared Resources
In case you have multiple shared resources/different files, using a single lock as above can block other concurrent tasks - **even** if they are going to modify different files. It makes sense to have more than one lock provided for each resource for maximum concurrency.

Tremolo by default has **5** usable locks, `0 - 4`.

```python
# in current task
    lock = server['lock']

    async with lock(0):
        # lock acquired, modify the file1
        # other processes that are acquiring this lock(0) should be waiting

# in another concurrent task, independent from lock(0)
    lock = server['lock']

    async with lock(1):
        # lock acquired, modify the file2
        # other processes that are acquiring this lock(1) should be waiting
```

In the example above we found using two locks, `lock(0)` and `lock(1)`, for two different files.

`lock(0)` is basically the same as `lock`, and if the given number exceeds the default range of `0 - 4`, for example **5**, it will be rotated using modulo internally. Meaning `lock(5)` will only be the same as `lock(0)` or `lock`.

If more than 5 locks are required, they can be set with the [thread_pool_size](/tremolo-docs/configuration.html#thread_pool_size) configuration.

This is a design decision; that locks cannot be added on-demand/dynamically. Otherwise the implementation will be more complex and likely to be [overhead](https://docs.python.org/3/library/multiprocessing.html#multiprocessing.Manager).
