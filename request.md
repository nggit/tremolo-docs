---
layout: page
nav_order: 5
title: Request Object
---

`server['request']` is basically an instantiation of [HTTPRequest](https://github.com/nggit/tremolo/blob/master/tremolo/lib/http_request.py) class.

Here are some of available objects in addition to those in [Headers and Cookies](headers.html) and [Body and POST](body.html):

```python
@app.route('/hello')
async def hello_world(**server):
    request = server['request']

    print('REMOTE_ADDR:',    request.ip)
    print('HTTP_HOST:',      request.host)
    print('REQUEST_METHOD:', request.method)
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
HTTP_HOST:      bytearray(b'localhost:8000')
REQUEST_METHOD: bytearray(b'GET')
REQUEST_URI:    bytearray(b'/hello?a=1&b=2')
PATH:           bytearray(b'/hello')
QUERY:          {'a': ['1'], 'b': ['2']}
QUERY_STRING:   bytearray(b'a=1&b=2')
VERSION:        bytearray(b'1.1')
```

Note that tremolo focuses on *bytes-like*, except in some dict objects like `request.query`, `request.cookies`, and `request.form()`.

We do this for the sake of interoperability, to reduce the possibility of encoding - decoding back and forth, etc. With the tradeoff of being a bit inconvenient.

FYI, `request.ip` is not security wise. It will take the ip address from `X-Forwarded-For` / HTTP header when available.

It only becomes handy when Tremolo is deployed behind a proxy server, otherwise `request.client[0]` should be used.
