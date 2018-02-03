# Sanic-Basics

## 1.- Intro

<details><summary>Show 1.- Intro</summary>
<p>

### 1.1.- Sanic

<details><summary>Show 1.1.- Sanic</summary>
<p>

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
    
### 1.2.- Getting Started

<details><summary>Show 1.2.- Getting Started</summary>
<p>

Make sure you have both pip and at least version 3.5 of Python before starting. Sanic uses the new `async`/`await` syntax, so earlier versions of python won't work.

Install Sanic: 
```
python3 -m pip install sanic
```
Create a file called `main.py` with the following code:
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
Run the server: 
```
python3 main.py
```
Open the address `http://0.0.0.0:8000` in your web browser. You should see the message Hello world!.
You now have a working Sanic server!

</p>
</details>

</p>
</details>

## 2.- Routing

<details><summary>Show 2.- Routing</summary>
<p>
    
### 2.1.- Routing

<details><summary>Show 2.1.- Routing</summary>
<p>

Routing allows the user to specify handler functions for different URL endpoints.

A basic route looks like the following, where `app` is an instance of the `Sanic` class:
```python
from sanic.response import json

@app.route("/")
async def test(request):
    return json({ "hello": "world" })
```
When the url `http://server.url/` is accessed (the base url of the server), the final `/` is matched by the router to the handler function, `test`, which then returns a JSON object.

Sanic handler functions must be defined using the `async def` syntax, as they are asynchronous functions.

</p>
</details>

### 2.2.- Request parameters

<details><summary>Show 2.2.- Request parameters</summary>
<p>

Sanic comes with a basic router that supports request parameters.

To specify a parameter, surround it with angle quotes like so: `<PARAM>`. Request parameters will be passed to the route handler functions as keyword arguments.
```python
from sanic.response import text

@app.route('/tag/<tag>')
async def tag_handler(request, tag):
    return text('Tag - {}'.format(tag))
```
To specify a type for the parameter, add a `:type` after the parameter name, inside the quotes. If the parameter does not match the specified type, Sanic will throw a `NotFound` exception, resulting in a `404: Page not found` error on the URL.
```python
from sanic.response import text

@app.route('/number/<integer_arg:int>')
async def integer_handler(request, integer_arg):
    return text('Integer - {}'.format(integer_arg))

@app.route('/number/<number_arg:number>')
async def number_handler(request, number_arg):
    return text('Number - {}'.format(number_arg))

@app.route('/person/<name:[A-z]+>')
async def person_handler(request, name):
    return text('Person - {}'.format(name))

@app.route('/folder/<folder_id:[A-z0-9]{0,4}>')
async def folder_handler(request, folder_id):
    return text('Folder - {}'.format(folder_id))
```

</p>
</details>

### 2.3.- HTTP request types

<details><summary>Show 2.3.- HTTP request types</summary>
<p>

By default, a route defined on a URL will be available for only GET requests to that URL. However, the `@app.route` decorator accepts an optional parameter, `methods`, which allows the handler function to work with any of the HTTP methods in the list.
```python
from sanic.response import text

@app.route('/post', methods=['POST'])
async def post_handler(request):
    return text('POST request - {}'.format(request.json))

@app.route('/get', methods=['GET'])
async def get_handler(request):
    return text('GET request - {}'.format(request.args))
```
There is also an optional host argument (which can be a list or a string). This restricts a route to the host or hosts provided. If there is a also a route with no host, it will be the default.
```python
@app.route('/get', methods=['GET'], host='example.com')
async def get_handler(request):
    return text('GET request - {}'.format(request.args))

# if the host header doesn't match example.com, this route will be used
@app.route('/get', methods=['GET'])
async def get_handler(request):
    return text('GET request in default - {}'.format(request.args))
```
There are also shorthand method decorators:
```python
from sanic.response import text

@app.post('/post')
async def post_handler(request):
    return text('POST request - {}'.format(request.json))

@app.get('/get')
async def get_handler(request):
    return text('GET request - {}'.format(request.args))
```

</p>
</details>

### 2.4.- The `add_route` method

<details><summary>Show 2.4.- The `add_route` method</summary>
<p>

As we have seen, routes are often specified using the `@app.route` decorator. However, this decorator is really just a wrapper for the `app.add_route method`, which is used as follows:
```python
from sanic.response import text

# Define the handler functions
async def handler1(request):
    return text('OK')

async def handler2(request, name):
    return text('Folder - {}'.format(name))

async def person_handler2(request, name):
    return text('Person - {}'.format(name))

# Add each handler function as a route
app.add_route(handler1, '/test')
app.add_route(handler2, '/folder/<name>')
app.add_route(person_handler2, '/person/<name:[A-z]>', methods=['GET'])
```

</p>
</details>

### 2.5.- URL building with `url_for`

<details><summary>Show 2.5.- URL building with `url_for`</summary>
<p>

Sanic provides a `url_for` method, to generate URLs based on the handler method name. This is useful if you want to avoid hardcoding url paths into your app; instead, you can just reference the handler name. For example:
```python
@app.route('/')
async def index(request):
    # generate a URL for the endpoint `post_handler`
    url = app.url_for('post_handler', post_id=5)
    # the URL is `/posts/5`, redirect to it
    return redirect(url)


@app.route('/posts/<post_id>')
async def post_handler(request, post_id):
    return text('Post - {}'.format(post_id))
```
Other things to keep in mind when using `url_for`:

