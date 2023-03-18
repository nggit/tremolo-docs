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

When using `request.form()`, you can limit how much data that allowed to enter internal form parser. You can set it with the `limit` argument.

If you sure / only need short amount of form data, eg. under 64KiB, you can do the following:

```python
form_data = await request.form(limit=65536)
```

Note that if the coming request body higher than the `limit`, it will return an empty `{}`.
The default `limit` is 8MiB.

By lowering its value will help mitigating DoS attacks.
