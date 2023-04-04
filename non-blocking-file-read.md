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

Using a library like `aiofiles` should gives better performance.

Or reading the file in a separate thread with [loop.run_in_executor](https://docs.python.org/3/library/asyncio-eventloop.html#asyncio.loop.run_in_executor):

```python
from concurrent.futures import ThreadPoolExecutor

@app.route('/my/url/file.mp4')
async def my_video(content_type='video/mp4', **server):
    loop = server['loop']
    buffer_size = server['context'].options['buffer_size']

    def read_file():
        with open('/my/folder/file.mp4', 'rb') as f:
            chunk = True

            while chunk:
                chunk = f.read(buffer_size)

                yield chunk

    with ThreadPoolExecutor(max_workers=1) as executor:
        gen = read_file()
        data = True

        while data:
            data = await loop.run_in_executor(executor, gen.__next__)

            yield data
```

Note that this method may add some overhead compared to using the standar method.
