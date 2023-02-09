---
layout: page
nav_order: 6
title: Headers and Cookies
---

## How to get cookies?

Consider the following HTTP request:

```
GET /hello HTTP/1.1
Host: localhost:8000
User-Agent: curl/7.74.0
Accept: */*
Cookie: a=123
```

You can get the `Cookie` value with:

```python
print(
    server['request'].headers.get(b'cookie')
)
```

A *bytes-like* object will be returned:

```
bytearray(b'a=123')
```

What if there are two cookies?

```
Cookie: a=123
Cookie: a=xyz
```

In this case a `list` object will be returned:

```
[bytearray(b'a=123'), bytearray(b'a=xyz')]
```

There is a better way to get cookies with:

```python
print(
    server['request'].cookies.get('a')
)
```

It then may return:

`['123']` or `['123', 'xyz']`

## How to set cookies?

There are two ways.

First, using `set_header`:

```python

server['response'].set_header('Cookie', 'a=xyz')
```

And the proper way is using `set_cookie`. It provides a convenient parameter for easily set cookie expiration, etc.

```python
server['response'].set_cookie('a', 'xyz', expires=3600)
```
