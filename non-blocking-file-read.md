---
layout: page
title: Non-blocking File Read
---

Using an innocent way like this:

```python
@app.route('/my/url/big.data')
async def my_big_data(content_type='application/octet-stream', **server):
    buffer_size = server['context'].options['buffer_size']

    with open('/my/folder/big.data', 'rb') as f:
        chunk = True

        while chunk:
            chunk = f.read(buffer_size)

            yield chunk
```
normally will **not** block the event loop. As long as the blocking io call `open()` and `f.read()` is quick (ie. You have a fast drive).

The subsequent calls to it (inside `while chunk/True`) is really safe. Thanks to the download speed limiter.

Internally, Tremolo will suspend each chunk / `yield` at some amount of time depending on the given speed setting.
The lower speed setting given, the more chance for other coroutines to run.

Using a library like [awaiter](https://github.com/nggit/awaiter) should gives better performance.
