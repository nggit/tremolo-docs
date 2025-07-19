---
layout: page
title: app.run()
parent: Application object
grand_parent: Reference
nav_order: 20
---

Tremolo uses [multiprocessing](https://docs.python.org/3/library/multiprocessing.html) so you have to guard `app.run()` with:

```python
if __name__ == '__main__':
```

Otherwise it can be re-executed in the worker/child and will lead to confusion.

