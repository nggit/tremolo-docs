---
layout: page
title: WebSocket
---

Tremolo has a built-in, minimal implementation of [WebSocket](https://en.wikipedia.org/wiki/WebSocket).

To enable WebSocket support for a [handler](handlers.html), use the `websocket=None` placeholder as follows:

```python
@app.route('/ws')
async def ws_handler(websocket=None, **server):
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

Alternatively, you can use *async iterator* instead of the `while` loop. This will also perform `websocket.accept()` under the hood:
```python
    # ...

    async for data in websocket:
        await websocket.send(f'You said: {message}')
```

Here's a working example of a WebSocket Chat:

```python
#!/usr/bin/env python3

import time

from tremolo import Tremolo
from tremolo.exceptions import BadRequest

app = Tremolo()


@app.on_request
async def middleware_handler(**server):
    for char in b'&<>"\'':
        if char in server['request'].host:
            raise BadRequest('illegal host')

    # add more validations, CORS headers, etc if needed


@app.route('/')
async def ws_handler(websocket=None, request=None, stream=False, **_):
    """A hybrid handler.

    Normally, you should separate http:// and ws:// respectively.
    """
    if websocket is not None:
        # an upgrade request is received.
        # accept it by sending the "101 Switching Protocols"
        await websocket.accept()

        while True:
            message = await websocket.receive()
            # send back the received message
            await websocket.send(
                '[{:s}] Guest{:d}: {:s}'.format(
                    time.strftime('%H:%M:%S'), request.client[1], message)
            )

    # not an upgrade request. show the html page
    yield b"""\
    <!DOCTYPE html><html lang="en"><head><title>WebSocket Chat</title></head>
    <body>
        <h1>WebSocket Chat</h1>
        <form>
            <input type="text" id="message" autocomplete="off" />
            <button type="button" id="send">Send</button>
        </form>
        <ul id="messages"></ul>
        <script>
    """
    ws_scheme = b'ws'

    if request.scheme == b'https':
        ws_scheme = b'wss'

    yield b"\
        var socket = new WebSocket('%s://%s/');" % (ws_scheme, request.host)
    yield b"""
            var messages = document.getElementById('messages');
            var sendButton = document.getElementById('send');

            socket.onmessage = function(event) {
                var message = document.createElement('li');
                message.textContent = event.data;
                messages.insertBefore(message, messages.firstChild);
            };

            sendButton.onclick = function() {
                var message = document.getElementById('message');

                if (message) {
                    socket.send(message.value);
                    message.value = '';
                }
            };
        </script>
    </body>
    </html>
    """

if __name__ == '__main__':
    # don't forget to disable debug and reload on production!
    app.run('0.0.0.0', 8000, debug=True, reload=True)
```

If you think the built-in WebSocket support does not fulfill the features you expect, or want to use an external WebSocket server, you can disable it with `ws=False` in the framework [configuration](configuration.html#ws).

Or `--no-ws` in the ASGI server configuration.
