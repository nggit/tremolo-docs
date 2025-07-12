---
layout: default
title: WebSocket object
parent: Reference
has_children: false
has_toc: false
---

Tremolo has a built-in, minimal implementation of [WebSocket](https://en.wikipedia.org/wiki/WebSocket).

To enable WebSocket support for a [handler](/tremolo-docs/basics/handlers.html), use the `websocket=None` placeholder as follows:

```python
@app.route('/ws')
async def ws_handler(websocket=None):
    if websocket is None:
        # show a not found page
        raise NotFound

    # an upgrade request is received.
    # accept it by sending the "101 Switching Protocols"
    await websocket.accept()

    while True:
        message = await websocket.receive()
        # send back the received message
        await websocket.send(f'You said: {message}')
```

Alternatively, you can use *async iterator* instead of the `while` loop. This will also perform `websocket.accept()` under the hood. Thus calling `websocket.accept()` separately becomes optional:
```python
    # ...

    async for message in websocket:
        await websocket.send(f'You said: {message}')
```

Here's a working example of a WebSocket Chat: [https://github.com/nggit/tremolo/blob/main/websocket_chat.py](https://github.com/nggit/tremolo/blob/main/websocket_chat.py) .

## Close the connection gracefully
When a client initiates a connection closure or closes a browser tab, Tremolo will close the connection at the transport level **without sending the websocket close code**.

To close the connection gracefully, you can for example use `websocket.close(code=1000)`:

```python
    from tremolo.exceptions import WebSocketClientClosed

    # ...
    await websocket.accept()

    try:
        while True:
            message = await websocket.receive()
            # send back the received message
            await websocket.send(f'You said: {message}')
    except WebSocketClientClosed as exc:
        print(f'the client has closed the connection with code {exc.code}')
        # close the connection accordingly
        await websocket.close()
```

Or just use *async for* without *try - except*, with the difference that you can't customize the closing code.

If you think the built-in WebSocket support does not fulfill the features you expect, or want to use an external WebSocket server, you can disable it with `ws=False` in the framework [configuration](/tremolo-docs/configuration.html#ws).

Or `--no-ws` in the ASGI server configuration.
