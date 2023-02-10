---
layout: page
nav_order: 3
title: Routing
---

Tremolo has no difficult abstractions, and never will. You don't need to learn a lot of trivial things. All you need to learn is Python.

Tremolo uses only basic routing and built-in [re](https://docs.python.org/3/library/re.html) module.

## Basic
The following will handle
http://example.com/hello and query string (if any).

```python
@app.route('/hello')
async def hello_world(**server):
    yield b'Hello '
    yield b'world!'
```

## Regular expression
To be recognized as a regex string, at least the path must start with `^` or end with `$` character.

The following will match for example with http://example.com/page/12, http://example.com/page/101, and so on.

```python
@app.route(r'^/page/(?P<page_id>\d+)')
async def my_page(**server):
    request = server['request']
    page_id = request.params['url'].get('page_id', b'')

    """Tremolo often uses bytes-like objects as is,
    rather than converting to str or int.
    so do not assume page_id is an int.
    """
    yield b'You are on page ' + page_id
```

The regex syntax above uses *named groups* which you can learn more about at [https://docs.python.org/3/library/re.html#re.Match.groupdict](https://docs.python.org/3/library/re.html#re.Match.groupdict)

You can always check what kind of data is received if using regex in `request.params['url']`. It's a dict object.
