---
layout: default
title: HTTP Exceptions
parent: Reference
has_children: true
has_toc: true
---

You can raise an [HTTPException](https://github.com/nggit/tremolo/blob/main/tremolo/exceptions.py) inside [handlers](/handlers.html) or [middlewares](/middleware.html).

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

## Custom HTTP Exceptions
You can also create your custom `HTTPException`:

```python
from tremolo.exceptions import HTTPException

# ...

raise HTTPException(
    'Site down for maintenance',
    code=503,
    message='Under Maintenance'
)
```
