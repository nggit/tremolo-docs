---
layout: page
title: response.set_status()
parent: Response object
grand_parent: Reference
nav_order: 6
---

To set the HTTP status code.

Example:
```python
# ...

@app.route('/hello')
async def hello_world(request, response):
    response.set_status(403, 'Forbidden')

    return 'Hello, World!'

# ...
```
