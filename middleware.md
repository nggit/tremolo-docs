---
layout: page
title: Middleware
---

Middleware is like MITM. It can *intercept* **requests** before they are processed by handlers, and **responses** before they are sent to the client/browser.

Tremolo has two kinds of middleware that can be created. Which are `on_request` and `on_response`.

The `on_request` allows you to put certain code globally at the very front.
You can filter, authenticate, then halt (if necessary) before the request goes to / is processed by handlers.

Middleware can be used to extend functionality. For example, [tremolo-session](https://github.com/nggit/tremolo-session) is built upon middleware.

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
from tremolo.exceptions import BadRequest

@app.on_request
async def my_request_middleware(**server):
    request = server['request']
    response = server['response']

    if not request.is_valid
        raise BadRequest

    if request.method not in (b'GET', b'POST'):
        response.set_status(405, 'Method Not Allowed')
        response.set_content_type('text/plain')

        """Halt with return.

        The request will end at this point.
        The next middlewares (if any), and handlers
        will not be executed.
        """
        return b'Request method %s is not supported!' % request.method

@app.route('/hello')
async def hello_world(**server):
    yield b'Hello'
    yield b'World!'
```

You might notice, other than using `return` to halt, you can `raise` an `HTTPException`:

```python
from tremolo.exceptions import BadRequest, MethodNotAllowed
# ...

    if request.method not in (b'GET', b'POST'):
        raise MethodNotAllowed('Request method', request.method, 'is not supported!')
# ...
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

## Decorators
In addition to `on_request` and `on_response` middleware, there are also decorators such as `on_connect` and `on_close`.

```python
@app.on_close
async def on_close(**server):
    print('=== CLOSED ===')
```
