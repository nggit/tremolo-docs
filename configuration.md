---
layout: page
title: Configuration
---

Below are some parameters that can be used in `app.listen()` or `app.run()`.

### host
E.g. `localhost`, `127.0.0.1`, etc.

### port
E.g. `8000`, `8080`, etc.

### reuse_port
The default is `True`.

### worker_num
The default is `1`.

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

### log_level
The default is `'DEBUG'` (string). For more info, please check [https://docs.python.org/3/library/logging.html#levels](https://docs.python.org/3/library/logging.html#levels).

### download_rate
Limits the sending speed to the client / download speed per second.

The default is `1048576`, which means **1MiB/s** or **8.39Mbps**.

You can also apply download rate per [handler](handlers.html) using `rate`.

### upload_rate
Limits the upload / POST speed.

The default is the same as *download_rate*, `1048576`.

### buffer_size
The maximum amount of data at each `request.read()`, and each "write" to the client.
The default is `16384`, or **16KiB**.

On write, the `buffer_size` value is also used to determine the watermark.
With `buffer_size=16384`, the high value of the watermark will be 4x or **65536**, and the low value will be 0.5x or **8192**.

You can also apply `buffer_size` in the [handler](handlers.html) or `response.write()`.

In the case of `response.write()` (may be called multiple times), only the first is considered.

But do not passing different `buffer_size` each time in the `response.write()`.

### client_max_body_size
Maximum body on requests such as POST / file upload.

The default is `2 * 1048576`, or **2MiB**.

### request_timeout
The maximum period between when the client connects, until the server receives the data.

The default is `30` seconds.

### keepalive_timeout
Maximum client inactivity time in HTTP keep-alive state.

The default is `30` seconds.

### server_name
Set the `Server` field in the response header.

The default is `b'Tremolo'`.
