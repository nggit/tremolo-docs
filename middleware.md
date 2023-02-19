---
layout: page
title: Middleware
---

Middleware is like MITM. It can *intercept* **requests** before they are processed by handlers, and **responses** before they are sent to the client/browser.

Currently, Tremolo has two kinds of middleware that can be created. Which are `on_request` and `on_data`.

The `on_request` allows you to put certain code globally at the very front.
You can filter, authenticate, then halt (if necessary) before the request goes to / is processed by handlers.

Let's say we have a `hello_world` handler:

```python
@app.route('/hello')
async def hello_world(**server):
    yield b'Hello'
    yield b'World!'
```

Normally, the response will be as follows if you do `curl -i http://localhost:8000/hello`:

```
HTTP/1.1 200 OK
Date: Thu, 09 Feb 2023 03:22:36 GMT
Server: Tremolo
Transfer-Encoding: chunked
Content-Type: text/html; charset=utf-8
Connection: keep-alive

Hello World!
```

## Middleware in action
As Tremolo supports arbitrary request methods, You can halt for example if the received request method is neither `GET` nor `POST`:
```python
@app.on_request
async def my_request_middleware(**server):
    request = server['request']
    response = server['response']

    if request.is_valid and request.method not in (b'GET', b'POST'):
        response.set_status(405, 'Method Not Allowed')
        response.set_content_type('text/plain')

        """
        Halt with return. The request will end at this point.
        The next middlewares (if any), and handlers
        will not be executed.
        """
        return b'Request method %s is not supported!' % request.method

@app.route('/hello')
async def hello_world(**server):
    yield b'Hello'
    yield b'World!'
```

The response result of `curl -X FOO -i http://localhost:8000/hello` will be as follows:

```
HTTP/1.1 405 Method Not Allowed
Content-Type: text/plain
Content-Length: 36
Connection: close
Date: Thu, 09 Feb 2023 03:30:45 GMT
Server: Tremolo

Request method FOO is not supported!
```

You may need to use `on_data` for but not limited to, implementing caching.

Consider the following code:

```python
@app.on_data
async def my_data_middleware(**server):
    data = server['options']['data']

    print(data)
```

Here is a `print` result of the data before it is sent to the client in the form of a tuple pair, `(name, data)`.

```python
('header', b'HTTP/1.1 200 OK\r\n')
('header', bytearray(b'Date: Thu, 09 Feb 2023 04:00:36 GMT\r\nServer: Tremolo\r\nTransfer-Encoding: chunked\r\n'))
('header', b'Content-Type: text/html; charset=utf-8\r\nConnection: keep-alive\r\n\r\n')
('body', b'Hello')
('body', b'World!')
('body', b'')
('data', None)
```

Since the `on_data` handler may be called multiple times, the `name` field is handy to tell if it is `header` or `body` part.
