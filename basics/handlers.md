---
layout: page
nav_order: 4
title: Handlers
parent: Basics
---

A **route handler** or simply **handler** is a function that binds to a certain route. The following is a handler called `hello_world`, and it binds to a `/hello`.

```python
@app.route('/hello')
async def hello_world(**server):
    yield b'Hello'
    yield b'World!'
```

or alternative version:

```python
@app.route('/hello')
async def hello_world(request, response):
    await response.write(b'Hello')
    await response.write(b'World!')
    await response.write(b'')

    # optional. FYI, it is required on middleware
    response.close()
```


Each handler is required to accept a *keyword arguments* in this case `**server`. Or on an individual basis like `request`, `response`, etc. (number and order have no effect).

`server` is a dict object, which contains other objects that are often needed, such as `server['request']` which is an [HTTPRequest](/tremolo-docs/reference/request/) object, and `server['response']` which is an [HTTPResponse](/tremolo-docs/reference/response/) object.

You can provide some options to fine tune the handler such as `chunked`, `rate`, `buffer_size`, etc.

```python
@app.route('/hello')
async def hello_world(request, chunked=False, rate=2097152, buffer_size=32768):
    yield b'Hello'
    yield b'World!'
```

* *chunked=False* will disable chunked HTTP responses, the default is automatically adjusted for clients.
* *rate* 2MiB/s is the download speed limit for each client. This is useful for limiting bandwidth usage as well as mitigating bandwidth hogs.
* *buffer_size* Tremolo will always process data every chunk based on this size.

In addition to the options above, you can define arbitrary options, and they will magically become available in `request.ctx.options`.

```python
@app.route('/hello')
async def hello_world(request, a=1, rate=2097152, buffer_size=32768):
    print(request.ctx.options)

    yield b'Hello'
    yield b'World!'
```

## Server behavior regarding the return value of the handler
Unlike most frameworks, Tremolo favors the use of built-in data types in return values, rather than an opinionated *HTTPResponse*.

### 1. return b''
```python
@app.route('/hello')
async def hello_world(**server):
    return b'Hello, World!'
```
If the handler returns a *string* / *bytes*-like, the data will be sent as a response body along with the `Content-Length` (for HTTP/1.0) or `Transfer-Encoding: chunked` (for HTTP/1.1).
The connection will be **closed with respect to keep-alive**.

### 2. return None
```python
@app.route('/hello')
async def hello_world(**server):
    pass
```
If the handler returns nothing, or None, the connection will be **forcibly closed**, without sending any data to the client, not even headers.

Just like doing `response.close()`.

### 3. return anything, e.g. True
```python
@app.route('/hello')
async def hello_world(**server):
    // do something
    return True
```
If the handler returns something other than the above, such as `True`, the connection **will not be closed**.
This assumes you will be managing the connection manually.

Although you can use anything, always use `True` to avoid breakage in the future.

You will see a log:
```
handler hello_world has exited with the connection possibly left open
```
Be careful that the connection may hang forever, and not be subject to keep-alive termination.
In other words, the client can abuse by not closing the connection.

## Handler cancellation
Tremolo will automatically kill handlers that exceed the [app_handler_timeout](/tremolo-docs/configuration.html#app_handler_timeout) or [app_close_timeout](/tremolo-docs/configuration.html#app_close_timeout) limit.

The handler will not be killed if you are working with an upgraded connection like [WebSocket](/tremolo-docs/reference/websocket/). The client will be connected virtually forever, [as long as the server is not full](https://github.com/nggit/tremolo/discussions/276).

You can wrap your code using *try - finally* if you want to do some clean up.
```python
@app.route('/')
async def my_handler(**server):
    try:
        await asyncio.sleep(123)
    finally:
        # do clean up

    # not reached
    return 'OK'

```

Make sure you allow enough time for uploading cases. On downloads with [response.sendfile()](/tremolo-docs/how-to/resumable-downloads.html), there is no need to worry as browsers can generally re-request `Range`s that are still needed.

## Synchronous handlers
The async paradigm may be painful for beginners or old school. Support for synchronous handlers was added in [#286](https://github.com/nggit/tremolo/pull/286) .

You can simply omit the `async` keyword in handler declarations. You can run blocking code inside without interrupting the main thread as each handler will be executed on a separate thread. There are 5 executor threads available as default which you can configure with [thread_pool_size](/tremolo-docs/configuration.html#thread_pool_size).

The sync handlers are suitable for longer, blocking IO operations such as loading a template file in one go, etc. For persistent connections such as WebSocket/SSE, async is still the optimal choice. As it scales well because it requires fewer threads to handle more connections.

```python
from threading import current_thread


@app.route('/')
def sync_handler(request, **server):
    return current_thread().name

```

If you are using a sync handler then you obviously can't use `await`. And since Tremolo is an *async-first* framework, most functions should be awaited.

Therefore in [#289](https://github.com/nggit/tremolo/pull/289) `AsyncToSyncWrapper` is implemented to work around this. Basically you don't need to use `await` and it still works.

```python
@app.route('/async')
async def async_handler(request, **server):
    print(request)  # <tremolo.lib.http_request.HTTPRequest object at 0x...>

    data = await request.body()
    return data


@app.route('/sync')
def sync_handler(request, **server):
    print(request)  # <tremolo.utils.AsyncToSyncWrapper object at 0x...>

    data = request.body()  # it works!
    return data

```

Cool! But keep in mind, reading the request body in the sync handler will be limited to 5 concurrent connections only. Because 1 thread can only hold one *request - response* cycle at a time.

This is not necessarily a bad thing, [thread size limits]((/tremolo-docs/configuration.html#thread_pool_size)) can also naturally *backpressure* memory usage.

{: .warning }
**A caveat about sync handlers:** [Handler cancellation](#handler-cancellation) like the previous topic will not work as expected; it does not propagate to the thread. Even if the client is disconnected, tasks on the thread cannot be stopped and **there is no way to kill the thread from the outside**. You may encounter thread pool starvation, a blocked situation **if you do not make sure your code terminates**. Although this case does not degrade the async part of the server.