- Keyword arguments passed to `url_for` that are not request parameters will be included in the URL's query string. For example:
```python
url = app.url_for('post_handler', post_id=5, arg_one='one', arg_two='two')
# /posts/5?arg_one=one&arg_two=two
```
- Multivalue argument can be passed to `url_for`. For example:
```python
url = app.url_for('post_handler', post_id=5, arg_one=['one', 'two'])
# /posts/5?arg_one=one&arg_one=two
```
- Also some special arguments (`_anchor`, `_external`, `_scheme`, `_method`, `_server`) passed to `url_for` will have special url building (`_method` is not support now and will be ignored). For example:
```python
url = app.url_for('post_handler', post_id=5, arg_one='one', _anchor='anchor')
# /posts/5?arg_one=one#anchor

url = app.url_for('post_handler', post_id=5, arg_one='one', _external=True)
# //server/posts/5?arg_one=one
# _external requires passed argument _server or SERVER_NAME in app.config or url will be same as no _external

url = app.url_for('post_handler', post_id=5, arg_one='one', _scheme='http', _external=True)
# http://server/posts/5?arg_one=one
# when specifying _scheme, _external must be True

# you can pass all special arguments one time
url = app.url_for('post_handler', post_id=5, arg_one=['one', 'two'], arg_two=2, _anchor='anchor', _scheme='http', _external=True, _server='another_server:8888')
# http://another_server:8888/posts/5?arg_one=one&arg_one=two&arg_two=2#anchor
```
- All valid parameters must be passed to `url_for` to build a URL. If a parameter is not supplied, or if a parameter does not match the specified type, a `URLBuildError` will be thrown.

</p>
</details>

### 2.6.- WebSocket routes

<details><summary>Show 2.6.- WebSocket routes</summary>
<p>

Routes for the WebSocket protocol can be defined with the `@app.websocket` decorator:
```python
@app.websocket('/feed')
async def feed(request, ws):
    while True:
        data = 'hello!'
        print('Sending: ' + data)
        await ws.send(data)
        data = await ws.recv()
        print('Received: ' + data)
```
Alternatively, the `app.add_websocket_route` method can be used instead of the decorator:
```python
async def feed(request, ws):
    pass

app.add_websocket_route(my_websocket_handler, '/feed')
```

Handlers for a WebSocket route are passed the request as first argument, and a WebSocket protocol object as second argument. The protocol object has `send` and `recv` methods to send and receive data respectively.

WebSocket support requires the websockets package by Aymeric Augustin. - https://github.com/aaugustin/websockets

</p>
</details>

### 2.7.- About `strict_slashes`

<details><summary>Show 2.7.- About `strict_slashes`</summary>
<p>

You can make `routes` strict to trailing slash or not, it's configurable.

```python
# provide default strict_slashes value for all routes
app = Sanic('test_route_strict_slash', strict_slashes=True)

# you can also overwrite strict_slashes value for specific route
@app.get('/get', strict_slashes=False)
def handler(request):
    return text('OK')

# It also works for blueprints
bp = Blueprint('test_bp_strict_slash', strict_slashes=True)

@bp.get('/bp/get', strict_slashes=False)
def handler(request):
    return text('OK')

app.blueprint(bp)
```

</p>
</details>

### 2.8.- User defined route name

<details><summary>Show 2.8.- User defined route name</summary>
<p>

You can pass `name` to change the route name to avoid using the default name (`handler.__name__`).

```python
app = Sanic('test_named_route')

@app.get('/get', name='get_handler')
def handler(request):
    return text('OK')

# then you need use `app.url_for('get_handler')`
# instead of # `app.url_for('handler')`

# It also works for blueprints
bp = Blueprint('test_named_bp')

@bp.get('/bp/get', name='get_handler')
def handler(request):
    return text('OK')

app.blueprint(bp)

# then you need use `app.url_for('test_named_bp.get_handler')`
# instead of `app.url_for('test_named_bp.handler')`

# different names can be used for same url with different methods

@app.get('/test', name='route_test')
def handler(request):
    return text('OK')

@app.post('/test', name='route_post')
def handler2(request):
    return text('OK POST')

@app.put('/test', name='route_put')
def handler3(request):
    return text('OK PUT')

# below url are the same, you can use any of them
# '/test'
app.url_for('route_test')
# app.url_for('route_post')
# app.url_for('route_put')

# for same handler name with different methods
# you need specify the name (it's url_for issue)
@app.get('/get')
def handler(request):
    return text('OK')

@app.post('/post', name='post_handler')
def handler(request):
    return text('OK')

# then
# app.url_for('handler') == '/get'
# app.url_for('post_handler') == '/post'
```

</p>
</details>

### 2.9.- Build URL for static files

<details><summary>Show 2.9.- Build URL for static files</summary>
<p>

You can use `url_for` for static file url building now. If it's for file directly, `filename` can be ignored.

```python
app = Sanic('test_static')
app.static('/static', './static')
app.static('/uploads', './uploads', name='uploads')
app.static('/the_best.png', '/home/ubuntu/test.png', name='best_png')

bp = Blueprint('bp', url_prefix='bp')
bp.static('/static', './static')
bp.static('/uploads', './uploads', name='uploads')
bp.static('/the_best.png', '/home/ubuntu/test.png', name='best_png')
app.blueprint(bp)

# then build the url
app.url_for('static', filename='file.txt') == '/static/file.txt'
app.url_for('static', name='static', filename='file.txt') == '/static/file.txt'
app.url_for('static', name='uploads', filename='file.txt') == '/uploads/file.txt'
app.url_for('static', name='best_png') == '/the_best.png'

# blueprint url building
app.url_for('static', name='bp.static', filename='file.txt') == '/bp/static/file.txt'
app.url_for('static', name='bp.uploads', filename='file.txt') == '/bp/uploads/file.txt'
app.url_for('static', name='bp.best_png') == '/bp/static/the_best.png'
```

</p>
</details>

</p>
</details>

# References

- https://sanic.readthedocs.io/en/latest/
- https://github.com/channelcat/sanic/
