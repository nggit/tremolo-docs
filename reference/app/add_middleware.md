---
layout: page
title: app.add_middleware()
parent: Application object
grand_parent: Reference
nav_order: 2
---

You can add [middlewares](/tremolo-docs/basics/middleware.html) with either decorator style or `app.add_middleware()`. The latter is more suitable in case you are creating an extension.

```python
@app.on_request
async def func(request, **server):
    print('before handler')
```

behaves the same way as:

```python
app.add_middleware(func, name='request', priority=999)
```

Available `name`s: `'request'` and `'response'`.

