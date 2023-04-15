---
layout: page
title: Configuration
---

Below are some parameters that can be used in `listen()` or `run()`.

### host
E.g. `localhost`, `127.0.0.1`, etc.

### port
E.g. `8000`, `8080`, etc.

### reuse_port
The default is `True`

### worker_num
The default is `1`

### ssl
If you want to enable https, fill this parameter with `dict` for example:

```python
ssl={'cert': '/path/to/fullchain.pem', 'key': '/path/to/privkey.pem'}
```

The default is `None`

### debug
`debug=True` will show a backtrace if there is an error. You should disable this in production with `debug=False`.

if you do not pass this parameter, the default is `debug=False`.

### log_level
The default is `log_level='DEBUG'`. For more info, please check [https://docs.python.org/3/library/logging.html#levels](https://docs.python.org/3/library/logging.html#levels)

### download_rate
Limits the sending speed to the client / download speed per second.

The default is `1048576`, which means 1MiB/s or 8.39Mbps

### upload_rate
Limits the upload / POST speed

The default is the same as *download_rate*, `1048576`

### buffer_size
The maximum amount of data at each `request.read()` and "send" to the client.
The default is `16384`, or 16KiB.

On send, the *buffer_size* value is also used to determine the watermark.
With `buffer_size=16384`, the high value of the watermark is 4x which is **65536**, and the low value is 0.5x which is **8192**.

### client_max_body_size
Maximum body on requests such as POST / file upload.

The default is `client_max_body_size=2*1048576`, or 2MiB.

### request_timeout
The maximum period between when the client connects, until the server receives the data.

The default is `30` seconds.

### keepalive_timeout
Maximum client inactivity time in HTTP keep-alive state.

The default is `30` seconds.

### server_name
The default is `b'Tremolo'`
