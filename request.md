---
layout: page
nav_order: 4
title: Request Object
---

`server['request']` is basically an instantiation [HTTPRequest](https://github.com/nggit/tremolo/blob/master/tremolo/lib/http_request.py)

Here are some of available objects:

```python
@app.route('/hello')
async def hello_world(**server):
    print('HTTP_HOST:',      server['request'].host)
    print('REQUEST_METHOD:', server['request'].method)
    print('PATH:',           server['request'].path)
    print('QUERY_STRING:',   server['request'].query)
    print('VERSION:',        server['request'].version)

    yield b'Hello'
    yield b'World!'
```

`print` result on http://localhost:8000/hello?a=1&b=2

```
HTTP_HOST:      bytearray(b'localhost:8000')
REQUEST_METHOD: bytearray(b'GET')
PATH:           bytearray(b'/hello?a=1&b=2')
QUERY_STRING:   {'a': ['1'], 'b': ['2']}
VERSION:        bytearray(b'1.1')
```
