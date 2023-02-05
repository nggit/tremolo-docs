
```python
@app.route('/hello')
async def hello_world(**server):
    yield b'Hello '
    yield b'world!'
```
