---
layout: page
title: Logging
---

If you want to print your log messages, you can make use of `server['logger']` within [handlers](handlers.html) or [middlewares](middlewares.html).
It is a Python's [Logger](https://docs.python.org/3/library/logging.html) object.

{: .warning }
You are responsible for sanitizing input to the logger.

```python
from tremolo import Tremolo

app = Tremolo()

@app.route('/hello')
async def hello_world(**server):
    logger = server['logger']

    logger.info('Current page: /hello')
    return 'Hello world!', 'latin-1'


if __name__ == '__main__':
    app.run('0.0.0.0', 8000, debug=True)
```
