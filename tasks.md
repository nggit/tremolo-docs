---
layout: page
title: Tasks and Contexts
---

You can create tasks using the [server['loop']](https://docs.python.org/3/library/asyncio-eventloop.html). Consider the following example:

```python
import asyncio

from tremolo import Tremolo

app = Tremolo()

async def my_coro(fut):
    await asyncio.sleep(10)
    fut.set_result(b'Promised result after sleep 10s')

@app.route('/hello')
async def hello_world(**server):
    tasks = server['context'].tasks
    loop = server['loop']

    fut = loop.create_future()

    tasks.append(
        loop.create_task(my_coro(fut))
    )

    yield b'Processing... '

    await fut # wait promise done

    yield fut.result()

if __name__ == '__main__':
    app.run('0.0.0.0', 8000, debug=True)
```

The code above runs an *awaitable* `my_coro` using `loop.create_task()`. So that it does not wait for `my_coro` in the main task, not blocking the following `yield b'Processing...` to be executed.

Appending tasks to `server['context'].tasks` is not required. But it will help Tremolo to cancel pending tasks on client disconnect.

For convenience, you can use the `ServerTasks.create`. Just add `tasks=None` placeholder parameter in the [handler](handlers.html) to use it:

```python
@app.route('/hello')
async def hello_world(tasks=None, **server):
    # ...

    task = tasks.create(my_coro(fut))
```

## Contexts
`server['context']` is a mutable object (think a `dict` with a dot notation, or a [SimpleNamespace](https://docs.python.org/3/library/types.html#types.SimpleNamespace)).

It can be used to share state or data e.g. between [middleware](middleware.html) and [handler](handlers.html), in a lifetime of a request.

```python
ctx = server['context']
# or
# ctx = server['request'].context

ctx.anykey = 'mydata'
```

Note that the following attributes are reserved:
`options`, and `tasks`.
