---
layout: page
title: Configuration
---

Below are some parameters that can be used in `app.listen()` or `app.run()`.

### host
E.g. `'localhost'`, `'127.0.0.1'`, etc.

In Tremolo, `'::'` means listen on all IPv6 interfaces, but also enables dual-stack support.

### port
E.g. `8000`, `8080`, etc.

### reuse_port
The default is `True`.

### worker_num
The default is `1`.

### thread_pool_size
Number of executor threads per worker/process. Mainly used in [synchronous handlers](/tremolo-docs/basics/handlers.html#synchronous-handlers) as well to assist [Inter-process Synchronization](/tremolo-docs/how-to/inter-process-sync.html#multiple-shared-resources).

The default is `5`.

### limit_memory
Restart the worker if this limit (in KiB) is reached (Linux-only).

Defaults to `0` or unlimited.

### backlog
The default is `100`.

### ssl
If you want to enable https, fill this parameter with a `dict` for example:

```python
ssl={'cert': '/path/to/fullchain.pem', 'key': '/path/to/privkey.pem'}
```

The default is `None`.

### debug
By using the `debug=True`, A backtrace will also included in the error message. You should disable this in production with `debug=False`.

If you do not pass this parameter, the default value is `False`.

### reload
You can set this to `True` to enable worker reloading on code changes. Only Intended for development.

### ws
To Disable built-in WebSocket support, you can set this to `False`. The default is `True` or enabled.

To disable it in the ASGI server mode, use `--no-ws`.

### ws_max_payload_size
Maximum payload size for the built-in WebSocket.

The default is `2 * 1048576`, or **2MiB**.

### log_level
The default is `'DEBUG'` (string). For more info, please check [https://docs.python.org/3/library/logging.html#levels](https://docs.python.org/3/library/logging.html#levels).

### log_fmt
Python's log format. If empty defaults to `'%(message)s'`.

### loop
A fully qualified event loop name. E.g. `'asyncio'` or `'asyncio.SelectorEventLoop'`.
It expects the respective module to already be present.

The default is `'asyncio'`.

### download_rate
Limits the sending speed to the client / download speed per second.

The default is `1048576`, which means **1MiB/s** or **8.39Mbps**.

If you want to increase the value too far from the default value, ideally you should also increase the *buffer_size* too.

You can also apply download rate per [handler](/tremolo-docs/basics/handlers.html) using `rate`.

### upload_rate
Limits the upload / POST speed.

The default is the same as *download_rate*, `1048576`.

### buffer_size
The maximum amount of data at each `request.read()`, and each "write" to the client.
The default is `16384`, or **16KiB**.

On write, the `buffer_size` value is also used to determine the watermark.
With `buffer_size=16384`, the high value of the watermark will be 4x or **65536**, and the low value will be 0.5x or **8192**.

You can also apply `buffer_size` in the [handler](/tremolo-docs/basics/handlers.html) or `response.write()`.

In the case of `response.write()` (may be called multiple times), only the first is considered.

But do not passing different `buffer_size` each time in the `response.write()`.

### client_max_body_size
Maximum body on requests such as POST / file upload.

The default is `2 * 1048576`, or **2MiB**.

### client_max_header_size
The default is `8192`, or **8KiB**.

### max_queue_size
Maximum number of [buffers](#buffer_size) in the queue, for a connection.
Exceeding these limits the connection will be aborted.

The default is `128`.

### request_timeout
The maximum period between when the client connects, until the server receives the data.

The default is `30` seconds.

### keepalive_timeout
Maximum client inactivity time in HTTP keep-alive state.

The default is `30` seconds.

### keepalive_connections
Maximum number of keep-alive connections.

The defaults is `512` (connections/worker). The oldest, or **513th** will be kicked out.

This value often gives false impressions on benchmarks. A good benchmark uses concurrency between 1 - 500. If you want to use concurrency above 1000, you must increase this value, *ulimit*, and possibly [worker_num](#worker_num). Otherwise you will see a lot of connections being dropped.

### app_handler_timeout
The common term for this is "maximum execution time". The default is `120` seconds.

This is a redundant protection. To make sure the task on the handler / ASGI app ends within a certain amount of time, thus preventing poorly written applications, or unwanted never-ending stream scenarios. Especially if it has not been covered by any of the timeouts above.

Example:
```python
@app.route('/')
async def my_handler(**server):
    await asyncio.sleep(123)

    # will be killed before it even returns value
    return 'OK'
```

Note that the WebSocket/upgraded connection will not be affected. And won't work if you are running a blocking function, such as `time.sleep()` inside it.

### app_close_timeout
This is another handler timeout similar to *app_handler_timeout*, but it is initiated when the client connection is lost and the handler is still running for some reason. The default is `30`.

It is possible that it is initiated while *app_handler_timeout* is still waiting. And the lowest timeout takes precedence.

### shutdown_timeout
Maximum number of seconds to wait after `SIGTERM` is sent to a worker process. During this time it can still accept new connections.
For a quick shutdown you can set this to `0`.

Defaults to `30` (seconds).

### server_name
Set the `Server` field in the response header.

The default is `b'Tremolo'`.
