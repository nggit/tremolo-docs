---
layout: page
title: Security and Tips
---

It should be noted that Tremolo does not validate HTTP requests by default.
Tremolo is a *microframework*, it tends to accept data and parse headers as is.

Some objects like `request.method` may contain anything. This is a design decision that emphasizes flexibility. Not an unentional security issue.

Validation must be done on your side. For example by using the [Middleware](middleware.html).

Deploying Tremolo behind a CDN like Cloudflare, or using a reverse proxy / TLS termination proxy like Nginx is preferred. It can help mitigate some malicious header attacks like Null-byte injection, etc.

## Avoid high memory consumptions
You should be careful when using `request.body()`. It's not memory wise. Consider using `request.stream()` instead. Otherwise, you have to set [client_max_body_size](configuration.html#client_max_body_size) to a lower best value.

When using `request.form()`, you can limit how much data that allowed to enter internal form parser. You can set it with the `max_size` argument.

If you sure / only need short amount of form data, eg. under 64KiB, you can do the following:

```python
form_data = await request.form(max_size=65536)
```

Note that if the coming request body higher than the `max_size`, it will raise `ValueError`.
The default `max_size` is 8MiB.

Lowering its value will help prevent DoS attacks or unexpected memory consumptions.
