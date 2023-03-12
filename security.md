---
layout: page
title: Security
---

It should be noted that Tremolo does not validate HTTP requests by default.
Tremolo is a *microframework*, it tends to accept data and parse headers as is.

Some objects like `request.method` may contain anything.

This is a design decision that emphasizes flexibility. Not a weakness.

Validation must be done on your side. For example by using the [Middleware](middleware.html).

Deploying Tremolo behind a CDN like Cloudflare, or using a reverse proxy / TLS termination proxy like Nginx is preferred. It can help mitigate malicious header attacks.
