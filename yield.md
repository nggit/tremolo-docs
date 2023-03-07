---
layout: page
nav_order: 2
title: yield vs return
---

In Tremolo, `yield` only accepts *bytes-like* objects like `bytes` or `bytearray`. Whereas `return` accepts both `str` and *bytes-like* objects.

Both `yield` and `return` are used to send the response body to the client. Each `yield` will usually be a [chunked HTTP response](https://en.wikipedia.org/wiki/Chunked_transfer_encoding).

{: .note }
Instead of using *return*, you can also use the *end()* method of the [Response object](response.html)

It will be faster than using `return` because it skips some logic. But only `bytes-like` objects are supported.

```python
@app.route('/hello-x')
async def hello_world_x(**server):
    response = server['response']

    await response.end(b'Hello world!')
```

Note that both `return` and `end()` are only suitable for sending relatively small amounts of data at a time.

## Send RAW data
In some cases you may want to send RAW data. You can use the `send()` method of the [Response object](response.html).

It's a lower-level method, which only supports `bytes-like` objects.

```python
@app.route('/hello-r')
async def hello_world_r(**server):
    response = server['response']

    await response.send(b'HTTP/1.1 200 OK\r\nConnection: close\r\n\r\n')
    await response.send(b'Hello World!')
```
