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

## Disable Response Streaming on "yield"
By default, every `yield` will be sent to the client immediately, even if it is a single byte. This behavior is noticeable in web browser, not in curl.

The following code will print `Hello, World!` to the browser, each character + 0.2 second delay in sequence:

```python
@app.route('/hello')
async def hello_world(**server):
    for b in b'Hello, World!':
        # put a delay on each character
        await asyncio.sleep(0.2)

        yield bytes([b])
```

![Tremolo yield stream=True](https://raw.githubusercontent.com/nggit/tremolo-docs/main/assets/images/tremolo-yield-stream-true.gif)

`stream=False` can be used to disable response streaming. This will make the yields buffered; meaning performance may improve on multiple yields.

```python
@app.route('/hello')
async def hello_world(stream=False, **server):
    for b in b'Hello, World!':
        # put a delay on each character
        await asyncio.sleep(0.2)

        yield bytes([b])
```

![Tremolo yield stream=False](https://raw.githubusercontent.com/nggit/tremolo-docs/main/assets/images/tremolo-yield-stream-false.gif)

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
