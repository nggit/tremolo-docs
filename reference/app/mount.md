---
layout: page
title: app.mount()
parent: Application object
grand_parent: Reference
nav_order: 10
---

Application `mount()`ing is a concept borrowed from Bottle that is useful for modular routing.

It is also a similar concept to Flask's Blueprints for building larger applications.

In Tremolo, mounting has the following characteristics:
- The middlewares (`@app.on_request`) applied to the *sub* application are independent/isolated
- The middlewares (`@app.on_request`) applied to the *main* application are global (only when the sub has no middleware)
- Hooks (`@app.on_worker_start`) that are applied *anywhere* are global, affecting all parties. Order can be by setting `priority=999`, etc.

## Example
```python
from tremolo import Application

main = Application()
sub = Application()
subsub = Application()

# main.run() -> http://localhost:8000/sub/subsub/hello
@subsub.route('/hello')
async def hello_world(**server):
    return 'Hello world!', 'latin-1'


sub.mount('/subsub', subsub)
main.mount('/sub', sub)


if __name__ == '__main__':
    main.run('0.0.0.0', 8000, debug=True)
```
