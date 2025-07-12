---
layout: default
title: Request object
parent: Reference
has_children: true
has_toc: true
---

`server['request']` is basically an instantiation of [HTTPRequest](https://github.com/nggit/tremolo/blob/main/tremolo/lib/http_request.py) class. It gives you access to an HTTP request.

Here are some of interesting objects in addition to those in [Headers and Cookies](/tremolo-docs/basics/headers.html) and [Body and POST](/tremolo-docs/basics/body.html):

```python
@app.route('/hello')
async def hello_world(request):
    print('REMOTE_ADDR:',    request.ip)
    print('HTTP_HOST:',      request.host)
    print('REQUEST_METHOD:', request.method)
    print('REQUEST_SCHEME:', request.scheme)
    print('REQUEST_URI:',    request.url)
    print('PATH:',           request.path)
    print('QUERY:',          request.query)
    print('QUERY_STRING:',   request.query_string)
    print('VERSION:',        request.version)

    yield b'Hello'
    yield b'World!'
```

`print` result on http://localhost:8000/hello?a=1&b=2

```
REMOTE_ADDR:    b'127.0.0.1'
HTTP_HOST:      b'localhost:8000'
REQUEST_METHOD: b'GET'
REQUEST_SCHEME: b'http'
REQUEST_URI:    b'/hello?a=1&b=2'
PATH:           b'/hello'
QUERY:          {'a': ['1'], 'b': ['2']}
QUERY_STRING:   b'a=1&b=2'
VERSION:        b'1.1'
```

Note that Tremolo focuses on *bytes-like*, **except** in some dict objects like `request.query`, `request.cookies`, and `request.form()`.

This is for maximum interoperability, to reduce the possibility of encoding - decoding back and forth, etc. With the tradeoff of being a bit inconvenient.

FYI, `request.ip` is not security wise. It will take the ip address from `X-Forwarded-For` / HTTP header when available.

It only becomes handy when Tremolo is deployed behind a proxy server, otherwise `request.client[0]` should be used.
