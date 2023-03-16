---
layout: page
title: Security and Tips
---

It should be noted that Tremolo does not validate HTTP requests by default.
Tremolo is a *microframework*, it tends to accept data and parse headers as is.

Some objects like `request.method` may contain anything.

This is a design decision that emphasizes flexibility. Not a weakness.

Validation must be done on your side. For example by using the [Middleware](middleware.html).

Deploying Tremolo behind a CDN like Cloudflare, or using a reverse proxy / TLS termination proxy like Nginx is preferred. It can help mitigate malicious header attacks.

## Avoid high memory consumptions
You should be careful when sing `request.body()`. It's not memory wise. Consider using `request.read()` instead. Otherwise, you have to set `client_max_body_size` to a lower best value.

When using `request.form()`, you can limit how much data that allowed to enter internal form parser.

You can set it with `limit` argument. If you sure / only need short amount of form data, eg. 16KiB, you can do the following:

```python
form_data = await request.form(limit=16384)
```

The default is 8MiB. By lowering its value will help mitigating DoS attacks.
