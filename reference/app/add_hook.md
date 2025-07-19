---
layout: page
title: app.add_hook()
parent: Application object
grand_parent: Reference
nav_order: 3
---

You can add *hooks* with either decorator style or `app.add_hook()`. The latter is more suitable in case you are creating an extension.

```python
@app.on_worker_start
async def func(app, **server):
    print('before serving')
```

behaves the same way as:

```python
app.add_hook(func, name='worker_start', priority=999)
```

Available `name`s: `'worker_start'`, `'worker_stop'`, `'connect'` and `'close'`.

