---
layout: page
title: response.set_header()
parent: Response object
grand_parent: Reference
nav_order: 4
---

To set a custom header to the [response.headers](/reference/response/headers.html).
This will overwrite the existing header (if any).

Example:
```python
# ...

@app.route('/hello')
async def hello_world(request, response):
    response.set_header('X-Powered-By', 'bar')

    return 'Hello, World!'

# ...
```
