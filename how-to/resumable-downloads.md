---
layout: page
title: Resumable Downloads
parent: How-To
---

The key to getting static files resumable during download is to implement `Accept-Ranges`, `Content-Range` and related support.

So the client/browser can request partial files with `Range`, e.g. `Range: bytes=500-999`.

This is also what makes video files *seekable* by standard players in browsers (improving user experience).

A helper, `response.sendfile()` comes for this purpose. Also with a **multipart ranges** support.

Usage:

```python
# http://localhost:8000/download/myvideo.mp4

@app.route('/download/myvideo.mp4')
async def download(response):
    # video file location
    path = '/home/user/Videos/myvideo.mp4'

    await response.sendfile(path, content_type='video/mp4')

    # optional, to enable keep-alive
    return True
```

The default `content_type` is `'application/octet-stream'`. You must manually specify it for the browser to render properly. Please refer to [https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/MIME_types/Common_types](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/MIME_types/Common_types)
 .
