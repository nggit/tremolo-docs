---
layout: page
nav_order: 3
title: Handlers
---

A **route handler** or simply **handler** is a function that binds to a certain route. The following is a handler called `hello_world`, and it binds to a `/hello`.

```python
@app.route('/hello')
async def hello_world(**server):
    yield b'Hello'
    yield b'World!'
```

Each handler is required to accept a *keyword arguments* in this case `**server`. Although the name does not have to be `server`, for example `**kwargs`.

`server` is a dict object, which contains other objects that are often needed, such as `server['request']` which is an `HTTPRequest` object, and `server['response']` which is an `HTTPResponse` object.

Before the `**server`, you can provide some options to fine tune the handler such as `chunked`, `rate`, `buffer_size`, etc.

```python
@app.route('/hello')
async def hello_world(chunked=False, rate=2097152, buffer_size=65536, **server):
    yield b'Hello'
    yield b'World!'
```

* *chunked=False* will disable chunked HTTP responses, the default is automatically adjusted for clients.
* *rate* 2MiB/s is the download speed limit for each client. This is useful for limiting bandwidth usage as well as mitigating bandwidth hogs.
* *buffer_size* is the maximum size of data that must be sent to the client immediately. although in Tremolo it more accurately means "the maximum size of an individual chunk". Because no matter how large a buffer you set for reading a file, Tremolo may divide it according to the `buffer_size` on each transmission.

In addition to the options above, you can even define your own options, and they will magically become available in `server['options']`.

```python
@app.route('/hello')
async def hello_world(a=1, rate=2097152, buffer_size=65536, **server):
    print(server['options'])

    yield b'Hello'
    yield b'World!'
```
