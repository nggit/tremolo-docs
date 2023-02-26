---
layout: page
title: Non-blocking File Read
---

Using an innocent way like this:

```python
@app.route('/my/url/big.data')
async def my_big_data(content_type='application/octet-stream', **server):
    buffer_size = server['options']['buffer_size']

    with open('/my/folder/big.data', 'rb') as f:
        chunk = True

        while chunk:
            chunk = f.read(buffer_size)

            yield chunk
```
is not blocking (although it's inefficient). Thanks to the download speed limiter.

Internally, Tremolo will suspend each chunk / `yield` at some amount of time depending on the given speed setting.
The lower speed setting given, the more chance for other coroutines to run.

Using a library like `aiofile` should gives better performance. Or reading the file in a separate thread with [loop.run_in_executor](https://docs.python.org/3/library/asyncio-eventloop.html#asyncio.loop.run_in_executor):

```python
import concurrent.futures

@app.route('/my/url/file.mp4')
async def my_video(content_type='video/mp4', **server):
    loop = server['loop']
    buffer_size = server['options']['buffer_size']

    with open('/my/folder/file.mp4', 'rb') as f:
        with concurrent.futures.ThreadPoolExecutor(max_workers=1) as executor:
            chunk = True

            while chunk:
                chunk = await loop.run_in_executor(executor, f.read, buffer_size)

                yield chunk
```

Or a slight variation:

```python
import concurrent.futures

@app.route('/my/url/file.mp4')
async def my_video(content_type='video/mp4', **server):
    loop = server['loop']
    buffer_size = server['options']['buffer_size']

    def read_file():
        with open('/my/folder/file.mp4', 'rb') as f:
            chunk = True

            while chunk:
                chunk = f.read(buffer_size)

                yield chunk

    with concurrent.futures.ThreadPoolExecutor(max_workers=1) as executor:
        for data in await loop.run_in_executor(executor, read_file):
            yield data
```
