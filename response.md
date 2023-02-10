---
layout: page
nav_order: 6
title: Response Object
---
`server['response']` is basically an instantiation of [HTTPResponse](https://github.com/nggit/tremolo/blob/master/tremolo/lib/http_response.py) class.

Here are some of available methods:

```python
set_content_type(content_type=b'text/html; charset=utf-8')
set_cookie(name, value='', expires=0, path='/', domain=None, secure=False, httponly=False, samesite=None)
set_header(name, value='')
set_status(status=200, message=b'OK')
```
