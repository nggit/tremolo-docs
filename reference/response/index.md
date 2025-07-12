---
layout: default
title: Response object
parent: Reference
has_children: true
has_toc: true
---

`server['response']` is basically an instantiation of [HTTPResponse](https://github.com/nggit/tremolo/blob/main/tremolo/lib/http_response.py) class.

Here are some of available methods:

```python
set_content_type(content_type=b'text/html; charset=utf-8')
set_cookie(name, value='', expires=0, path='/', domain=None, secure=False, httponly=False, samesite=None)
set_header(name, value='')
set_status(status=200, message=b'OK')
```

You can use the `write()` or `end()` method instead of [yield / return](/tremolo-docs/basics/yield.html) to send the HTTP response body. Although the latter is preferred.

You can imagine that `write()` is like `yield`, it can be called more than once.
Whereas `end()` is like `return`, which is only allowed to be called once.

Whether `yield`, `return`, `write()`, or `end()` will **implicitly** send the HTTP header once.
Normally, You don't need to set anything except a few things like response status, `Content-Type`, `Set-Cookie` by using the `set_*` method above (should be called earlier).

If you need to send RAW responses, consider using the `send()` method. This will not implicitly send the HTTP header. The `set_*` method call will be meaningless.

Here's a summary of the comparison:

| Methods                              | Implicit HTTP Header | Info                                                                                |
|:-------------------------------------|:---------------------|:------------------------------------------------------------------------------------|
| yield b'Hello'                       | Yes                  |                                                                                     |
| return 'Hello'                       | Yes                  |                                                                                     |
| await response.write(b'Hello')       | Yes                  |                                                                                     |
| await response.write(b'')            | Yes                  | May send as `b'0\r\n\r\n'` if *chunked* is enabled                                  |
| await response.write(None)           | Yes                  | Send the prepared HTTP header then call `send(None)`                                |
| await response.end(b'Hello')         | Yes                  |                                                                                     |
| await response.end()                 | Yes                  | Same as calling `write(b'')` then `send(None)`                                      |
| await response.send(b'HTTP/1.1 ...') | No                   | Send RAW responses, Non-HTTP is possible                                            |
| await response.send(None)            | No                   | Same as `response.close(keepalive=True)`                                            |
| response.close(keepalive=True)       | No                   | Mark the end of response. Connection will be closed, but respect HTTP keep-alive    |
| response.close()                     | No                   | Just close the connection. Called automatically in handlers when `None` is returned |

See also: [response.sendfile()](/tremolo-docs/how-to/resumable-downloads.html)
