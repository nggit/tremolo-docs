---
layout: page
title: app.add_route()
parent: Application object
grand_parent: Reference
nav_order: 1
---

You can add [route handlers](/tremolo-docs/basics/handlers.html) with either decorator style or `app.add_route()`. The latter is more suitable in case you are creating an extension.

```python
@app.route('/hello')
async def func(request, **server):
    print('handler')
    return 'Hello, World!'
```

behaves the same way as:

```python
app.add_route(func, path='/')
```

The `func` parameter can be a class type for a [CBV](/tremolo-docs/basics/routing.html#class-based-views) usage.

