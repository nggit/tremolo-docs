---
layout: page
nav_order: 4
title: Handlers
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

`server` is a dict object, which contains other objects that are often needed, such as `server['request']` which is an [HTTPRequest](https://nggit.github.io/tremolo-docs/request.html) object, and `server['response']` which is an [HTTPResponse](https://nggit.github.io/tremolo-docs/response.html) object.

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

You will see a log:
```
handler hello_world has exited with the connection possibly left open
```
Be careful that the connection may hang forever, and not be subject to keep-alive termination.
In other words, the client can abuse by not closing the connection.
