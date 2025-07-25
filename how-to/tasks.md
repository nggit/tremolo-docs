---
layout: page
title: Tasks and Contexts
parent: How-To
---

You can create asynchronous tasks using the [loop.create_task()](https://docs.python.org/3/library/asyncio-eventloop.html#asyncio.loop.create_task) directly or `request.server.create_task()`. The difference is that the latter is tracked by the server and will be canceled when the client disconnects.

To run tasks that keep running in the background even if the client connection is lost use `server['app'].create_task()`. The reference of these tasks will be in `server['globals'].tasks`.

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
    request.server.create_task(my_coro(fut))

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

### 1. RequestContext
`request.context` can be used to share state or data e.g. between [middleware](/tremolo-docs/basics/middleware.html) and [handler](/tremolo-docs/basics/handlers.html), in a lifetime of a request.

```python
ctx = request.context
# or
# ctx = request.ctx

ctx.anykey = 'mydata'
```

Note that the following attributes are reserved:
`options`, and `tasks`.

### 2. WorkerContext
`server['globals']` is a worker-level context. Which means it can be accessed from anywhere, anytime.

### 3. ConnectionContext
`server['context']` is a connection-level context that is alive from the start of the connection until the client disconnects. Rarely used.

## Contexts lifetime
```
|-W1- server['globals'] (WorkerContext) ----------------------------W2->|

|-C1- server['context'] -C2->|- server['context'] (ConnectionContext) ->|

|-R1--- request.ctx ----R2-> |--- request.ctx --->|--- request.ctx ---->|


Hooks:

W1: @app.on_worker_start
W2: @app.on_worker_stop

C1: @app.on_connect (TCP connection)
C2: @app.on_close

R1: @app.on_request (HTTP transaction)
R2: @app.on_response
```

A single TCP connection can hold multiple consecutive HTTP transactions/requests (Keep-Alive).
In a non-Keep-Alive situation, `server['context']` has the same lifetime as `request.ctx`.

Anyway, `request.ctx` is generally all you need.
