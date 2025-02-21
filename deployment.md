---
layout: page
title: Deploy to Production
---

Tremolo app is very easy to deploy. It's just as simple as running a single script:

```
python3 hello.py
```

Tremolo does not differentiate between development/production mode server. By default the built-in server is intended for production.

Although the use of TLS termination proxy like Nginx is preferred. This will help reduce encryption/CPU load, rather than using the [ssl](https://nggit.github.io/tremolo-docs/configuration.html#ssl) option in Tremolo/Python. Also adds further protection from crafty clients.

## Secure Deployment with Docker
By default, a container uses the unprivileged root. Which is a root user with limited set of capabilities.

Most projects will be fine with that.

But to make it more difficult for attackers to gain access to bare metal / container breakout, it is necessary to create a non-root user in the container for example with:

```
# useradd --home-dir /app --create-home --user-group app
# or (Alpine/Busybox's version):
adduser -Dh /app -u 1000 app app
```

With the `app` (non-root) user created, we cannot bind ports below 1024. Unless we `setcap` the Python binary first:

```
setcap 'cap_net_bind_service=ep' $(readlink -f /usr/bin/python3)
```

This is possible because the `SETFCAP` capability is enabled by default.

Then to execute Python in `CMD` as a user `app`:
```
su -c 'exec python3 hello.py' - app
```

If translated into a full `Dockerfile`, it becomes as follows (adjust to your project):

```
FROM alpine:3.20

# update system
RUN apk update && apk upgrade

# install required packages
RUN apk add --no-cache libcap python3

# create a non-root user app:app
RUN adduser -Dh /app -u 1000 app app

# install python packages
RUN python3 -m venv --system-site-packages /usr/local; \
    python3 -m pip install tremolo; \
    python3 -m pip install uvloop

# clean up
RUN rm -rf /tmp/* /var/cache/apk/*

EXPOSE 80 443
COPY hello.py /app/
WORKDIR /app

ENTRYPOINT ["/usr/bin/env", "--"]
CMD ["sh", "-c", "chown -R app:app /app; \
    setcap 'cap_net_bind_service=ep' $(readlink -f /usr/bin/python3); \
    su -c 'exec python3 hello.py' - app"]
```

You can also further hardening by dropping capabilities such as `SYS_CHROOT`, `NET_RAW`, `SETPCAP`, etc.

You can do your own research regarding your application needs.
