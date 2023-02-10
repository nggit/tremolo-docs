---
layout: page
nav_order: 7
title: Body and POST
---

**POST** is one of the request methods that has a body. `POST` is usually used to send forms / upload data.

Currently Tremolo does not support `multipart/form-data` only supports `application/x-www-form-urlencoded`.

However, Tremolo supports RAW Body and HTTP chunked upload out of the box.

You can process `multipart/form-data` yourself if necessary.

## Form

Tremolo by default will populate the form if the `POST` data is `application/x-www-form-urlencoded` in:

```python
server['request'].params['post']
```

Here's a simple example of how to handle login form:

```python
import secrets

@app.route('/login')
async def my_login_handler(**server):
    request = server['request']
    response = server['response']

    credentials = 'myuser:mypass'

    try:
        user = request.params['post']['user'][-1]
        password = request.params['post']['password'][-1]

        if secrets.compare_digest('{:s}:{:s}'.format(user, password), credentials):
            return 'Login success!'

    except (KeyError, IndexError):
        response.set_status(400, 'Bad Request')
        return 'Bad request.'

    response.set_status(401, 'Unauthorized')
    return 'Login failed!'
```

Here is the response if you login with `curl -X POST -d 'user=myuser&password=mypass' -i http://localhost:8000/login`

```
HTTP/1.1 200 OK
Date: Fri, 10 Feb 2023 12:16:15 GMT
Server: Tremolo
Transfer-Encoding: chunked
Content-Type: text/html; charset=utf-8
Connection: keep-alive

Login success!
```

## RAW Body upload

`POST` data other than forms should be consumed with `server['response'].read()` which is an *async generator*, or `server['response'].body()` which is a *coroutine object*.

Here's a code example to receive a binary data upload, then save it.

```python
@app.route('/upload')
async def upload(**server):
    with open('/save/to/image_uploaded.png', 'wb') as f:
        # read body chunk by chunk
        async for data in server['request'].read():

            # write to file on each chunk
            f.write(data)

    return 'Done.'
```

or

```python
@app.route('/upload')
async def upload(**server):
    with open('/path/to/image_uploaded.png', 'wb') as f:
        # read body at once
        data = await server['request'].body()

        # write to file
        f.write(data)

    return 'Done.'
```

You can then upload files for example with:

```
curl -X POST -H 'Content-Type: application/octet-stream' --data-binary '@/path/to/image.png' -v http://localhost:8000/upload
```

or


```
curl -X POST -H 'Transfer-Encoding: chunked' -H 'Content-Type: application/octet-stream' --data-binary '@/path/to/image.png' -v http://localhost:8000/upload
```
