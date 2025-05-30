---
layout: page
title: request.files()
parent: Request object
grand_parent: Reference
nav_order: 6
---

An *async generator*, to stream the request body with `multipart/form-data` type.

On requests that are not of type `multipart/form-data`, `tremolo.exceptions.BadRequest` will be raised.
