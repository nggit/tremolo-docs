---
layout: page
nav_order: 3
title: Routing
parent: Basics
---

Tremolo uses only basic routing and built-in [re](https://docs.python.org/3/library/re.html) module. There are no difficult abstractions, and never will.

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
async def my_page(request):
    page_id = request.params.path.get('page_id', b'1')

    # Tremolo often uses bytes-like objects as is,
    # rather than converting to str or int.
    # so do not assume page_id is an int.
    yield b'You are on page ' + page_id

```

The regex syntax above uses *named groups* which you can learn more about at [https://docs.python.org/3/library/re.html#re.Match.groupdict](https://docs.python.org/3/library/re.html#re.Match.groupdict)

You can always check what kind of data is received if using regex in `request.params.path`. It's a dict object.

Alternatively, the matching results will be populated in `kwargs` as long as the names don't conflict with server objects. So this style works too:
```python
@app.route(r'^/page/(?P<page_id>\d+)')
async def my_page(request, page_id=b'1', **server):
    yield b'You are on page ' + page_id

```

{: .note }
*regex*-based routing pattern will be compared against `request.url` not `request.path`, meaning the match involves also query string. But for convenience, `$` can still match the end of `request.path`.

## Class-based views
From the beginning, Tremolo exclusively supports only `@app.route`.

`@app.get`, `@app.post`, etc. are not implemented to prevent unnecessary lines of code or redundant API/methods.

To compensate, As of [#300](https://github.com/nggit/tremolo/pull/300) `@app.route` supports decorating classes as well, so you can separate methods more cleanly.

**OLD:**
```python
@app.route('/hello')
async def hello_world(request, **server):
    if request.method == b'GET':
        return 'Hello, World!'

    if request.method == b'POST':
        return await request.body()

    raise MethodNotAllowed
```

**NEW:**
```python
@app.route('/hello')
class HelloWorld:
    async def get(self, **server):
        return 'Hello, World!'

    async def post(self, request, **server):
        return await request.body()

```

{: .note }
If the method is not implemented, the request will fall on the default handler `405 Method Not Allowed`.

## Passing arbitrary options
Any arbitrary options in `@app.route(, **options)` or `app.add_route(, **options)` will be received in the CBV class, in `__init__(, **kwargs)`.

This allows complex use cases.

```python
class HelloModel:
    def get_message(self):
        return 'Hello, clean architecture!'

    def set_message(self, message):
        self.msg = message


@app.route('/hello', model=HelloModel)
class HelloWorld:
    def __init__(self, model, **kwargs):
        self.model = model()

    async def get(self, **server):
        return self.model.get_message()

    async def post(self, request, **server):
        self.model.set_message(await request.body())
        return 'OK'


# reuse?
app.add_route(HelloWorld, '/hello101', model=AnotherModel)

```

## Custom 40x/50x page
You can hook all exceptions raised by the internal framework such as `NotFound`, as well as by user code.

```python
@app.error(404)
async def my_error_page(**server):
    return 'This is my custom 404 page.'

```

You can find more details about this in [the reference](/tremolo-docs/reference/exceptions/#handlinghooking-exceptions).
