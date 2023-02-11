---
layout: page
nav_order: 2
title: yield vs return
---

In Tremolo, `yield` only accepts *bytes-like* objects like `bytes` or `bytearray`. Whereas `return` accepts both `str` and *bytes-like* objects.

Both `yield` and `return` are used to send the response body to the client. Each `yield` will usually be a [chunked HTTP response](https://en.wikipedia.org/wiki/Chunked_transfer_encoding).
