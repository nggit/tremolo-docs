---
layout: page
title: response.set_content_type()
parent: Response object
grand_parent: Reference
nav_order: 7
---

To set a `Content-Type` header to the [response.headers](/tremolo-docs/reference/response/headers.html).

Example:
```python
# ...

@app.route('/hello')
async def hello_world(request, response):
    response.set_content_type('text/plain')

    return 'Hello, World!'

# ...
```
