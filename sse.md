---
layout: page
title: Server-Sent Events
---

Tremolo provides a [Server-Sent Events (SSE)](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events) helper to send continuous data over HTTP.

Just like the [WebSocket way](websocket.html), you only need to put the `sse=None` placeholder in the [handlers](handlers.html) to be able to send SSE data.

```python
@app.route('/sse')
async def sse_handler(sse=None, **server):
    if sse is None:
        # for some reason, the sse object was not created
        # due to an invalid request
        raise BadRequest

    await sse.send('Hel\nlo', event='greeting')
    await sse.send('World!', event='greeting')
    await sse.close()
```

The client should see the following response body:

```
data: Hel
data: lo
event: greeting

data: World!
event: greeting

```

You may need to add some CORS headers.

Here's a simple example, but not necessarily a good way to do it.

```python
@app.route('/sse')
async def sse_handler(sse=None, **server):
    if sse is None:
        # for some reason, the sse object was not created
        # due to an invalid request
        raise BadRequest

    # set CORS headers with the response object
    sse.response.set_header(b'Access-Control-Allow-Credentials', b'true')

    if b'origin' in sse.request.headers:
        sse.response.set_header(
            b'Access-Control-Allow-Origin', sse.request.headers[b'origin']
        )

    await sse.send('Hel\nlo', event='greeting')
    await sse.send('World!', event='greeting')
    await sse.close()
```
