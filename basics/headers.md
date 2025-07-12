---
layout: page
nav_order: 7
title: Headers and Cookies
parent: Basics
---

You can **get** Headers and Cookies with the [Request Object](/tremolo-docs/reference/request/), and how to **set** it is by using the  [Response Object](/tremolo-docs/reference/response/).

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
    request.headers.get(b'cookie')
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
    request.cookies.get('a')
)
```

It then may return:

`['123']` or `['123', 'xyz']`

## How to set cookies?

There are two ways.

First, using the `set_header` of the [Response Object](/tremolo-docs/reference/response/):

```python

response.set_header('Set-Cookie', 'a=xyz')
```

And the proper way is by using the `set_cookie`. It provides a convenient parameter for easily set cookie expiration, etc.

```python
response.set_cookie('a', 'xyz', expires=3600)
```

## Get the values of multiple fields as a list
In HTTP, some fields are allowed to have the same name, such as `Cookie`, `Accept-Encoding`, etc.

In such special cases, `request.headers.getlist()` can be used to retrieve all values at once.

In the request header as follows:

```
Accept-Encoding: br
Accept-Encoding: gzip, deflate
```

`print(request.headers.getlist(b'accept-encoding'))` will return a list with three items:

```
[b'br', b'gzip', d'deflate']
```
