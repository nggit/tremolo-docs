---
layout: page
title: Tasks and Contexts
---

You can create asynchronous tasks using the [loop.create_task()](https://docs.python.org/3/library/asyncio-eventloop.html#asyncio.loop.create_task) directly or `request.protocol.create_task()`. The difference is that the latter is tracked by the server and will be canceled when the client disconnects.

To run tasks that keep running in the background even if the client connection is lost use `request.protocol.create_background_task()`. The reference of these tasks will be in `server['globals'].tasks`.

Consider the following example:

```python
import asyncio

from tremolo import Application

app = Application()

async def my_coro(fut):
    await asyncio.sleep(10)
    fut.set_result(b'Promised result after sleep 10s')

@app.route('/hello')
async def hello_world(request, loop):
    fut = loop.create_future()
    request.protocol.create_task(my_coro(fut))

    yield b'Processing... '

    await fut # wait promise done

    yield fut.result()


if __name__ == '__main__':
    app.run('0.0.0.0', 8000, debug=True)
```

If you want to execute the synchronous/blocking tasks instead, you can look at the [awaiter module](https://pypi.org/project/awaiter/).

## Contexts
A *Context* is a mutable object (think a `dict` with a dot notation, or a [SimpleNamespace](https://docs.python.org/3/library/types.html#types.SimpleNamespace)).

There are three kinds of context, which are `WorkerContext`, `ConnectionContext` and `RequestContext`.

### RequestContext
`request.context` can be used to share state or data e.g. between [middleware](middleware.html) and [handler](handlers.html), in a lifetime of a request.

```python
ctx = request.context
# or
# ctx = request.ctx

ctx.anykey = 'mydata'
```

Note that the following attributes are reserved:
`options`, and `tasks`.

### WorkerContext
`server['globals']` is a worker-level context. Which means it can be accessed from anywhere.
