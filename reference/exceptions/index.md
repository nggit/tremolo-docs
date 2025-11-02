---
layout: default
title: HTTP Exceptions
parent: Reference
has_children: false
has_toc: false
---

You can raise an [HTTPException](https://github.com/nggit/tremolo/blob/main/tremolo/exceptions.py) inside [handlers](/tremolo-docs/basics/handlers.html) or [middlewares](/tremolo-docs/basics/middleware.html).

Consider the following code:

```python
from tremolo import Tremolo
from tremolo.exceptions import ServiceUnavailable

app = Tremolo()


@app.route('/hello')
async def hello_world(**server):
    raise ServiceUnavailable

    return 'Hello, World!'


if __name__ == '__main__':
    app.run('0.0.0.0', 8000)
```

A `Hello, World!` will not be displayed, but rather `Service Unavailable` as follows (with header):

```
HTTP/1.1 503 Service Unavailable
Content-Type: text/html; charset=utf-8
Content-Length: 19
Connection: close
Date: Wed, 12 Apr 2023 22:48:40 GMT
Server: Tremolo

Service Unavailable
```

You can fine-tune the `ServiceUnavailable`:

```python
raise ServiceUnavailable(
    'Site down for maintenance',
    message='Under Maintenance',
    content_type='text/plain'
)
```
So it will result as follows:

```
HTTP/1.1 503 Under Maintenance
Content-Type: text/plain
Content-Length: 25
Connection: close
Date: Wed, 12 Apr 2023 23:01:08 GMT
Server: Tremolo

Site down for maintenance
```

## Handling/hooking exceptions
Here is an example of how to *catch-all* exceptions in one handler:

```python
@app.error(400)  # internal BadRequest
@app.error(404)  # internal NotFound
@app.error(405)  # internal MethodNotAllowed (CBV)
@app.error(500)
async def all_error(exc, response, **server):
    # response.set_status(exc.code, exc.message)
    response.set_content_type('text/plain')
    return str(exc)

```

Other/Custom exceptions like `ValueError` can be represented by `code` 500.

If you don't want to change the body, just return `None`. You still have the chance to modify `exc`:

```python
@app.error(500)
async def all_error(exc, **server):
    if exc.code == 403:
        exc.content_type = 'text/plain'

```

### Note
- Won't be executed when headers already sent
- Only `return` str/bytes is supported, `yield` is not
- Must accept `**server`/`**kwargs`
- `exc` is an `HTTPException`, a *wrapped* exception rather than original.
  Use `exc.__cause__` to access the original exception

## Create and raise custom HTTPException
Not all `HTTPException`s are defined by Tremolo. Only the common ones and those used internally, such as `Forbidden`, `BadRequest`, etc.

But you can always create custom classes by subclassing `HTTPException`. And if they are raised, they will still be hookable via `@app.error(code)`, as long as they use a `code` in the range **400** - **511**.

Here is the minimal form:
```python
from tremolo.exceptions import HTTPException


class PaymentRequired(HTTPException):
    code = 402
    message = 'Payment Required'
    content_type = 'text/html; charset=utf-8'  # optional

```
