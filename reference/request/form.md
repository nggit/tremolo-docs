---
layout: page
title: request.form()
parent: Request object
grand_parent: Reference
nav_order: 5
---

An awaitable object, to get the request body with `application/x-www-form-urlencoded` type.

On requests that are not of type `application/x-www-form-urlencoded`, `tremolo.exceptions.BadRequest` will be raised.
