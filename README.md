# Sanic-Basics

## 1.- Intro

<details><summary>Show 1.- Intro</summary>
<p>

Sanic
Sanic is a Flask-like Python 3.5+ web server that’s written to go fast. It’s based on the work done by the amazing folks at magicstack, and was inspired by this article - https://magic.io/blog/uvloop-blazing-fast-python-networking/

On top of being Flask-like, Sanic supports async request handlers. This means you can use the new shiny async/await syntax from Python 3.5, making your code non-blocking and speedy.

Sanic is developed on GitHub. Contributions are welcome!

Sanic aspires to be simple
```python
from sanic import Sanic
from sanic.response import json

app = Sanic()

@app.route("/")
async def test(request):
    return json({"hello": "world"})

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000)
```
    
</p>
</details>

# References

- https://sanic.readthedocs.io/en/latest/
- https://github.com/channelcat/sanic/
