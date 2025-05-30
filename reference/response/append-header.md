---
layout: page
title: response.append_header()
parent: Response object
grand_parent: Reference
nav_order: 3
---

To append a custom header to the [response.headers](/reference/response/headers.html).

Example:
```python
# ...

@app.route('/hello')
async def hello_world(request, response):
    response.append_header('X-Powered-By', 'foo')

    return 'Hello, World!'

# ...
```
