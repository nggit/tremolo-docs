---
layout: page
nav_order: 8
title: Body and POST
---

**POST** is one of the request methods that comes with a body. `POST` is usually used to send forms / upload data.

Tremolo supports `application/x-www-form-urlencoded` and `multipart/form-data`.

Tremolo also supports RAW Body and HTTP chunked upload out of the box.

## Form

If the `POST` data is `application/x-www-form-urlencoded`, it can be retrieved with:

```python
form_data = await server['request'].form()
```

After that being called (at least once), then the form data will also available at:

```python
server['request'].params['post']
```

The request body will also be **cached**, allowing `request.body()` to be *awaited* afterwards.

Here's a simple example of how to handle login form:

```python
from hmac import compare_digest

@app.route('/login')
async def my_login_handler(**server):
    request = server['request']
    response = server['response']

    credentials = 'myuser:mypass'

    try:
        form_data = await request.form()
        user = form_data['user'][0]
        password = form_data['password'][0]

        if compare_digest('{:s}:{:s}'.format(user, password), credentials):
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

## Multipart
You can *stream* multipart through the `server['request'].files()` *async generator*. Each will return a tuple object `(info, data)`.

```python
@app.route('/upload')
async def upload(**server):
    async for info, data in server['request'].files():
        print(info, data[:12])

    return 'Done.'
```
You can try:
```
curl -X POST -H "Content-Type: multipart/form-data; boundary=myboundary" -F "mytext=dataxxxx" -F "myfile=@file.txt" -F "myfile2=@image.jpg" http://localhost:8000/upload
```

Then the above snippet will print something like:

```python
{'name': 'mytext'} bytearray(b'dataxxxx')
{'name': 'myfile', 'filename': 'file.txt', 'type': 'text/plain'} bytearray(b'datayyyy\n')
{'name': 'myfile2', 'filename': 'image.jpg', 'type': 'image/jpeg'} bytearray(b'\xff\xd8\xff\xe0\x00\x10JFIF\x00\x01')
```

## Stream the request body / receive upload

`POST` data other than forms should be consumed with `server['request'].read()` which is an *async generator*, or `server['request'].body()` which is a *coroutine object*.

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

Note that the default maximum of body is **2MiB**. You can increase it by set the `client_max_body_size` in the `app.run()` for example:
```python
app.run('0.0.0.0', 8000, client_max_body_size=100 * 1048576)
```

Now you should be able to upload files for up to 100MiB in size.

## Read the request body up to a certain size
`read()` or `read(size=None)` is an alias of `stream()`, an *async generator*. If the `size` parameter is provided, it becomes *awaitable*.

```python
data = await server['request'].read(100)
next_data = await server['request'].read(50)
```

It will read **exactly** 150 bytes of the request body. When no more body can be read, an empty `bytearray()` will be returned.

If you pass `size=-1`, it literally means `next(stream())`, or it will read up to the maximum [buffer_size](https://nggit.github.io/tremolo-docs/configuration.html#buffer_size), typically **<= 16KiB**. To read the entire body use `await request.body()`.

Note that `read()`, `body()` will also decode *chunked encoding*. `recv(size)`, `body(raw=True)` can be used instead for reading the request body as is.
