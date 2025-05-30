---
layout: page
title: response.set_cookie()
parent: Response object
grand_parent: Reference
nav_order: 5
---

To set a `Set-Cookie` header to the [response.headers](/tremolo-docs/reference/response/headers.html).

Signature:
```python
response.set_cookie(name, value='', expires=0, path='/', domain=None,
                    secure=False, httponly=False, samesite=None)
```

Example:
```python
# ...

@app.route('/hello')
async def hello_world(request, response):
    response.set_cookie('foo', 'bar')

    return 'Hello, World!'

# ...
```
