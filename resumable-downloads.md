---
layout: page
title: Resumable Downloads
---

The key to getting static files resumable during download is to implement `Accept-Ranges`, `Content-Range` and related support.

So the client/browser can request partial files with `Range`, e.g. `Range: bytes=500-999`.

This is also what makes video files *seekable* by standard players in browsers.

A helper, `response.sendfile()` comes for this purpose. Also with **multipart ranges** support.

Usage:

```python
# http://localhost:8000/download/myvideo.mp4

@app.route('/download/myvideo.mp4')
async def download(**server):
    # video file location
    path = '/home/Videos/myvideo.mp4'

    await server['response'].sendfile(path, content_type='video/mp4')

    # optional, to enable keep-alive
    return True
```
Note that this built-in feature *just works*.
It does not use a shiny module like `aiofiles` for the sake of simplicity.
