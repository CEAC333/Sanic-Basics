# Sanic-Basics

## 1.- Intro

<details><summary>Show 1.- Intro</summary>
<p><ol start=""><ol start="">
  
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

</li>
    
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

</ol></ol></p>
</details>

## 2.- Routing

<details><summary>Show 2.- Routing</summary>
<p><ol start=""><ol start="">
    
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

</ol></ol></p>
</details>

## 3.- Request Data

<details><summary>Show 3.- Request Data</summary>
<p><ol start=""><ol start="">
    
### 3.1- Request Data
    
<details><summary>Show 3.1- Request Data</summary>
<p>
    
When an endpoint receives a HTTP request, the route function is passed a `Request` object.

The following variables are accessible as properties on `Request` objects:

- `json` (any) - JSON body
```python
from sanic.response import json

@app.route("/json")
def post_json(request):
    return json({ "received": True, "message": request.json })
```
- `args` (dict) - Query string variables. A query string is the section of a URL that resembles `?key1=value1&key2=value2`. If that URL were to be parsed, the `args` dictionary would look like `{'key1': ['value1'], 'key2': ['value2']}`. The request's `query_string` variable holds the unparsed string value.
```python
from sanic.response import json

@app.route("/query_string")
def query_string(request):
    return json({ "parsed": True, "args": request.args, "url": request.url, "query_string": request.query_string })
```
- `raw_args` (dict) - On many cases you would need to access the url arguments in a less packed dictionary. For same previous URL `?key1=value1&key2=value2`, the `raw_args` dictionary would look like `{'key1': 'value1', 'key2': 'value2'}`.

- `files` (dictionary of `File` objects) - List of files that have a name, body, and type
```python
from sanic.response import json

@app.route("/files")
def post_json(request):
    test_file = request.files.get('test')

    file_parameters = {
        'body': test_file.body,
        'name': test_file.name,
        'type': test_file.type,
    }

    return json({ "received": True, "file_names": request.files.keys(), "test_file_parameters": file_parameters })
```
- `form` (dict) - Posted form variables.
```python
from sanic.response import json

@app.route("/form")
def post_json(request):
    return json({ "received": True, "form_data": request.form, "test": request.form.get('test') })
```
- `body` (bytes) - Posted raw body. This property allows retrieval of the request's raw data, regardless of content type.
```python
from sanic.response import text

@app.route("/users", methods=["POST",])
def create_user(request):
    return text("You are trying to create a user with the following POST: %s" % request.body)
```
- `headers` (dict) - A case-insensitive dictionary that contains the request headers.

- `ip` (str) - IP address of the requester.

- `port` (str) - Port address of the requester.

- `socket` (tuple) - (IP, port) of the requester.

- `app` - a reference to the Sanic application object that is handling this request. This is useful when inside blueprints or other handlers in modules that do not have access to the global `app` object.
```python
from sanic.response import json
from sanic import Blueprint

bp = Blueprint('my_blueprint')

@bp.route('/')
async def bp_root(request):
    if request.app.config['DEBUG']:
        return json({'status': 'debug'})
    else:
        return json({'status': 'production'})
```

- `url`: The full URL of the request, ie: `http://localhost:8000/posts/1/?foo=bar`

- `scheme`: The URL scheme associated with the request: `http` or `https`

- `host`: The host associated with the request: `localhost:8080`

- `path`: The path of the request: `/posts/1/`

- `query_string`: The query string of the request: `foo=bar` or a blank string ''

- `uri_template`: Template for matching route handler: `/posts/<id>/`

- `token`: The value of Authorization header: `Basic YWRtaW46YWRtaW4=`
    
</p>
</details>

### 3.2.- Accessing values using `get` and `getlist`

<details><summary>Show 3.2.- Accessing values using `get` and `getlist`</summary>
<p>
    
The request properties which return a dictionary actually return a subclass of `dict` called `RequestParameters`. The key difference when using this object is the distinction between the `get` and `getlist` methods.

- `get(key, default=None)` operates as normal, except that when the value of the given key is a list, only the first item is returned.
- `getlist(key, default=None)` operates as normal, returning the entire list.
```python
from sanic.request import RequestParameters

args = RequestParameters()
args['titles'] = ['Post 1', 'Post 2']

args.get('titles') # => 'Post 1'

args.getlist('titles') # => ['Post 1', 'Post 2']
```
</p>
</details>
    

</ol></ol></p>
</details>

## 4.- Response

<details><summary>Show 4.- Response</summary>
<p><ol start=""><ol start="">
  
Use functions in `sanic.response` module to create responses.
    
### 4.1.- Plain text

<details><summary>Show 4.1- Plain text</summary>
<p>
  
```python
from sanic import response

@app.route('/text')
def handle_request(request):
    return response.text('Hello world!')
```

</p>
</details>

### 4.2.- HTML
    
<details><summary>Show 4.2.- HTML</summary>
<p>
  
```python
from sanic import response

@app.route('/html')
def handle_request(request):
    return response.html('<p>Hello world!</p>')
```
  
</p>
</details>

### 4.3.- JSON
    
<details><summary>Show 4.3.- JSON</summary>
<p>
  
```python
from sanic import response

@app.route('/json')
def handle_request(request):
    return response.json({'message': 'Hello world!'})
```
  
</p>
</details>

### 4.4.- File
    
<details><summary>Show 4.4.- File</summary>
<p>
  
```python
from sanic import response

@app.route('/file')
async def handle_request(request):
    return await response.file('/srv/www/whatever.png')
```
  
</p>
</details>

### 4.5.- Streaming
    
<details><summary>Show 4.5.- Streaming</summary>
<p>
  
```python
from sanic import response

@app.route("/streaming")
async def index(request):
    async def streaming_fn(response):
        response.write('foo')
        response.write('bar')
    return response.stream(streaming_fn, content_type='text/plain')
```
  
</p>
</details>

### 4.6.- File Streaming
    
<details><summary>Show 4.6.- File Streaming</summary>
<p>
  
For large files, a combination of File and Streaming above
  
```python
from sanic import response

@app.route('/big_file.png')
async def handle_request(request):
    return await response.file_stream('/srv/www/whatever.png')
```
  
</p>
</details>

### 4.7.- Redirect
    
<details><summary>Show 4.7.- Redirect</summary>
<p>
  
```python
from sanic import response

@app.route('/redirect')
def handle_request(request):
    return response.redirect('/json')
```
  
</p>
</details>

### 4.8.- Raw
    
<details><summary>Show 4.8.- Raw</summary>
<p>
  
Response without encoding the body
  
```python
from sanic import response

@app.route('/raw')
def handle_request(request):
    return response.raw('raw data')
```
  
</p>
</details>

### 4.9.- Modify headers or status
    
<details><summary>Show 4.9.- Modify headers or status</summary>
<p>
  
To modify headers or status code, pass the `headers` or `status` argument to those functions:
  
```python
from sanic import response

@app.route('/json')
def handle_request(request):
    return response.json(
        {'message': 'Hello world!'},
        headers={'X-Served-By': 'sanic'},
        status=200
    )
```
  
</p>
</details>
    

</ol></ol></p>
</details>

## 5.- Static Files
<details><summary>Show 5.- Static Files</summary>
<p>
  
Static files and directories, such as an image file, are served by Sanic when registered with the `app.static` method. The method takes an endpoint URL and a filename. The file specified will then be accessible via the given endpoint.
```python
from sanic import Sanic
from sanic.blueprints import Blueprint

app = Sanic(__name__)

# Serves files from the static folder to the URL /static
app.static('/static', './static')
# use url_for to build the url, name defaults to 'static' and can be ignored
app.url_for('static', filename='file.txt') == '/static/file.txt'
app.url_for('static', name='static', filename='file.txt') == '/static/file.txt'

# Serves the file /home/ubuntu/test.png when the URL /the_best.png
# is requested
app.static('/the_best.png', '/home/ubuntu/test.png', name='best_png')

# you can use url_for to build the static file url
# you can ignore name and filename parameters if you don't define it
app.url_for('static', name='best_png') == '/the_best.png'
app.url_for('static', name='best_png', filename='any') == '/the_best.png'

# you need define the name for other static files
app.static('/another.png', '/home/ubuntu/another.png', name='another')
app.url_for('static', name='another') == '/another.png'
app.url_for('static', name='another', filename='any') == '/another.png'

# also, you can use static for blueprint
bp = Blueprint('bp', url_prefix='/bp')
bp.static('/static', './static')

# servers the file directly
bp.static('/the_best.png', '/home/ubuntu/test.png', name='best_png')
app.blueprint(bp)

app.url_for('static', name='bp.static', filename='file.txt') == '/bp/static/file.txt'
app.url_for('static', name='bp.best_png') == '/bp/test_best.png'

app.run(host="0.0.0.0", port=8000)
```

</p>
</details>

## 6.- Exceptions

<details><summary>Show 6.- Exceptions</summary>
<p><ol start=""><ol start="">
  
Exceptions can be thrown from within request handlers and will automatically be handled by Sanic. Exceptions take a message as their first argument, and can also take a status code to be passed back in the HTTP response.
  
### 6.1.- Throwing an exception
  
<details><summary>Show 6.1.- Throwing an exception</summary>
<p>
  
To throw an exception, simply `raise` the relevant exception from the `sanic.exceptions` module.

```python
from sanic.exceptions import ServerError

@app.route('/killme')
def i_am_ready_to_die(request):
    raise ServerError("Something bad happened", status_code=500)
```

You can also use the `abort` function with the appropriate status code:

```python
from sanic.exceptions import abort
from sanic.response import text

@app.route('/youshallnotpass')
def no_no(request):
        abort(401)
        # this won't happen
        text("OK")
```

</p>
</details>

### 6.2.- Handling exceptions
  
<details><summary>Show 6.2.- Handling exceptions</summary>
<p>
  
To override Sanic's default handling of an exception, the `@app.exception` decorator is used. The decorator expects a list of exceptions to handle as arguments. You can pass `SanicException` to catch them all! The decorated exception handler function must take a `Request` and `Exception` object as arguments.

```python
from sanic.response import text
from sanic.exceptions import NotFound

@app.exception(NotFound)
def ignore_404s(request, exception):
    return text("Yep, I totally found the page: {}".format(request.url))
```

</p>
</details>

### 6.3.- Useful exceptions
  
<details><summary>Show 6.3.- Useful exceptions</summary>
<p>
  
Some of the most useful exceptions are presented below:

- `NotFound`: called when a suitable route for the request isn't found.
- `ServerError`: called when something goes wrong inside the server. This usually occurs if there is an exception raised in user code.

See the `sanic.exceptions` module for the full list of exceptions to throw.

</p>
</details>
  
</ol></ol></p>
</details>

## 7.- Middleware And Listeners

<details><summary>Show 7.- Middleware And Listeners</summary>
<p><ol start=""><ol start="">
  
Middleware are functions which are executed before or after requests to the server. They can be used to modify the request to or response from user-defined handler functions.

Additionally, Sanic provides listeners which allow you to run code at various points of your application's lifecycle.
  
### 7.1.- Middleware

<details><summary>Show 7.1.- Middleware</summary>
<p>
  
There are two types of middleware: request and response. Both are declared using the `@app.middleware` decorator, with the decorator's parameter being a string representing its type: `'request'` or `'response'`. Response middleware receives both the request and the response as arguments.

The simplest middleware doesn't modify the request or response at all:

```python
@app.middleware('request')
async def print_on_request(request):
    print("I print when a request is received by the server")

@app.middleware('response')
async def print_on_response(request, response):
    print("I print when a response is returned by the server")
```
</p>
</details>

### 7.2.- Modifying the request or response

<details><summary>Show 7.2.- Modifying the request or response</summary>
<p>
  
Middleware can modify the request or response parameter it is given, as long as it does not return it. The following example shows a practical use-case for this.

```python
app = Sanic(__name__)

@app.middleware('response')
async def custom_banner(request, response):
    response.headers["Server"] = "Fake-Server"

@app.middleware('response')
async def prevent_xss(request, response):
    response.headers["x-xss-protection"] = "1; mode=block"

app.run(host="0.0.0.0", port=8000)
```

The above code will apply the two middleware in order. First, the middleware **custom_banner** will change the HTTP response header *Server to Fake-Server*, and the second middleware **prevent_xss** will add the HTTP header for preventing Cross-Site-Scripting (XSS) attacks. These two functions are invoked *after* a user function returns a response.
  
</p>
</details>

### 7.3.- Responding early

<details><summary>Show 7.3.- Responding early</summary>
<p>
  
If middleware returns a `HTTPResponse` object, the request will stop processing and the response will be returned. If this occurs to a request before the relevant user route handler is reached, the handler will never be called. Returning a response will also prevent any further middleware from running.

```python
@app.middleware('request')
async def halt_request(request):
    return text('I halted the request')

@app.middleware('response')
async def halt_response(request, response):
    return text('I halted the response')
```

</p>
</details>

### 7.4.- Listeners

<details><summary>Show 7.4.- Listeners</summary>
<p>
  
If you want to execute startup/teardown code as your server starts or closes, you can use the following listeners:

- `before_server_start`
- `after_server_start`
- `before_server_stop`
- `after_server_stop`

These listeners are implemented as decorators on functions which accept the app object as well as the asyncio loop.

For example:

```python
@app.listener('before_server_start')
async def setup_db(app, loop):
    app.db = await db_setup()

@app.listener('after_server_start')
async def notify_server_started(app, loop):
    print('Server successfully started!')

@app.listener('before_server_stop')
async def notify_server_stopping(app, loop):
    print('Server shutting down!')

@app.listener('after_server_stop')
async def close_db(app, loop):
    await app.db.close()
```

If you want to schedule a background task to run after the loop has started, Sanic provides the `add_task` method to easily do so.
```python 
async def notify_server_started_after_five_seconds():
    await asyncio.sleep(5)
    print('Server successfully started!')

app.add_task(notify_server_started_after_five_seconds())
```

</p>
</details>
  
</ol></ol></p>
</details>

## 8.- Blueprints

<details><summary>Show 8.- Blueprints</summary>
<p><ol start=""><ol start="">
  
Blueprints are objects that can be used for sub-routing within an application. Instead of adding routes to the application instance, blueprints define similar methods for adding routes, which are then registered with the application in a flexible and pluggable manner.

Blueprints are especially useful for larger applications, where your application logic can be broken down into several groups or areas of responsibility.
  
### 8.1.- My First Blueprint

<details><summary>Show 8.1.- My First Blueprint</summary>
<p>
  
The following shows a very simple blueprint that registers a handler-function at the root `/` of your application.

Suppose you save this file as `my_blueprint.py`, which can be imported into your main application later.

```python
from sanic.response import json
from sanic import Blueprint

bp = Blueprint('my_blueprint')

@bp.route('/')
async def bp_root(request):
    return json({'my': 'blueprint'})
```  

</p>
</details>

### 8.2.- Registering blueprints

<details><summary>Show 8.2.- Registering blueprints</summary>
<p>
  
Blueprints must be registered with the application.

```python
from sanic import Sanic
from my_blueprint import bp

app = Sanic(__name__)
app.blueprint(bp)

app.run(host='0.0.0.0', port=8000, debug=True)
```

This will add the blueprint to the application and register any routes defined by that blueprint. In this example, the registered routes in the `app.router` will look like:

```
[Route(handler=<function bp_root at 0x7f908382f9d8>, methods=None, pattern=re.compile('^/$'), parameters=[])]
```
  
</p>
</details>

### 8.3.- Using blueprints

<details><summary>Show 8.3.- Using blueprints</summary>
<p>
  
Blueprints have much the same functionality as an application instance.

#### WebSocket routes
WebSocket handlers can be registered on a blueprint using the `@bp.websocket` decorator or `bp.add_websocket_route` method.

#### Middleware
Using blueprints allows you to also register middleware globally.

```python
@bp.middleware
async def print_on_request(request):
    print("I am a spy")

@bp.middleware('request')
async def halt_request(request):
    return text('I halted the request')

@bp.middleware('response')
async def halt_response(request, response):
    return text('I halted the response')
```

#### Exceptions
Exceptions can be applied exclusively to blueprints globally.

```python
@bp.exception(NotFound)
def ignore_404s(request, exception):
    return text("Yep, I totally found the page: {}".format(request.url))
```

#### Static files
Static files can be served globally, under the blueprint prefix.

```python 
# suppose bp.name == 'bp'

bp.static('/web/path', '/folder/to/serve')
# also you can pass name parameter to it for url_for
bp.static('/web/path', '/folder/to/server', name='uploads')
app.url_for('static', name='bp.uploads', filename='file.txt') == '/bp/web/path/file.txt'
```

</p>
</details>

### 8.4.- Start and Stop

<details><summary>Show 8.4.- Start and Stop</summary>
<p>
  
Blueprints can run functions during the start and stop process of the server. If running in multiprocessor mode (more than 1 worker), these are triggered after the workers fork.

Available events are:

- `before_server_start`: Executed before the server begins to accept connections
- `after_server_start`: Executed after the server begins to accept connections
- `before_server_stop`: Executed before the server stops accepting connections
- `after_server_stop`: Executed after the server is stopped and all requests are complete
bp = Blueprint('my_blueprint')

```python
@bp.listener('before_server_start')
async def setup_connection(app, loop):
    global database
    database = mysql.connect(host='127.0.0.1'...)

@bp.listener('after_server_stop')
async def close_connection(app, loop):
    await database.close()
```  

</p>
</details>

### 8.5.- Use-case: API versioning

<details><summary>Show 8.5.- Use-case: API versioning</summary>
<p>
  
Blueprints can be very useful for API versioning, where one blueprint may point at `/v1/<routes>`, and another pointing at `/v2/<routes>`.

When a blueprint is initialised, it can take an optional `url_prefix` argument, which will be prepended to all routes defined on the blueprint. This feature can be used to implement our API versioning scheme.

```python
# blueprints.py
from sanic.response import text
from sanic import Blueprint

blueprint_v1 = Blueprint('v1', url_prefix='/v1')
blueprint_v2 = Blueprint('v2', url_prefix='/v2')

@blueprint_v1.route('/')
async def api_v1_root(request):
    return text('Welcome to version 1 of our documentation')

@blueprint_v2.route('/')
async def api_v2_root(request):
    return text('Welcome to version 2 of our documentation')
```

When we register our blueprints on the app, the routes `/v1` and `/v2` will now point to the individual blueprints, which allows the creation of sub-sites for each API version.

```python
# main.py
from sanic import Sanic
from blueprints import blueprint_v1, blueprint_v2

app = Sanic(__name__)
app.blueprint(blueprint_v1, url_prefix='/v1')
app.blueprint(blueprint_v2, url_prefix='/v2')

app.run(host='0.0.0.0', port=8000, debug=True)
```  

</p>
</details>

### 8.6.- URL Building with `url_for`

<details><summary>Show </summary>
<p>
  
If you wish to generate a URL for a route inside of a blueprint, remember that the endpoint name takes the format `<blueprint_name>.<handler_name>`. For example:

```python
@blueprint_v1.route('/')
async def root(request):
    url = request.app.url_for('v1.post_handler', post_id=5) # --> '/v1/post/5'
    return redirect(url)


@blueprint_v1.route('/post/<post_id>')
async def post_handler(request, post_id):
    return text('Post {} in Blueprint V1'.format(post_id))  
```
</p>
</details>
  
</ol></ol></p>
</details>

## 9.- Configuration

<details><summary>Show 9.- Configuration</summary>
<p><ol start=""><ol start="">
  
Any reasonably complex application will need configuration that is not baked into the actual code. Settings might be different for different environments or installations.
  
### 9.1.- Basics

<details><summary>Show 9.1.- Basics</summary>
<p>
  
Sanic holds the configuration in the `config` attribute of the application object. The configuration object is merely an object that can be modified either using dot-notation or like a dictionary:

```python
app = Sanic('myapp')
app.config.DB_NAME = 'appdb'
app.config.DB_USER = 'appuser'
```

Since the config object actually is a dictionary, you can use its `update` method in order to set several values at once:

```python
db_settings = {
    'DB_HOST': 'localhost',
    'DB_NAME': 'appdb',
    'DB_USER': 'appuser'
}
app.config.update(db_settings)
```

In general the convention is to only have UPPERCASE configuration parameters. The methods described below for loading configuration only look for such uppercase parameters.
  
</p>
</details>

### 9.2.- Loading Configuration

<details><summary>Show 9.2.- Loading Configuration</summary>
<p>
  
There are several ways how to load configuration.

#### From Environment Variables
Any variables defined with the `SANIC_` prefix will be applied to the sanic config. For example, setting `SANIC_REQUEST_TIMEOUT` will be loaded by the application automatically and fed into the `REQUEST_TIMEOUT` config variable. You can pass a different prefix to Sanic:

```python
app = Sanic(load_env='MYAPP_')
```

Then the above variable would be `MYAPP_REQUEST_TIMEOUT`. If you want to disable loading from environment variables you can set it to `False` instead:

```python
app = Sanic(load_env=False)
```

#### From an Object
If there are a lot of configuration values and they have sensible defaults it might be helpful to put them into a module:

```python
import myapp.default_settings

app = Sanic('myapp')
app.config.from_object(myapp.default_settings)
```

You could use a class or any other object as well.

#### From a File
Usually you will want to load configuration from a file that is not part of the distributed application. You can load configuration from a file using `from_pyfile(/path/to/config_file)`. However, that requires the program to know the path to the config file. So instead you can specify the location of the config file in an environment variable and tell Sanic to use that to find the config file:

```python
app = Sanic('myapp')
app.config.from_envvar('MYAPP_SETTINGS')
``` 
Then you can run your application with the `MYAPP_SETTINGS` environment variable set:

```python
$ MYAPP_SETTINGS=/path/to/config_file python3 myapp.py
INFO: Goin' Fast @ http://0.0.0.0:8000
```

The config files are regular Python files which are executed in order to load them. This allows you to use arbitrary logic for constructing the right configuration. Only uppercase variables are added to the configuration. Most commonly the configuration consists of simple key value pairs:

```python
# config_file
DB_HOST = 'localhost'
DB_NAME = 'appdb'
DB_USER = 'appuser'
```

</p>
</details>

### 9.3.- Builtin Configuration Values

<details><summary>Show 9.3.- Builtin Configuration Values</summary>
<p>
  
Out of the box there are just a few predefined values which can be overwritten when creating the application.

```
| Variable           | Default   | Description                                   |
| ------------------ | --------- | --------------------------------------------- |
| REQUEST_MAX_SIZE   | 100000000 | How big a request may be (bytes)              |
| REQUEST_TIMEOUT    | 60        | How long a request can take to arrive (sec)   |
| RESPONSE_TIMEOUT   | 60        | How long a response can take to process (sec) |
| KEEP_ALIVE         | True      | Disables keep-alive when False                |
| KEEP_ALIVE_TIMEOUT | 5         | How long to hold a TCP connection open (sec)  |
```

#### The different Timeout variables:

A request timeout measures the duration of time between the instant when a new open TCP connection is passed to the Sanic backend server, and the instant when the whole HTTP request is received. If the time taken exceeds the `REQUEST_TIMEOUT` value (in seconds), this is considered a Client Error so Sanic generates a HTTP 408 response and sends that to the client. Adjust this value higher if your clients routinely pass very large request payloads or upload requests very slowly.

A response timeout measures the duration of time between the instant the Sanic server passes the HTTP request to the Sanic App, and the instant a HTTP response is sent to the client. If the time taken exceeds the `RESPONSE_TIMEOUT` value (in seconds), this is considered a Server Error so Sanic generates a HTTP 503 response and sets that to the client. Adjust this value higher if your application is likely to have long-running process that delay the generation of a response.

#### What is Keep Alive? And what does the Keep Alive Timeout value do?

Keep-Alive is a HTTP feature indroduced in HTTP 1.1. When sending a HTTP request, the client (usually a web browser application) can set a Keep-Alive header to indicate for the http server (Sanic) to not close the TCP connection after it has send the response. This allows the client to reuse the existing TCP connection to send subsequent HTTP requests, and ensures more efficient network traffic for both the client and the server.

The `KEEP_ALIVE` config variable is set to `True` in Sanic by default. If you don't need this feature in your application, set it to `False` to cause all client connections to close immediately after a response is sent, regardless of the Keep-Alive header on the request.

The amount of time the server holds the TCP connection open is decided by the server itself. In Sanic, that value is configured using the `KEEP_ALIVE_TIMEOUT` value. By default, it is set to 5 seconds, this is the same default setting as the Apache HTTP server and is a good balance between allowing enough time for the client to send a new request, and not holding open too many connections at once. Do not exceed 75 seconds unless you know your clients are using a browser which supports TCP connections held open for that long.

For reference:

``` 
Apache httpd server default keepalive timeout = 5 seconds
Nginx server default keepalive timeout = 75 seconds
Nginx performance tuning guidelines uses keepalive = 15 seconds
IE (5-9) client hard keepalive limit = 60 seconds
Firefox client hard keepalive limit = 115 seconds
Opera 11 client hard keepalive limit = 120 seconds
Chrome 13+ client keepalive limit > 300+ seconds
```
  
</p>
</details>
  
</ol></ol></p>
</details>

## 10.- Cookies

<details><summary>Show 10.- Cookies</summary>
<p><ol start=""><ol start="">
  
Cookies are pieces of data which persist inside a user’s browser. Sanic can both read and write cookies, which are stored as key-value pairs.

```
Warning

Cookies can be freely altered by the client. Therefore you 
cannot just store data such as login information in cookies 
as-is, as they can be freely altered by the client. To ensure 
data you store in cookies is not forged or tampered with by 
the client, use something like itsdangerous to 
cryptographically sign the data. 
- https://pythonhosted.org/itsdangerous/
```
  
### 10.1.- Reading cookies

<details><summary>Show 10.1.- Reading cookies</summary>
<p>
  
A user’s cookies can be accessed via the Request object’s cookies dictionary.
  
```python 
from sanic.response import text

@app.route("/cookie")
async def test(request):
    test_cookie = request.cookies.get('test')
    return text("Test cookie set to: {}".format(test_cookie))
```
  
</p>
</details>

### 10.2.- Writing cookies

<details><summary>Show 10.2.- Writing cookies</summary>
<p>
  
When returning a response, cookies can be set on the Response object.
  
```python 
from sanic.response import text

@app.route("/cookie")
async def test(request):
    response = text("There's a cookie up in this response")
    response.cookies['test'] = 'It worked!'
    response.cookies['test']['domain'] = '.gotta-go-fast.com'
    response.cookies['test']['httponly'] = True
    return response
```
  
</p>
</details>

### 10.3.- Deleting cookies

<details><summary>Show 10.3.- Deleting cookies</summary>
<p>
  
Cookies can be removed semantically or explicitly.
  
```python 
from sanic.response import text

@app.route("/cookie")
async def test(request):
    response = text("Time to eat some cookies muahaha")

    # This cookie will be set to expire in 0 seconds
    del response.cookies['kill_me']

    # This cookie will self destruct in 5 seconds
    response.cookies['short_life'] = 'Glad to be here'
    response.cookies['short_life']['max-age'] = 5
    del response.cookies['favorite_color']

    # This cookie will remain unchanged
    response.cookies['favorite_color'] = 'blue'
    response.cookies['favorite_color'] = 'pink'
    del response.cookies['favorite_color']

    return response
```

Response cookies can be set like dictionary values and have the following parameters available:

- `expires` (datetime): The time for the cookie to expire on the client’s browser.
- `path` (string): The subset of URLs to which this cookie applies. Defaults to /.
- `comment` (string): A comment (metadata).
- `domain` (string): Specifies the domain for which the cookie is valid. An explicitly specified domain must always start with a dot.
- `max-age` (number): Number of seconds the cookie should live for.
- `secure` (boolean): Specifies whether the cookie will only be sent via HTTPS.
- `httponly` (boolean): Specifies whether the cookie cannot be read by Javascript.
  
</p>
</details>
  
</ol></ol></p>
</details>

## 11.- Handle Decorators

<details><summary>Show 11.- Handle Decorators</summary>
<p><ol start=""><ol start="">
  
Since Sanic handlers are simple Python functions, you can apply decorators to them in a similar manner to Flask. A typical use case is when you want some code to run before a handler's code is executed.
  
### Authorization Decorator

<details><summary>Show Authorization Decorator</summary>
<p>
  
Let's say you want to check that a user is authorized to access a particular endpoint. You can create a decorator that wraps a handler function, checks a request if the client is authorized to access a resource, and sends the appropriate response.
  
```python 
from functools import wraps
from sanic.response import json

def authorized():
    def decorator(f):
        @wraps(f)
        async def decorated_function(request, *args, **kwargs):
            # run some method that checks the request
            # for the client's authorization status
            is_authorized = check_request_for_authorization_status(request)

            if is_authorized:
                # the user is authorized.
                # run the handler method and return the response
                response = await f(request, *args, **kwargs)
                return response
            else:
                # the user is not authorized. 
                return json({'status': 'not_authorized'}, 403)
        return decorated_function
    return decorator


@app.route("/")
@authorized()
async def test(request):
    return json({status: 'authorized'})
```

</p>
</details>
  
</ol></ol></p>
</details>

## 12.- Streaming

<details><summary>Show 12.- Streaming</summary>
<p><ol start=""><ol start="">
  
### 12.1.- Request Streaming

<details><summary>Show 12.1.- Request Streaming</summary>
<p>
  
Sanic allows you to get request data by stream, as below. When the request ends, `request.stream.get()` returns `None`. Only post, put and patch decorator have stream argument.
  
```python 
from sanic import Sanic
from sanic.views import CompositionView
from sanic.views import HTTPMethodView
from sanic.views import stream as stream_decorator
from sanic.blueprints import Blueprint
from sanic.response import stream, text

bp = Blueprint('blueprint_request_stream')
app = Sanic('request_stream')


class SimpleView(HTTPMethodView):

    @stream_decorator
    async def post(self, request):
        result = ''
        while True:
            body = await request.stream.get()
            if body is None:
                break
            result += body.decode('utf-8')
        return text(result)


@app.post('/stream', stream=True)
async def handler(request):
    async def streaming(response):
        while True:
            body = await request.stream.get()
            if body is None:
                break
            body = body.decode('utf-8').replace('1', 'A')
            response.write(body)
    return stream(streaming)


@bp.put('/bp_stream', stream=True)
async def bp_handler(request):
    result = ''
    while True:
        body = await request.stream.get()
        if body is None:
            break
        result += body.decode('utf-8').replace('1', 'A')
    return text(result)


async def post_handler(request):
    result = ''
    while True:
        body = await request.stream.get()
        if body is None:
            break
        result += body.decode('utf-8')
    return text(result)

app.blueprint(bp)
app.add_route(SimpleView.as_view(), '/method_view')
view = CompositionView()
view.add(['POST'], post_handler, stream=True)
app.add_route(view, '/composition_view')


if __name__ == '__main__':
    app.run(host='127.0.0.1', port=8000)
```
  
</p>
</details>

### 12.2.- Response Streaming

<details><summary>Show 12.2.- Response Streaming</summary>
<p>
  
Sanic allows you to stream content to the client with the `stream` method. This method accepts a coroutine callback which is passed a `StreamingHTTPResponse` object that is written to. A simple example is like follows:
  
```python 
from sanic import Sanic
from sanic.response import stream

app = Sanic(__name__)

@app.route("/")
async def test(request):
    async def sample_streaming_fn(response):
        response.write('foo,')
        response.write('bar')

    return stream(sample_streaming_fn, content_type='text/csv')
```

This is useful in situations where you want to stream content to the client that originates in an external service, like a database. For example, you can stream database records to the client with the asynchronous cursor that `asyncpg` provides:

```python 
@app.route("/")
async def index(request):
    async def stream_from_db(response):
        conn = await asyncpg.connect(database='test')
        async with conn.transaction():
            async for record in conn.cursor('SELECT generate_series(0, 10)'):
                response.write(record[0])

    return stream(stream_from_db)
```
  
</p>
</details>
  
</ol></ol></p>
</details>

## 13.- Class-Based Views

<details><summary>Show 13.- Class-Based Views</summary>
<p><ol start=""><ol start="">
  
Class-based views are simply classes which implement response behaviour to requests. They provide a way to compartmentalise handling of different HTTP request types at the same endpoint. Rather than defining and decorating three different handler functions, one for each of an endpoint's supported request type, the endpoint can be assigned a class-based view.
  
### 13.1.- Defining views

<details><summary>Show 13.1.- Defining views</summary>
<p>
  
A class-based view should subclass `HTTPMethodView`. You can then implement class methods for every HTTP request type you want to support. If a request is received that has no defined method, a `405: Method not allowed` response will be generated.

To register a class-based view on an endpoint, the `app.add_route` method is used. The first argument should be the defined class with the method `as_view` invoked, and the second should be the URL endpoint.

The available methods are `get`, `post`, `put`, `patch`, and `delete`. A class using all these methods would look like the following.
  
```python 
from sanic import Sanic
from sanic.views import HTTPMethodView
from sanic.response import text

app = Sanic('some_name')

class SimpleView(HTTPMethodView):

  def get(self, request):
      return text('I am get method')

  def post(self, request):
      return text('I am post method')

  def put(self, request):
      return text('I am put method')

  def patch(self, request):
      return text('I am patch method')

  def delete(self, request):
      return text('I am delete method')

app.add_route(SimpleView.as_view(), '/')
```

You can also use `async` syntax.

```python 
from sanic import Sanic
from sanic.views import HTTPMethodView
from sanic.response import text

app = Sanic('some_name')

class SimpleAsyncView(HTTPMethodView):

  async def get(self, request):
      return text('I am async get method')

app.add_route(SimpleAsyncView.as_view(), '/')
```
  
</p>
</details>

### 13.2.- URL parameters

<details><summary>Show 13.2.- URL parameters</summary>
<p>
  
If you need any URL parameters, as discussed in the routing guide, include them in the method definition.
  
```python 
class NameView(HTTPMethodView):

  def get(self, request, name):
    return text('Hello {}'.format(name))

app.add_route(NameView.as_view(), '/<name>')
```
  
</p>
</details>

### 13.3.- Decorators

<details><summary>Show 13.3.- Decorators</summary>
<p>
  
If you want to add any decorators to the class, you can set the `decorators` class variable. These will be applied to the class when `as_view` is called.
  
```python 
class ViewWithDecorator(HTTPMethodView):
  decorators = [some_decorator_here]

  def get(self, request, name):
    return text('Hello I have a decorator')

app.add_route(ViewWithDecorator.as_view(), '/url')
```

#### URL Building

If you wish to build a URL for an HTTPMethodView, remember that the class name will be the endpoint that you will pass into `url_for`. For example:

```python 
@app.route('/')
def index(request):
    url = app.url_for('SpecialClassView')
    return redirect(url)


class SpecialClassView(HTTPMethodView):
    def get(self, request):
        return text('Hello from the Special Class View!')


app.add_route(SpecialClassView.as_view(), '/special_class_view')
```
  
</p>
</details>

### 13.4.- Using CompositionView

<details><summary>Show 13.4.- Using CompositionView</summary>
<p>
  
As an alternative to the `HTTPMethodView`, you can use `CompositionView` to move handler functions outside of the view class.

Handler functions for each supported HTTP method are defined elsewhere in the source, and then added to the view using the `CompositionView.add` method. The first parameter is a list of HTTP methods to handle (e.g. `['GET', 'POST']`), and the second is the handler function. The following example shows `CompositionView` usage with both an external handler function and an inline lambda:
  
```python 
from sanic import Sanic
from sanic.views import CompositionView
from sanic.response import text

app = Sanic(__name__)

def get_handler(request):
    return text('I am a get method')

view = CompositionView()
view.add(['GET'], get_handler)
view.add(['POST', 'PUT'], lambda request: text('I am a post/put method'))

# Use the new view to handle requests to the base URL
app.add_route(view, '/')
```

Note: currently you cannot build a URL for a CompositionView using `url_for`.
  
</p>
</details>
  
</ol></ol></p>
</details>

## 14.- Custom Protocols

<details><summary>Show 14.- Custom Protocols</summary>
<p><ul><ul type="square">
  
Note: this is advanced usage, and most readers will not need such functionality.

You can change the behavior of Sanic's protocol by specifying a custom protocol, which should be a subclass of `asyncio.protocol`. This protocol can then be passed as the keyword argument `protocol` to the `sanic.run` method. - https://docs.python.org/3/library/asyncio-protocol.html#protocol-classes

The constructor of the custom protocol class receives the following keyword arguments from Sanic.

- `loop`: an `asyncio`-compatible event loop.

- `connections`: a `set` to store protocol objects. When Sanic receives `SIGINT` or `SIGTERM`, it executes `protocol.close_if_idle` for all protocol objects stored in this set.

- `signal`: a `sanic.server.Signal` object with the `stopped` attribute. When Sanic receives `SIGINT` or `SIGTERM`, `signal.stopped` is assigned `True`.

- `request_handler`: a coroutine that takes a `sanic.request.Request` object and a response callback as arguments.
error_handler: a sanic.exceptions.Handler which is called when exceptions are raised.

- `request_timeout`: the number of seconds before a request times out.
 `request_max_size`: an integer specifying the maximum size of a request, in bytes.


  
### Example

<details><summary>Show Example</summary>
<p>
  
An error occurs in the default protocol if a handler function does not return an `HTTPResponse` object.

By overriding the `write_response` protocol method, if a handler returns a string it will be converted to an `HTTPResponse object`.
  
```python 
from sanic import Sanic
from sanic.server import HttpProtocol
from sanic.response import text

app = Sanic(__name__)


class CustomHttpProtocol(HttpProtocol):

    def __init__(self, *, loop, request_handler, error_handler,
                 signal, connections, request_timeout, request_max_size):
        super().__init__(
            loop=loop, request_handler=request_handler,
            error_handler=error_handler, signal=signal,
            connections=connections, request_timeout=request_timeout,
            request_max_size=request_max_size)

    def write_response(self, response):
        if isinstance(response, str):
            response = text(response)
        self.transport.write(
            response.output(self.request.version)
        )
        self.transport.close()


@app.route('/')
async def string(request):
    return 'string'


@app.route('/1')
async def response(request):
    return text('response')

app.run(host='0.0.0.0', port=8000, protocol=CustomHttpProtocol)
```
  
</p>
</details>
  
</ol></ol></p>
</details>

## 15.- SSL Example

<details><summary>Show 15.- SSL Example</summary>
<p><ul><ul type="square">
  
Optionally pass in an SSLContext:
  
```python 
import ssl
context = ssl.create_default_context(purpose=ssl.Purpose.CLIENT_AUTH)
context.load_cert_chain("/path/to/cert", keyfile="/path/to/keyfile")

app.run(host="0.0.0.0", port=8443, ssl=context)
```

You can also pass in the locations of a certificate and key as a dictionary:

```python 
ssl = {'cert': "/path/to/cert", 'key': "/path/to/keyfile"}
app.run(host="0.0.0.0", port=8443, ssl=ssl)
```
  
</ul></ul></p>
</details>

## 16.- Logging

<details><summary>Show 16.- Logging</summary>
<p><ul><ul type="square">
  
Sanic allows you to do different types of logging (access log, error log) on the requests based on the python3 logging API. You should have some basic knowledge on python3 logging if you want to create a new configuration.

- https://docs.python.org/3/howto/logging.html
  
### 16.1.- Quick Start 

<details><summary>Show 16.1.- Quick Start</summary>
<p>
  
A simple example using default settings would be like this:
  
```python 
from sanic import Sanic

app = Sanic('test')

@app.route('/')
async def test(request):
    return response.text('Hello World!')

if __name__ == "__main__":
  app.run(debug=True, access_log=True)
```

To use your own logging config, simply use `logging.config.dictConfig`, or pass `log_config` when you initialize `Sanic` app:

```python 
app = Sanic('test', log_config=LOGGING_CONFIG)
```

And to close logging, simply assign access_log=False:

```python 
if __name__ == "__main__":
  app.run(access_log=False)
```

This would skip calling logging functions when handling requests. And you could even do further in production to gain extra speed:

```python 
if __name__ == "__main__":
  # disable debug messages
  app.run(debug=False, access_log=False)
```
  
</p>
</details>

### 16.2.- Configuration

<details><summary>Show 16.2.- Configuration</summary>
<p>
  
By default, log_config parameter is set to use sanic.log.LOGGING_CONFIG_DEFAULTS dictionary for configuration.

There are three `loggers` used in sanic, and **must be defined if you want to create your own logging configuration**:

- root:
Used to log internal messages.
- sanic.error:
Used to log error logs.
- sanic.access:
Used to log access logs.

#### Log format

In addition to default parameters provided by python (asctime, levelname, message), Sanic provides additional parameters for access logger with:

- host (str)
request.ip

- request (str)
request.method + " " + request.url

- status (int)
response.status

- byte (int)
len(response.body)

The default access log format is

```python 
%(asctime)s - (%(name)s)[%(levelname)s][%(host)s]: %(request)s %(message)s %(status)d %(byte)d
```
  
</p>
</details>
  
</ul></ul></p>
</details>

## 17.- Testing

<details><summary>Show 17.- Testing</summary>
<p><ul><ul type="square">
  
Sanic endpoints can be tested locally using the `test_client` object, which depends on the additional aiohttp library. -https://aiohttp.readthedocs.io/en/stable/

The `test_client` exposes `get`, `post`, `put`, `delete`, `patch`, `head` and `options` methods for you to run against your application. A simple example (using pytest) is like follows:
  
```python 
# Import the Sanic app, usually created with Sanic(__name__)
from external_server import app

def test_index_returns_200():
    request, response = app.test_client.get('/')
    assert response.status == 200

def test_index_put_not_allowed():
    request, response = app.test_client.put('/')
    assert response.status == 405
```
Internally, each time you call one of the `test_client` methods, the Sanic app is run at `127.0.0.1:42101` and your test request is executed against your application, using `aiohttp`.

The `test_client` methods accept the following arguments and keyword arguments:

- `uri` (default `'/'`) A string representing the URI to test.

- `gather_request` (default `True`) A boolean which determines whether the original request will be returned by the function. If set to `True`, the return value is a tuple of `(request, response)`, if `False` only the response is returned.

- `server_kwargs` *(default `{}`) a dict of additional arguments to pass into `app.run` before the test request is run.

- `debug` (default `False`) A boolean which determines whether to run the server in debug mode.

The function further takes the `*request_args` and `**request_kwargs`, which are passed directly to the aiohttp ClientSession request.

For example, to supply data to a GET request, you would do the following:

```python 
def test_get_request_includes_data():
    params = {'key1': 'value1', 'key2': 'value2'}
    request, response = app.test_client.get('/', params=params)
    assert request.args.get('key1') == 'value1'
```

And to supply data to a JSON POST request:

```python 
def test_post_json_request_includes_data():
    data = {'key1': 'value1', 'key2': 'value2'}
    request, response = app.test_client.post('/', data=json.dumps(data))
    assert request.json.get('key1') == 'value1'
```

More information about the available arguments to aiohttp can be found in the documentation for ClientSession. -https://aiohttp.readthedocs.io/en/stable/client_reference.html#client-session
  
### pytest-sanic

<details><summary>Show pytest-sanic</summary>
<p>
  
pytest-sanic is a pytest plugin, it helps you to test your code asynchronously. Just write tests like - https://github.com/yunstanford/pytest-sanic
  
```python 
async def test_sanic_db_find_by_id(app):
    """
    Let's assume that, in db we have,
        {
            "id": "123",
            "name": "Kobe Bryant",
            "team": "Lakers",
        }
    """
    doc = await app.db["players"].find_by_id("123")
    assert doc.name == "Kobe Bryant"
    assert doc.team == "Lakers"
```

pytest-sanic also provides some useful fixtures, like loop, unused_port, test_server, test_client.

```python 
@pytest.yield_fixture
def app():
    app = Sanic("test_sanic_app")

    @app.route("/test_get", methods=['GET'])
    async def test_get(request):
        return response.json({"GET": True})

    @app.route("/test_post", methods=['POST'])
    async def test_post(request):
        return response.json({"POST": True})

    yield app


@pytest.fixture
def test_cli(loop, app, test_client):
    return loop.run_until_complete(test_client(app, protocol=WebSocketProtocol))


#########
# Tests #
#########

async def test_fixture_test_client_get(test_cli):
    """
    GET request
    """
    resp = await test_cli.get('/test_get')
    assert resp.status == 200
    resp_json = await resp.json()
    assert resp_json == {"GET": True}

async def test_fixture_test_client_post(test_cli):
    """
    POST request
    """
    resp = await test_cli.post('/test_post')
    assert resp.status == 200
    resp_json = await resp.json()
    assert resp_json == {"POST": True}
```

</p>
</details>
  
</ul></ul></p>
</details>

## 18.- Deploying

<details><summary>Show 18.- Deploying</summary>
<p><ul><ul type="square">
  
Deploying Sanic is made simple by the inbuilt webserver. After defining an instance of `sanic.Sanic`, we can call the `run` method with the following keyword arguments:

- `host` (default `"127.0.0.1"`): Address to host the server on.
- `port` (default `8000`): Port to host the server on.
- `debug` (default `False`): Enables debug output (slows server).
- `ssl` (default `None`): `SSLContext` for SSL encryption of worker(s).
- `sock` (default `None`): Socket for the server to accept connections from.
- `workers` (default `1`): Number of worker processes to spawn.
- `loop` (default `None`): An `asyncio`-compatible event loop. If none is specified, Sanic creates its own event loop.
- `protocol` (default `HttpProtocol`): Subclass of asyncio.protocol. - https://docs.python.org/3/library/asyncio-protocol.html#protocol-classes
  
### 18.1.- Workers

<details><summary>Show 18.1.- Workers</summary>
<p>
  
By default, Sanic listens in the main process using only one CPU core. To crank up the juice, just specify the number of workers in the `run` arguments.
  
```python 
app.run(host='0.0.0.0', port=1337, workers=4)
```

Sanic will automatically spin up multiple processes and route traffic between them. We recommend as many workers as you have available cores.
  
</p>
</details>

### 18.2.- Running via command

<details><summary>Show 18.2.- Running via command</summary>
<p>
  
If you like using command line arguments, you can launch a Sanic server by executing the module. For example, if you initialized Sanic as `app` in a file named `server.py`, you could run the server like so:

`python -m sanic server.app --host=0.0.0.0 --port=1337 --workers=4`

With this way of running sanic, it is not necessary to invoke `app.run` in your Python file. If you do, make sure you wrap it so that it only executes when directly run by the interpreter.
  
```python 
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=1337, workers=4)
```
  
</p>
</details>

### 18.3.- Running via Gunicorn

<details><summary>Show 18.3.- Running via Gunicorn</summary>
<p>
  
Gunicorn ‘Green Unicorn’ is a WSGI HTTP Server for UNIX. It’s a pre-fork worker model ported from Ruby’s Unicorn project.- http://gunicorn.org/

In order to run Sanic application with Gunicorn, you need to use the special `sanic.worker.GunicornWorker` for Gunicorn `worker-class` argument:
  
``` 
gunicorn myapp:app --bind 0.0.0.0:1337 --worker-class sanic.worker.GunicornWorker
```

If your application suffers from memory leaks, you can configure Gunicorn to gracefully restart a worker after it has processed a given number of requests. This can be a convenient way to help limit the effects of the memory leak.

See the Gunicorn Docs for more information. - http://docs.gunicorn.org/en/latest/settings.html#max-requests
  
</p>
</details>

### 18.4.- Asynchronous support 

<details><summary>Show 18.4.- Asynchronous support</summary>
<p>
  
This is suitable if you need to share the sanic process with other applications, in particular the `loop`. However be advised that this method does not support using multiple processes, and is not the preferred way to run the app in general.

Here is an incomplete example (please see `run_async.py` in examples for something more practical):
  
```python 
server = app.create_server(host="0.0.0.0", port=8000)
loop = asyncio.get_event_loop()
task = asyncio.ensure_future(server)
loop.run_forever()
```
  
</p>
</details>
  
</ul></ul></p>
</details>

## 19.- Extensions

<details><summary>Show 19.- Extensions</summary>
<p><ul><ul type="square">
  
A list of Sanic extensions created by the community.

- Sanic-Plugins-Framework: Library for easily creating and using Sanic plugins. - https://github.com/ashleysommer/sanicpluginsframework
- Sessions: Support for sessions. Allows using redis, memcache or an in memory store. - https://github.com/subyraman/sanic_session
- CORS: A port of flask-cors. - https://github.com/ashleysommer/sanic-cors
- Compress: Allows you to easily gzip Sanic responses. A port of Flask-Compress. - https://github.com/subyraman/sanic_compress
- Jinja2: Support for Jinja2 template. - https://github.com/lixxu/sanic-jinja2
- JWT: Authentication extension for JSON Web Tokens (JWT). - https://github.com/ahopkins/sanic-jwt
- OpenAPI/Swagger: OpenAPI support, plus a Swagger UI. - https://github.com/channelcat/sanic-openapi
- Pagination: Simple pagination support. - https://github.com/lixxu/python-paginate
- Motor: Simple motor wrapper. - https://github.com/lixxu/sanic-motor
- Sanic CRUD: CRUD REST API generation with peewee models. - https://github.com/Typhon66/sanic_crud
- UserAgent: Add `user_agent` to request - https://github.com/lixxu/sanic-useragent
- Limiter: Rate limiting for sanic. - https://github.com/bohea/sanic-limiter
- Sanic EnvConfig: Pull environment variables into your sanic config. - https://github.com/jamesstidard/sanic-envconfig
- Babel: Adds i18n/l10n support to Sanic applications with the help of the `Babel` library - https://github.com/lixxu/sanic-babel
- Dispatch: A dispatcher inspired by `DispatcherMiddleware` in werkzeug. Can act as a Sanic-to-WSGI adapter. - https://github.com/ashleysommer/sanic-dispatcher
- Sanic-OAuth: OAuth Library for connecting to & creating your own token providers.
- Sanic-nginx-docker-example: Simple and easy to use example of Sanic behined nginx using docker-compose. - https://github.com/itielshwartz/sanic-nginx-docker-example
- sanic-graphql: GraphQL integration with Sanic - https://github.com/graphql-python/sanic-graphql
- sanic-prometheus: Prometheus metrics for Sanic - https://github.com/dkruchinin/sanic-prometheus
- Sanic-RestPlus: A port of Flask-RestPlus for Sanic. Full-featured REST API with SwaggerUI generation. - https://github.com/ashleysommer/sanic-restplus
- sanic-transmute: A Sanic extension that generates APIs from python function and classes, and also generates Swagger UI/documentation automatically. - https://github.com/yunstanford/sanic-transmute
- pytest-sanic: A pytest plugin for Sanic. It helps you to test your code asynchronously. - https://github.com/yunstanford/pytest-sanic
- jinja2-sanic: a jinja2 template renderer for Sanic.(Documentation) - https://github.com/yunstanford/jinja2-sanic, http://jinja2-sanic.readthedocs.io/en/latest/
  
</ul></ul></p>
</details>

## 20.- Contributing

<details><summary>Show 20.- Contributing</summary>
<p><ul><ul type="square">
  
Thank you for your interest! Sanic is always looking for contributors. If you don't feel comfortable contributing code, adding docstrings to the source files is very appreciated.
  
### 20.1.- Installation

<details><summary>Show 20.1.- Installation</summary>
<p>
  
To develop on sanic (and mainly to just run the tests) it is highly recommend to install from sources.

So assume you have already cloned the repo and are in the working directory with a virtual environment already set up, then run:
  
``` 
python setup.py develop && pip install -r requirements-dev.txt
```

</p>
</details>

### 20.2.- Running tests

<details><summary>Show 20.2.- Running tests</summary>
<p>
  
To run the tests for sanic it is recommended to use tox like so:
  
``` 
tox
```

See it's that simple!

</p>
</details>

### 20.3.- Pull requests!

<details><summary>Show 20.3.- Pull requests!</summary>
<p>
  
So the pull request approval rules are pretty simple:

- All pull requests must pass unit tests
- All pull requests must be reviewed and approved by at least one current collaborator on the project
- All pull requests must pass flake8 checks
- If you decide to remove/change anything from any common interface a deprecation message should accompany it.
- If you implement a new feature you should have at least one unit test to accompany it.

</p>
</details>

### 20.4.- Documentation

<details><summary>Show 20.4.- Documentation</summary>
<p>
  
Sanic's documentation is built using sphinx. Guides are written in Markdown and can be found in the `docs` folder, while the module reference is automatically generated using `sphinx-apidoc`. - http://www.sphinx-doc.org/en/1.5.1/

To generate the documentation from scratch:
  
``` 
sphinx-apidoc -fo docs/_api/ sanic
sphinx-build -b html docs docs/_build
```

The HTML documentation will be created in the `docs/_build` folder.

</p>
</details>

### 20.5.- Warning

<details><summary>Show 20.5.- Warning</summary>
<p>
  
One of the main goals of Sanic is speed. Code that lowers the performance of Sanic without significant gains in usability, security, or features may not be merged. Please don't let this intimidate you! If you have any concerns about an idea, open an issue for discussion and help.

</p>
</details>
  
</ul></ul></p>
</details>

## 21.- API Reference

<details><summary>Show 21.- API Reference</summary>
<p><ul><ul type="square">
  
### Submodules
  
### 21.1.- sanic.app module
<details><summary>Show 21.1.- sanic.app module</summary>
<p>
  
> class **sanic.app.Sanic** *(name=None, router=None, error_handler=None, load_env=True, request_class=None, strict_slashes=False, log_config=None, configure_logging=True)*

<ul type="square">

Bases: `object`

> **add_route** *(handler, uri, methods=frozenset({'GET'}), host=None, strict_slashes=None, version=None, name=None)*

<ul type="square">

A helper method to register class instance or functions as a handler to the application url routes.

<ul type="square">

**Parameters**:	 
- **handler** 
– function or class instance
- **uri** – path of the URL
- **methods** – list or tuple of methods allowed, these are overridden if using a HTTPMethodView
- **host** –
- **strict_slashes** –
- **version** –
- **name** – user defined route name for url_for



**Returns**:	
function or class instance

</ul></ul>

> **add_task** *(task)*

<ul type="square">

Schedule a task to run later, after the loop has started. Different from asyncio.ensure_future in that it does not also return a future, and the actual ensure_future call is delayed until before server start.

<ul type="square">

**Parameters:	task** – future, couroutine or awaitable

</ul></ul>

> **add_websocket_route** *(handler, uri, host=None, strict_slashes=None, name=None)*

<ul type="square">

A helper method to register a function as a websocket route.

</ul>

> **blueprint** *(blueprint,* \** *options)*

<ul type="square">

Register a blueprint on the application.

<ul type="square">

**Parameters**:	

- **blueprint** – Blueprint object
- **options** – option dictionary with blueprint defaults

**Returns**:	Nothing

</ul>

</ul>

> **converted_response_type** *(response)*

> coroutine **create_server** *(host=None, port=None, debug=False, ssl=None, sock=None, protocol=None, backlog=100, stop_event=None, access_log=True)*

<ul type="square">

Asynchronous version of run.

> NOTE: This does not support multiprocessing and is not the preferred

<ul type="square">

way to run a Sanic application.

</ul>

</ul>

> **delete** *(uri, host=None, strict_slashes=None, version=None, name=None)*

> **enable_websocket** *(enable=True)*

<ul type="square">

Enable or disable the support for websocket.

Websocket is enabled automatically if websocket routes are added to the application.

</ul>

> **exception** *(* * *exceptions)*

<ul type="square">

Decorate a function to be registered as a handler for exceptions

<ul type="square">

**Parameters:	exceptions** – exceptions

**Returns:**	decorated function

</ul></ul>

> **get** *(uri, host=None, strict_slashes=None, version=None, name=None)*

> *coroutine* **handle_request** *(request, write_callback, stream_callback)*

<ul type="square">

Take a request from the HTTP Server and return a response object to be sent back The HTTP Server only expects a response object, so exception handling must be done here

<ul type="square">

**Parameters**:	

- **request** – HTTP Request object
- **write_callback** – Synchronous response function to be called with the response as the only argument
- **stream_callback** – Coroutine that handles streaming a StreamingHTTPResponse if produced by the handler.

**Returns**:	Nothing

</ul></ul>

> **head** *(uri, host=None, strict_slashes=None, version=None, name=None)*

> **listener** *(event)*

<ul type="square">

Create a listener from a decorated function.

<ul type="square">

**Parameters:	event** – event to listen to

</ul></ul>

> **loop**

<ul type="square">

Synonymous with asyncio.get_event_loop().

Only supported when using the app.run method.

</ul>

> **middleware** *(middleware_or_request)*

<ul type="square">

Decorate and register middleware to be called before a request. Can either be called as @app.middleware or @app.middleware(‘request’)

</ul>

> **options** *(uri, host=None, strict_slashes=None, version=None, name=None)*

> **patch** *(uri, host=None, strict_slashes=None, stream=False, version=None, name=None)*

> **post** *(uri, host=None, strict_slashes=None, stream=False, version=None, name=None)*

> **put** *(uri, host=None, strict_slashes=None, stream=False, version=None, name=None)*

> **register_blueprint** *(* *args, ** kwargs *)*

> **register_middleware** *(middleware, attach_to='request')*

> **remove_route** *(uri, clean_cache=True, host=None)*

> **route** *(uri, methods=frozenset({'GET'}), host=None, strict_slashes=None, stream=False, version=None, name=None)*

<ul type="square">

Decorate a function to be registered as a route

<ul type="square">

**Parameters**:

- **uri** – path of the URL
- **methods** – list or tuple of methods allowed
- **host** –
- **strict_slashes** –
- **stream** –
- **version** –
- **name** – user defined route name for url_for

**Returns**:	decorated function

</ul></ul>

> **run** *(host=None, port=None, debug=False, ssl=None, sock=None, workers=1, protocol=None, backlog=100, stop_event=None, register_sys_signals=True, access_log=True)*

<ul type="square">

Run the HTTP Server and listen until keyboard interrupt or term signal. On termination, drain connections before closing.

<ul type="square">

**Parameters**:	

- **host** – Address to host on
- **port** – Port to host on
- **debug** – Enables debug output (slows server)
- **ssl** – SSLContext, or location of certificate and key for SSL encryption of worker(s)
- **sock** – Socket for the server to accept connections from
- **workers** – Number of processes received before it is respected
- **backlog** –
- **stop_event** –
- **register_sys_signals** –
- **protocol** – Subclass of asyncio protocol class

**Returns**:	Nothing

</ul></ul>

> **static** *(uri, file_or_directory, pattern='/?.+', use_modified_since=True, use_content_range=False, stream_large_files=False, name='static', host=None, strict_slashes=None)*

<ul type="square">

Register a root to serve files from. The input can either be a file or a directory. See

</ul>

> **stop**()

<ul type="square">

This kills the Sanic

</ul>

> **test_client**

> *coroutine* **trigger_events** *(events, loop)*

<ul type="square">

Trigger events (functions or async) :param events: one or more sync or async functions to execute :param loop: event loop

</ul>

> **url_for** *(view_name: str,* ** kwargs *)*

<ul type="square">

Build a URL based on a view name and the values provided.

In order to build a URL, all request parameters must be supplied as keyword arguments, and each parameter must pass the test for the specified parameter type. If these conditions are not met, a URLBuildError will be thrown.

Keyword arguments that are not request parameters will be included in the output URL’s query string.

<ul type="square">

**Parameters**:	

- **view_name** – string referencing the view name
- **\**kwargs** – keys and values that are used to build request parameters and query string arguments.

**Returns**:	the built URL

</ul>

`Raises:`

<ul type="square">

URLBuildError

</ul></ul>

> **websocket** *(uri, host=None, strict_slashes=None, subprotocols=None, name=None)*

<ul type="square">

Decorate a function to be registered as a websocket route :param uri: path of the URL :param subprotocols: optional list of strings with the supported

<ul type="square">

subprotocols

</ul>

**Parameters:	host** –

**Returns:**	decorated function

</ul></ul>

  
</p>
</details>

### 21.2.- sanic.blueprints module
<details><summary>Show 21.2.- sanic.blueprints module</summary>
<p>
  
> *class* **sanic.blueprints.Blueprint** *(name, url_prefix=None, host=None, version=None, strict_slashes=False)*

<ul type="square">

Bases: `object`

> **add_route** *(handler, uri, methods=frozenset({'GET'}), host=None, strict_slashes=None, version=None, name=None)*

<ul type="square">

Create a blueprint route from a function.

<ul type="square">

**Parameters:**	

- **handler** – function for handling uri requests. Accepts function, or class instance with a view_class method.
- **uri** – endpoint at which the route will be accessible.
- **methods** – list of acceptable HTTP methods.
- **host** –
- **strict_slashes** –
- **version** –
- **name** – user defined route name for url_for

**Returns:**	function or class instance

</ul></ul>

> **add_websocket_route** *(handler, uri, host=None, version=None, name=None)*

<ul type="square">

Create a blueprint websocket route from a function.

<ul type="square">

**Parameters:**

- handler – function for handling uri requests. Accepts function, or class instance with a view_class method.
- uri – endpoint at which the route will be accessible.

**Returns:**	function or class instance

</ul></ul>

> **delete** *(uri, host=None, strict_slashes=None, version=None, name=None)*

> **exception** *(* * *args,* ** *kwargs)*

<ul type="square">

Create a blueprint exception from a decorated function.

</ul>

> **get** *(uri, host=None, strict_slashes=None, version=None, name=None)*

> **head** *(uri, host=None, strict_slashes=None, version=None, name=None)*

> **listener** *(event)*

<ul type="square">

Create a listener from a decorated function.

<ul type="square">

**Parameters:	event** – Event to listen to.

</ul></ul>

> **middleware** *(* * *args,* ** *kwargs)*

<ul type="square">

Create a blueprint middleware from a decorated function.

</ul>

> **options** *(uri, host=None, strict_slashes=None, version=None, name=None)*

> **patch** *(uri, host=None, strict_slashes=None, stream=False, version=None, name=None)*

> **post** *(uri, host=None, strict_slashes=None, stream=False, version=None, name=None)*

> **put** *(uri, host=None, strict_slashes=None, stream=False, version=None, name=None)*

> **register** *(app, options)*

<ul type="square">

Register the blueprint to the sanic app.

</ul>

> **route** *(uri, methods=frozenset({'GET'}), host=None, strict_slashes=None, stream=False, version=None, name=None)*

<ul type="square">

Create a blueprint route from a decorated function.

<ul type="square">

**Parameters:**

- **uri** – endpoint at which the route will be accessible.
- **methods** – list of acceptable HTTP methods.

</ul></ul>

> **static** *(uri, file_or_directory,* * *args,* ** *kwargs)*

<ul type="square">

Create a blueprint static route from a decorated function.

<ul type="square">

**Parameters:**

- **uri** – endpoint at which the route will be accessible.
- **file_or_directory** – Static asset.

</ul></ul>

> **websocket** *(uri, host=None, strict_slashes=None, version=None, name=None)*

<ul type="square">

Create a blueprint websocket route from a decorated function.

<ul type="square">

**Parameters:	uri** – endpoint at which the route will be accessible.

</ul></ul>

> **sanic.blueprints.FutureException**

<ul type="square">

alias of `Route`

</ul>

> **sanic.blueprints.FutureListener**

<ul type="square">

alias of `Listener`

</ul>

> **sanic.blueprints.FutureMiddleware**

<ul type="square">

alias of `Route`

</ul>

> **sanic.blueprints.FutureRoute**

<ul type="square">

alias of `Route`

</ul>

> **sanic.blueprints.FutureStatic**

<ul type="square">

alias of `Route`

</ul>
  
</p>
</details>

### 21.3.- sanic.config module
<details><summary>Show 21.3.- sanic.config module</summary>
<p>
  
> *class* **sanic.config.Config** *(defaults=None, load_env=True, keep_alive=True)*

<ul type="square">

Bases: `dict`

> **from_envvar** *(variable_name)*

<ul type="square">

Load a configuration from an environment variable pointing to a configuration file.

<ul type="square">

**Parameters:	variable_name** – name of the environment variable

**Returns:**	bool. `True` if able to load config, `False` otherwise.

</ul></ul>

> **from_object** *(obj)*

<ul type="square">

Update the values from the given object. Objects are usually either modules or classes.

Just the uppercase variables in that object are stored in the config. Example usage:

```python 
from yourapplication import default_config
app.config.from_object(default_config)
```

You should not use this function to load the actual configuration but rather configuration defaults. The actual config should be loaded with `from_pyfile()` and ideally from a location not within the package because the package might be installed system wide.

<ul type="square">
  
**Parameters:	obj** – an object holding the configuration

</ul></ul>

> **from_pyfile** *(filename)*

<ul type="square">

Update the values in the config from a Python file. Only the uppercase variables in that module are stored in the config.

<ul type="square">

**Parameters:	filename** – an absolute path to the config file

</ul></ul>

> **load_environment_vars** *(prefix='SANIC_')*

<ul type="square">

Looks for prefixed environment variables and applies them to the configuration if present.

</ul>
  
</p>
</details>

### 21.4.- sanic.constants module

### 21.5.- sanic.cookies module

<details><summary>Show 21.5.- sanic.cookies module</summary>
<p>
  
> *class* **sanic.cookies.Cookie** *(key, value)*

<ul type="square">

Bases: `dict`

A stripped down version of Morsel from SimpleCookie #gottagofast

> **encode** *(encoding)*

</ul>

> *class* **sanic.cookies.CookieJar** *(headers)*

<ul type="square">

Bases: `dict`

CookieJar dynamically writes headers as cookies are added and removed It gets around the limitation of one header per name by using the MultiHeader class to provide a unique key that encodes to Set-Cookie.

</ul>

> *class* **sanic.cookies.MultiHeader** *(name)*

Bases: `object`

String-holding object which allow us to set a header within response that has a unique key, but may contain duplicate header names

> **encode**()
  
</p>
</details>

### 21.6.- sanic.exceptions module

<details><summary>Show 21.6.- sanic.exceptions module</summary>
<p>
  
> *exception* **sanic.exceptions.ContentRangeError** *(message, content_range)*

<ul type="square">

Bases: `sanic.exceptions.SanicException`

> **status_code** *= 416*

</ul>

> *exception* **sanic.exceptions.FileNotFound** *(message, path, relative_url)*

<ul type="square">

Bases: `sanic.exceptions.NotFound`

</ul>

> *exception* **sanic.exceptions.Forbidden** *(message, status_code=None)*

<ul type="square">

Bases: `sanic.exceptions.SanicException`

> **status_code** *= 403*

</ul>

> *exception* **sanic.exceptions.HeaderNotFound** *(message, status_code=None)*

<ul type="square">

Bases: `sanic.exceptions.InvalidUsage`

</ul>

> *exception* **sanic.exceptions.InvalidRangeType** *(message, content_range)*

<ul type="square">
  
Bases: `sanic.exceptions.ContentRangeError`

</ul>

> *exception* **sanic.exceptions.InvalidUsage** *(message, status_code=None)*

<ul type="square">
  
Bases: `sanic.exceptions.SanicException`

> **status_code** *= 400*

</ul>

> *exception* **sanic.exceptions.MethodNotSupported** *(message, method, allowed_methods)*

<ul type="square">

Bases: `sanic.exceptions.SanicException`

> **status_code** *= 405*

</ul>

> *exception* **sanic.exceptions.NotFound** *(message, status_code=None)*

<ul type="square">

Bases: `sanic.exceptions.SanicException`

> **status_code** *= 404*

</ul>

> *exception* **sanic.exceptions.PayloadTooLarge** *(message, status_code=None)*

<ul type="square">

Bases: `sanic.exceptions.SanicException`

> **status_code** *= 413*

</ul>

> *exception* **sanic.exceptions.RequestTimeout** *(message, status_code=None)*

<ul type="square">

Bases: `sanic.exceptions.SanicException`

The Web server (running the Web site) thinks that there has been too long an interval of time between 1) the establishment of an IP connection (socket) between the client and the server and 2) the receipt of any data on that socket, so the server has dropped the connection. The socket connection has actually been lost - the Web server has ‘timed out’ on that particular socket connection.

> **status_code** *= 408*

</ul>

> *exception* **sanic.exceptions.SanicException** *(message, status_code=None)*

<ul type="square">

Bases: `Exception`

</ul>

> *exception* **sanic.exceptions.ServerError** *(message, status_code=None)*

<ul type="square">

Bases: `sanic.exceptions.SanicException`

> **status_code** *= 500*

</ul>

> *exception* **sanic.exceptions.ServiceUnavailable** *(message, status_code=None)*

<ul type="square">

Bases: `sanic.exceptions.SanicException`

The server is currently unavailable (because it is overloaded or down for maintenance). Generally, this is a temporary state.

> **status_code** *= 503*

</ul>

> *exception* **sanic.exceptions.URLBuildError** *(message, status_code=None)*

<ul type="square">

Bases: `sanic.exceptions.ServerError`

</ul>

> *exception* **sanic.exceptions.Unauthorized** *(message, status_code=None, scheme=None,* ** *kwargs)*

<ul type="square">

> Bases: `sanic.exceptions.SanicException`

Unauthorized exception (401 HTTP status code).

**Parameters:**

- **message** – Message describing the exception.
- **status_code** – HTTP Status code.
- **scheme** – Name of the authentication scheme to be used.

When present, kwargs is used to complete the WWW-Authentication header.

Examples:

```python 
# With a Basic auth-scheme, realm MUST be present:
raise Unauthorized("Auth required.",
                   scheme="Basic",
                   realm="Restricted Area")

# With a Digest auth-scheme, things are a bit more complicated:
raise Unauthorized("Auth required.",
                   scheme="Digest",
                   realm="Restricted Area",
                   qop="auth, auth-int",
                   algorithm="MD5",
                   nonce="abcdef",
                   opaque="zyxwvu")

# With a Bearer auth-scheme, realm is optional so you can write:
raise Unauthorized("Auth required.", scheme="Bearer")

# or, if you want to specify the realm:
raise Unauthorized("Auth required.",
                   scheme="Bearer",
                   realm="Restricted Area")
```

> **status_code** *= 401*

</ul>

> **sanic.exceptions.abort** *(status_code, message=None)*

<ul type="square">

Raise an exception based on SanicException. Returns the HTTP response message appropriate for the given status code, unless provided.

**Parameters:**

- **status_code** – The HTTP status code to return.
- **message** – The HTTP response body. Defaults to the messages in response.py for the given status code.

</ul>

> **sanic.exceptions.add_status_code** *(code)*

<ul type="square">

Decorator used for adding exceptions to _sanic_exceptions.

</ul>
  
</p>
</details>

### 21.7.- sanic.handlers module
<details><summary>Show 21.7.- sanic.handlers module</summary>
<p>
  
> *class* **sanic.handlers.ContentRangeHandler** *(request, stats)*

<ul type="square">

Bases: `object`

Class responsible for parsing request header

> **end**

> **headers**

> **size**

> **start**

> **total**

</ul>

> *class* **sanic.handlers.ErrorHandler**

<ul type="square">

Bases: `object`

> **add** *(exception, handler)*

> **cached_handlers** *= None*

> **default** *(request, exception)*

> **handlers** *= None*

> **log** *(message, level='error')*

<ul type="square">

Override this method in an ErrorHandler subclass to prevent logging exceptions.

</ul>

> **lookup** *(exception)*

> **response** *(request, exception)*

<ul type="square">

Fetches and executes an exception handler and returns a response object

<ul type="square">

**Parameters:**

- **request** – Request
- **exception** – Exception to handle

**Returns:**	Response object

</ul></ul></ul>
  
</p>
</details>

### 21.8.- sanic.log module

### 21.9.- sanic.request module

<details><summary>Show 21.9.- sanic.request module</summary>
<p>
  
> *class* **sanic.request.File** *(type, body, name)*

<ul type="square">

Bases: `tuple`

> **body**

<ul type="square">

Alias for field number 1

</ul>

> **name**

<ul type="square">

Alias for field number 2

</ul>

> **type**

<ul type="square">

Alias for field number 0

</ul></ul>

> *class* **sanic.request.Request** *(url_bytes, headers, version, method, transport)*

Bases: `dict`

<ul type="square">

Properties of an HTTP request such as URL, headers, etc.

> **app**

> **args**

> **body**

> **content_type**

> **cookies**

> **files**

> **form**

> **headers**

> **host**

> **ip**

> **json**

> **load_json** *(loads=<built-in function loads>)*

> **match_info**

<ul type="square">

return matched info after resolving route

</ul>

> **method**

> **parsed_args**

> **parsed_files**

> **parsed_form**

> **parsed_json**

> **path**

> **port**

> **query_string**

> **raw_args**

> **remote_addr**

<ul type="square">

Attempt to return the original client ip based on X-Forwarded-For.

<ul type="square">

**Returns:**	original client ip.

</ul></ul>

> **scheme**

> **socket**

> **stream**

> **token**

<ul type="square">

Attempt to return the auth header token.

<ul type="square">

**Returns:**	token related to request

</ul></ul>

> **transport**

> **uri_template**

> **url**

> **version**

</ul>

> *class* **sanic.request.RequestParameters**

<ul type="square">

Bases: `dict`

Hosts a dict with lists as values where get returns the first value of the list and getlist returns the whole shebang

> **get** *(name, default=None)*

<ul type="square">

Return the first value, either the default or actual

</ul>

> **getlist** *(name, default=None)*

<ul type="square">

Return the entire list

</ul></ul>

> **sanic.request.parse_multipart_form** *(body, boundary)*

<ul type="square">

Parse a request body and returns fields and files

<ul type="square">

**Parameters:**

- **body** – bytes request body
- **boundary** – bytes multipart boundary

**Returns:**	fields (RequestParameters), files (RequestParameters)

</ul></ul>
  
</p>
</details>

### 21.10.- sanic.response module

<details><summary>Show 21.10.- sanic.response module</summary>
<p>
  
> *class* **sanic.response.BaseHTTPResponse**

<ul type="square">

Bases: `object`

> **cookies**

</ul>

> *class* **sanic.response.HTTPResponse** *(body=None, status=200, headers=None, content_type='text/plain', body_bytes=b'')*

<ul type="square">

Bases: `sanic.response.BaseHTTPResponse`

> **body**

> **content_type**

> **cookies** 

> **headers**

> **output** *(version='1.1', keep_alive=False, keep_alive_timeout=None)*

> **status** 

</ul>

> *class* **sanic.response.StreamingHTTPResponse** *(streaming_fn, status=200, headers=None, content_type='text/plain')*

Bases: `sanic.response.BaseHTTPResponse`

> **content_type**

> **get_headers** *(version='1.1', keep_alive=False, keep_alive_timeout=None)*

> **headers**

> **status**

> *coroutine* **stream** *(version='1.1', keep_alive=False, keep_alive_timeout=None)*

<ul type="square">

Streams headers, runs the streaming_fn callback that writes content to the response body, then finalizes the response body.

</ul>

> **streaming_fn**

> **transport**

> **write** *(data)*

<ul type="square">

Writes a chunk of data to the streaming response.

<ul type="square">

**Parameters:	data** – bytes-ish data to be written.

</ul></ul>

> *coroutine* **sanic.response.file** *(location, mime_type=None, headers=None, filename=None, _range=None)*

<ul type="square">

Return a response object with file data.

<ul type="square">

**Parameters:**

- **location** – Location of file on system.
- **mime_type** – Specific mime_type.
- **headers** – Custom Headers.
- **filename** – Override filename.
- **_range** –

</ul></ul>

> *coroutine* **sanic.response.file_stream** *(location, chunk_size=4096, mime_type=None, headers=None, filename=None, _range=None)*

<ul type="square">

Return a streaming response object with file data.

<ul type="square">

**Parameters:	**

- **location** – Location of file on system.
- **chunk_size** – The size of each chunk in the stream (in bytes)
- **mime_type** – Specific mime_type.
- **headers** – Custom Headers.
- **filename** – Override filename.
- **_range** –

</ul></ul>

> **sanic.response.html** *(body, status=200, headers=None)*

<ul type="square">

Returns response object with body in html format.

<ul type="square">

**Parameters:**

- **body** – Response data to be encoded.
- **status** – Response code.
- **headers** – Custom Headers.

</ul></ul>

> **sanic.response.json** *(body, status=200, headers=None, content_type='application/json', dumps=<built-in function dumps>,* ** *kwargs)*

<ul type="square">

Returns response object with body in json format.

<ul type="square">

**Parameters:**

- **body** – Response data to be serialized.
- **status** – Response code.
- **headers** – Custom Headers.
- **kwargs** – Remaining arguments that are passed to the json encoder.

</ul></ul>

> **sanic.response.raw** *(body, status=200, headers=None, content_type='application/octet-stream')*

<ul type="square">

Returns response object without encoding the body.

<ul type="square">

**Parameters:**	

- **body** – Response data.
- **status** – Response code.
- **headers** – Custom Headers.
- **content_type** – the content type (string) of the response.

</ul></ul>

> **sanic.response.redirect** *(to, headers=None, status=302, content_type='text/html; charset=utf-8')*

<ul type="square">

Abort execution and cause a 302 redirect (by default).

<ul type="square">

**Parameters:**

- **to** – path or fully qualified URL to redirect to
- **headers** – optional dict of headers to include in the new request
- **status** – status code (int) of the new request, defaults to 302
- **content_type** – the content type (string) of the response

**Returns:**	the redirecting Response

</ul></ul>

> **sanic.response.stream** *(streaming_fn, status=200, headers=None, content_type='text/plain; charset=utf-8')*

<ul type="square">

Accepts an coroutine streaming_fn which can be used to write chunks to a streaming response. Returns a StreamingHTTPResponse.

Example usage:

```python 
@app.route("/")
async def index(request):
    async def streaming_fn(response):
        await response.write('foo')
        await response.write('bar')

    return stream(streaming_fn, content_type='text/plain')
```
    
<ul type="square">
    
**Parameters:**

- **streaming_fn** – A coroutine accepts a response and writes content to that response.
- **mime_type** – Specific mime_type.
- **headers** – Custom Headers.

</ul></ul>

> **sanic.response.text** *(body, status=200, headers=None, content_type='text/plain; charset=utf-8')*

<ul type="square">

Returns response object with body in text format.

<ul type="square">

**Parameters:**

- **body** – Response data to be encoded.
- **status** – Response code.
- **headers** – Custom Headers.
- **content_type** – the content type (string) of the response

</ul></ul>
  
</p>
</details>

### 21.11.- sanic.router module

<details><summary>Show 21.11.- sanic.router module</summary>
<p>
  
> *class* **sanic.router.Parameter** *(name, cast)*

<ul type="square">

Bases: `tuple`

> **cast**

<ul type="square">

Alias for field number 1

</ul>

> **name**

<ul type="square">

Alias for field number 0

</ul></ul>

> *class* **sanic.router.Route** *(handler, methods, pattern, parameters, name, uri)*

<ul type="square">

Bases: `tuple`

> **handler**

<ul type="square">

Alias for field number 0

</ul>

> **methods**

<ul type="square">

Alias for field number 1

</ul>

> **name**

<ul type="square">

Alias for field number 4

</ul>

> **parameters**

<ul type="square">

Alias for field number 3

</ul>

> **pattern**

<ul type="square">

Alias for field number 2

</ul>

> **uri**

<ul type="square">
  
Alias for field number 5

</ul></ul>

> *exception* **sanic.router.RouteDoesNotExist**

<ul type="square">

Bases: `Exception`

</ul>

> *exception* **sanic.router.RouteExists**

<ul type="square">

Bases: `Exception`

</ul>

> *class* **sanic.router.Router**

<ul type="square">

Bases: `object`

Router supports basic routing with parameters and method checks

Usage:

```python 
@sanic.route('/my/url/<my_param>', methods=['GET', 'POST', ...])
def my_route(request, my_param):
    do stuff...
```

or

```python 
@sanic.route('/my/url/<my_param:my_type>', methods['GET', 'POST', ...])
def my_route_with_type(request, my_param: my_type):
    do stuff...
```

Parameters will be passed as keyword arguments to the request handling function. Provided parameters can also have a type by appending :type to the <parameter>. Given parameter must be able to be type-casted to this. If no type is provided, a string is expected. A regular expression can also be passed in as the type. The argument given to the function will always be a string, independent of the type.

> **add** *(uri, methods, handler, host=None, strict_slashes=False, version=None, name=None)*

<ul type="square">

Add a handler to the route list

<ul type="square">

**Parameters:**

- **uri** – path to match
- **methods** – sequence of accepted method names. If none are provided, any method is allowed
- **handler** – request handler function. When executed, it should provide a response object.
- **strict_slashes** – strict to trailing slash
- **version** – current version of the route or blueprint. See docs for further details.

**Returns:**	Nothing

</ul></ul>

> *static* **check_dynamic_route_exists** *(pattern, routes_to_check)*

> **find_route_by_view_name**

<ul type="square">

Find a route in the router based on the specified view name.

<ul type="square">

**Parameters:**

- **view_name** – string of view name to search by
- **kwargs** – additional params, usually for static files

**Returns:**	tuple containing (uri, Route)

</ul></ul>

> **get** *(request)*

<ul type="square">

Get a request handler based on the URL of the request, or raises an error

<ul type="square">

**Parameters:	request** – Request object
**Returns:**	handler, arguments, keyword arguments

</ul></ul>

> **get_supported_methods** *(url)*

<ul type="square">

Get a list of supported methods for a url and optional host.

<ul type="square">

**Parameters:	url** – URL string (including host)

**Returns:**	frozenset of supported methods

</ul></ul>

> **is_stream_handler** *(request)*

<ul type="square">

Handler for request is stream or not. :param request: Request object :return: bool

</ul>

> **parameter_pattern** *= re.compile('<(.+?)>')*

> *classmethod* **parse_parameter_string** *(parameter_string)*

<ul type="square">

Parse a parameter string into its constituent name, type, and pattern

For example:

```python 
parse_parameter_string('<param_one:[A-z]>')` ->
    ('param_one', str, '[A-z]')
```

<ul type="square">

**Parameters:	parameter_string** – String to parse

**Returns:**	tuple containing (parameter_name, parameter_type, parameter_pattern)

</ul></ul>

> **remove** *(uri, clean_cache=True, host=None)*

> **routes_always_check** *= None*

> **routes_dynamic** *= None*

> **routes_static** *= None*

</ul>

> **sanic.router.url_hash** *(url)*
  
</p>
</details>

### 21.12.- sanic.server module

<details><summary>Show 21.12.- sanic.server module</summary>
<p>
  
> *class* **sanic.server.CIDict**

<ul type="square">

Bases: `dict`

Case Insensitive dict where all keys are converted to lowercase This does not maintain the inputted case when calling items() or keys() in favor of speed, since headers are case insensitive

> **get** *(key, default=None)*  

</ul>

> *class* **sanic.server.HttpProtocol** ( * *, loop, request_handler, error_handler, signal=<sanic.server.Signal object>, connections=set(), request_timeout=60, response_timeout=60, keep_alive_timeout=5, request_max_size=None, request_class=None, access_log=True, keep_alive=True, is_request_stream=False, router=None, state=None, debug=False,* ** *kwargs)*

<ul type="square">

Bases: `asyncio.protocols.Protocol`

> **access_log**

> **bail_out** *(message, from_error=False)*

> **cleanup**()

<ul type="square">

This is called when KeepAlive feature is used, it resets the connection in order for it to be able to handle receiving another request on the same connection.

</ul>

> **close**()

<ul type="square">

Force close the connection.

</ul>

> **close_if_idle**()

<ul type="square">

Close the connection if a request is not being sent or received

<ul type="square">

**Returns:**	boolean - True if closed, false if staying open

</ul></ul>

> **connection_lost** *(exc)*

> **connection_made** *(transport)*

> **connections**

> **data_received** *(data)*

> **execute_request_handler**()

> **headers**

> **is_request_stream**

> **keep_alive**

> **keep_alive_timeout**

> **keep_alive_timeout_callback**()

> **log_response** *(response)*

> **loop**

> **on_body** *(body)*

> **on_header** *(name, value)*

> **on_headers_complete**()

> **on_message_complete**()

> **on_url** *(url)*

> **parser**

> **request**

> **request_class**

> **request_handler**

> **request_max_size**

> **request_timeout**

> **request_timeout_callback**()

> **response_timeout**

> **response_timeout_callback**()

> **router**

> **signal**

> *coroutine* **stream_response** *(response)* 

<ul type="square">

Streams a response to the client asynchronously. Attaches the transport to the response so the response consumer can write to the response as needed.

</ul>

> **transport**

> **url**

> **write_error** *(exception)*

> **write_response** *(response)*

<ul type="square">

Writes response content synchronously to the transport.

</ul></ul>

> *class* **sanic.server.Signal**

<ul type="square">

Bases: `object`

> **stopped** *= False*

</ul>

> **sanic.server.serve** *(host, port, request_handler, error_handler, before_start=None, after_start=None, before_stop=None, after_stop=None, debug=False, request_timeout=60, response_timeout=60, keep_alive_timeout=5, ssl=None, sock=None, request_max_size=None, reuse_port=False, loop=None, protocol=<class 'sanic.server.HttpProtocol'>, backlog=100, register_sys_signals=True, run_async=False, connections=None, signal=<sanic.server.Signal object>, request_class=None, access_log=True, keep_alive=True, is_request_stream=False, router=None, websocket_max_size=None, websocket_max_queue=None, state=None, graceful_shutdown_timeout=15.0)*

<ul type="square">

Start asynchronous HTTP Server on an individual process.

<ul type="square">

**Parameters:**	

- **host** – Address to host on
- **port** – Port to host on
- **request_handler** – Sanic request handler with middleware
- **error_handler** – Sanic error handler with middleware
- **before_start** – function to be executed before the server starts listening. Takes arguments app instance and loop
- **after_start** – function to be executed after the server starts listening. Takes arguments app instance and loop
- **before_stop** – function to be executed when a stop signal is received before it is respected. Takes arguments app instance and loop
- **after_stop** – function to be executed when a stop signal is received after it is respected. Takes arguments app instance and loop
- **debug** – enables debug output (slows server)
- **request_timeout** – time in seconds
- **response_timeout** – time in seconds
- **keep_alive_timeout** – time in seconds
- **ssl** – SSLContext
- **sock** – Socket for the server to accept connections from
- **request_max_size** – size in bytes, None for no limit
- **reuse_port** – True for multiple workers
- **loop** – asyncio compatible event loop
- **protocol** – subclass of asyncio protocol class
- **request_class** – Request class to use
- **access_log** – disable/enable access log
- **is_request_stream** – disable/enable Request.stream
- **router** – Router object

**Returns:**	Nothing

</ul></ul>

> **sanic.server.serve_multiple** *(server_settings, workers)*

<ul type="square">

Start multiple server processes simultaneously. Stop on interrupt and terminate signals, and drain connections when complete.

<ul type="square">

**Parameters:**

- **server_settings** – kw arguments to be passed to the serve function
- **workers** – number of workers to launch
- **stop_event** – if provided, is used as a stop signal

**Returns:**	

</ul></ul>

> **sanic.server.trigger_events** *(events, loop)*

<ul type="square">

Trigger event callbacks (functions or async)

**Parameters:**

- **events** – one or more sync or async functions to execute
- **loop** – event loop

</ul>

> **sanic.server.update_current_time** *(loop)*

<ul type="square">

Cache the current time, since it is needed at the end of every keep-alive request to update the request timeout time

<ul type="square">

**Parameters:	loop** –

**Returns:**

</ul></ul>
  
</p>
</details>

### 21.13.- sanic.static module

<details><summary>Show 21.13.- sanic.static module</summary>
<p>
  
> **sanic.static.register** *(app, uri, file_or_directory, pattern, use_modified_since, use_content_range, stream_large_files, name='static', host=None, strict_slashes=None)*

<ul type="square">

Register a static directory handler with Sanic by adding a route to the router and registering a handler.

<ul type="square">

**Parameters:**

- **app** – Sanic
- **file_or_directory** – File or directory path to serve from
- **uri** – URL to serve from
- **pattern** – regular expression used to match files in the URL
- **use_modified_since** – If true, send file modified time, and return not modified if the browser’s matches the server’s
- **use_content_range** – If true, process header for range requests and sends the file part that is requested
- **stream_large_files** – If true, use the file_stream() handler rather than the file() handler to send the file If this is an integer, this represents the threshold size to switch to file_stream()
- **name** – user defined name used for url_for

</ul></ul>
  
</p>
</details>

### 21.14.- sanic.testing module

<details><summary>Show 21.14.- sanic.testing module</summary>
<p>
  
> *class* **sanic.testing.SanicTestClient** *(app, port=42101)*

<ul type="square">

Bases: `object`

> **delete** ( * *args,* ** *kwargs)*

> **get** ( * *args,* ** *kwargs)*

> **head** ( * *args,* ** *kwargs)*

> **options** ( * *args,* ** *kwargs)*

> **patch** ( * *args,* ** *kwargs)*

> **post** ( * *args,* ** *kwargs)*

> **put** ( * *args,* ** *kwargs)*

</ul>
  
</p>
</details>

### 21.15.- sanic.views module

<details><summary>Show 21.15.- sanic.views module</summary>
<p>
  
> *class* **sanic.views.CompositionView**

<ul type="square">

Bases: `object`

Simple method-function mapped view for the sanic. You can add handler functions to methods (get, post, put, patch, delete) for every HTTP method you want to support.

> For example:

<ul type="square">

view = CompositionView() view.add([‘GET’], lambda request: text(‘I am get method’)) view.add([‘POST’, ‘PUT’], lambda request: text(‘I am post/put method’))

</ul>

etc.

If someone tries to use a non-implemented method, there will be a 405 response.

> **add** *(methods, handler, stream=False)*

</ul>

> *class* **sanic.views.HTTPMethodView**

<ul type="square">

Bases: `object`

Simple class based implementation of view for the sanic. You should implement methods (get, post, put, patch, delete) for the class to every HTTP method you want to support.

For example:

```python 
class DummyView(HTTPMethodView):
    def get(self, request, *args, **kwargs):
        return text('I am get method')
    def put(self, request, *args, **kwargs):
        return text('I am put method')
```

etc.

If someone tries to use a non-implemented method, there will be a 405 response.

If you need any url params just mention them in method definition:

```python 
class DummyView(HTTPMethodView):
    def get(self, request, my_param_here, *args, **kwargs):
        return text('I am get method with %s' % my_param_here)
```

> To add the view into the routing you could use

<ul type="square">

1.- app.add_route(DummyView.as_view(), ‘/’)
2.- app.route(‘/’)(DummyView.as_view())

</ul>

To add any decorator you could set it into decorators variable

> *classmethod* **as_view** ( * *class_args,* ** *class_kwargs)*

<ul type="square">

Return view function for use with the routing system, that dispatches request to appropriate handler method.

</ul>

> **decorators** *= []*

> **dispatch_request** *(request,* * *args,* ** *kwargs)*

</ul>

> **sanic.views.stream** *(func)*
  
</p>
</details>

### 21.16.- sanic.websocket module

<details><summary>Show 21.16.- sanic.websocket module</summary>
<p>
  
> *class* **sanic.websocket.WebSocketProtocol** ( * *args, websocket_max_size=None, websocket_max_queue=None,* ** *kwargs)*

<ul type="square">

Bases: `sanic.server.HttpProtocol`

> **connection_lost** *(exc)*

> **data_received** *(data)*

> **keep_alive_timeout_callback** ()

> **request_timeout_callback**()

> **response_timeout_callback**()

> *coroutine* **websocket_handshake** *(request, subprotocols=None)*

> **write_response** *(response)*

</ul>
  
</p>
</details>

### 21.17.- sanic.worker module

### 21.18.- Module contents
<details><summary>Show 21.18.- Module contents</summary>
<p>
  
> *class* **sanic.Sanic** *(name=None, router=None, error_handler=None, load_env=True, request_class=None, strict_slashes=False, log_config=None, configure_logging=True)*

<ul type="square">

Bases: `object`

> **add_route** *(handler, uri, methods=frozenset({'GET'}), host=None, strict_slashes=None, version=None, name=None)*

<ul type="square">

A helper method to register class instance or functions as a handler to the application url routes.

**Parameters:**

- **handler** – function or class instance
- **uri** – path of the URL
- **methods** – list or tuple of methods allowed, these are overridden if using a HTTPMethodView
- **host** –
- **strict_slashes** –
- **version** –
- **name** – user defined route name for url_for

**Returns:**	function or class instance

</ul></ul>

> **add_task** *(task)*

<ul type="square">

Schedule a task to run later, after the loop has started. Different from asyncio.ensure_future in that it does not also return a future, and the actual ensure_future call is delayed until before server start.

<ul type="square">

**Parameters:	task** – future, couroutine or awaitable

</ul></ul>

> **add_websocket_route** *(handler, uri, host=None, strict_slashes=None, name=None)*

<ul type="square">

A helper method to register a function as a websocket route.

</ul>

> **blueprint** *(blueprint,* ** *options)*

<ul type="square">

Register a blueprint on the application.

<ul type="square">

**Parameters:**

- **blueprint** – Blueprint object
- **options** – option dictionary with blueprint defaults

**Returns:**	Nothing

</ul></ul>

> **converted_response_type** *(response)*

> *coroutine* **create_server** *(host=None, port=None, debug=False, ssl=None, sock=None, protocol=None, backlog=100, stop_event=None, access_log=True)*

<ul type="square">

Asynchronous version of run.

> NOTE: This does not support multiprocessing and is not the preferred

<ul type="square">

way to run a Sanic application.

</ul></ul>

> **delete** *(uri, host=None, strict_slashes=None, version=None, name=None)*

> **enable_websocket** *(enable=True)*

<ul type="square">

Enable or disable the support for websocket.

Websocket is enabled automatically if websocket routes are added to the application.

</ul>

> **exception** ( * *exceptions)*

<ul type="square">

Decorate a function to be registered as a handler for exceptions

<ul type="square">

**Parameters:	exceptions** – exceptions

**Returns:**	decorated function

</ul></ul>

> **get** *(uri, host=None, strict_slashes=None, version=None, name=None)*

> *coroutine* **handle_request** *(request, write_callback, stream_callback)*

<ul type="square">

Take a request from the HTTP Server and return a response object to be sent back The HTTP Server only expects a response object, so exception handling must be done here

<ul type="square">

**Parameters:**

- **request** – HTTP Request object
- **write_callback** – Synchronous response function to be called with the response as the only argument
- **stream_callback** – Coroutine that handles streaming a StreamingHTTPResponse if produced by the handler.

**Returns:**	Nothing

</ul></ul>

> **head** *(uri, host=None, strict_slashes=None, version=None, name=None)*

> **listener** *(event)*

<ul type="square">

Create a listener from a decorated function.

<ul type="square">

**Parameters:	event** – event to listen to

</ul></ul>

> **loop**

<ul type="square">

Synonymous with asyncio.get_event_loop().

Only supported when using the app.run method.

</ul>

> **middleware** *(middleware_or_request)*

<ul type="square">

Decorate and register middleware to be called before a request. Can either be called as @app.middleware or @app.middleware(‘request’)

</ul>

> **options** *(uri, host=None, strict_slashes=None, version=None, name=None)*

> **patch** *(uri, host=None, strict_slashes=None, stream=False, version=None, name=None)*

> **post** *(uri, host=None, strict_slashes=None, stream=False, version=None, name=None)*

> **put** *(uri, host=None, strict_slashes=None, stream=False, version=None, name=None)*

> **register_blueprint** ( * *args,* ** *kwargs)*

> **register_middleware** *(middleware, attach_to='request')*

> **remove_route** *(uri, clean_cache=True, host=None)*

> **route** *(uri, methods=frozenset({'GET'}), host=None, strict_slashes=None, stream=False, version=None, name=None)*

<ul type="square">

Decorate a function to be registered as a route

<ul type="square">

**Parameters:**

- **uri** – path of the URL
- **methods** – list or tuple of methods allowed
- **host** –
- **strict_slashes** –
- **stream** –
- **version** –
- **name** – user defined route name for url_for

**Returns:**	decorated function

</ul></ul>

> **run** *(host=None, port=None, debug=False, ssl=None, sock=None, workers=1, protocol=None, backlog=100, stop_event=None, register_sys_signals=True, access_log=True)*

<ul type="square">

Run the HTTP Server and listen until keyboard interrupt or term signal. On termination, drain connections before closing.

<ul type="square">

**Parameters:**

- **host** – Address to host on
- **port** – Port to host on
- **debug** – Enables debug output (slows server)
- **ssl** – SSLContext, or location of certificate and key for SSL encryption of worker(s)
- **sock** – Socket for the server to accept connections from
- **workers** – Number of processes received before it is respected
- **backlog** –
- **stop_event** –
- **register_sys_signals** –
- **protocol** – Subclass of asyncio protocol class

**Returns:**	Nothing

</ul></ul>

**static** *(uri, file_or_directory, pattern='/?.+', use_modified_since=True, use_content_range=False, stream_large_files=False, name='static', host=None, strict_slashes=None)*

<ul type="square">

Register a root to serve files from. The input can either be a file or a directory. See

</ul>

> **stop**()

<ul type="square">

This kills the Sanic

</ul>

> **test_client**

> *coroutine* **trigger_events** *(events, loop)*

<ul type="square">

Trigger events (functions or async) :param events: one or more sync or async functions to execute :param loop: event loop

</ul>

> **url_for** *(view_name: str,* ** *kwargs)*

<ul type="square">

Build a URL based on a view name and the values provided.

In order to build a URL, all request parameters must be supplied as keyword arguments, and each parameter must pass the test for the specified parameter type. If these conditions are not met, a *URLBuildError* will be thrown.

Keyword arguments that are not request parameters will be included in the output URL’s query string.

<ul type="square">

**Parameters:**

- **view_name** – string referencing the view name
- ** **kwargs** – keys and values that are used to build request parameters and query string arguments.

**Returns:**	the built URL

**Raises:** URLBuildError

</ul></ul>

> **websocket** *(uri, host=None, strict_slashes=None, subprotocols=None, name=None)* 

<ul type="square">

Decorate a function to be registered as a websocket route :param uri: path of the URL :param subprotocols: optional list of strings with the supported

<ul type="square">

subprotocols

</ul>

**Parameters:	host** –
**Returns:**	decorated function

</ul>

> *class* **sanic.Blueprint** *(name, url_prefix=None, host=None, version=None, strict_slashes=False)*

<ul type="square">

Bases: `object`

> **add_route** *(handler, uri, methods=frozenset({'GET'}), host=None, strict_slashes=None, version=None, name=None)*

<ul type="square">

Create a blueprint route from a function.

<ul type="square">

**Parameters:**

- **handler** – function for handling uri requests. Accepts function, or class instance with a view_class method.
- **uri** – endpoint at which the route will be accessible.
- **methods** – list of acceptable HTTP methods.
- **host** –
- **strict_slashes** –
- **version** –
- **name** – user defined route name for url_for

**Returns:**	function or class instance

</ul></ul>

> **add_websocket_route** *(handler, uri, host=None, version=None, name=None)*

<ul type="square">

Create a blueprint websocket route from a function.

<ul type="square">

**Parameters:**

- **handler** – function for handling uri requests. Accepts function, or class instance with a view_class method.
- **uri** – endpoint at which the route will be accessible.

**Returns:**	function or class instance

</ul></ul>

> **delete** *(uri, host=None, strict_slashes=None, version=None, name=None)*

> **exception** ( * *args,* ** *kwargs)*

<ul type="square">

Create a blueprint exception from a decorated function.

</ul>

> **get** *(uri, host=None, strict_slashes=None, version=None, name=None)*

> **head** *(uri, host=None, strict_slashes=None, version=None, name=None)*

> **listener** *(event)*

<ul type="square">

Create a listener from a decorated function.

<ul type="square">

**Parameters:	event** – Event to listen to.

</ul></ul>

> **middleware** ( * *args,* ** *kwargs)*

<ul type="square">

Create a blueprint middleware from a decorated function.

</ul>

> **options** *(uri, host=None, strict_slashes=None, version=None, name=None)*

> **patch** *(uri, host=None, strict_slashes=None, stream=False, version=None, name=None)*

> **post** *(uri, host=None, strict_slashes=None, stream=False, version=None, name=None)*

> **put** *(uri, host=None, strict_slashes=None, stream=False, version=None, name=None)*

> **register** *(app, options)*

<ul type="square">

Register the blueprint to the sanic app.

</ul>

> **route** *(uri, methods=frozenset({'GET'}), host=None, strict_slashes=None, stream=False, version=None, name=None)*

<ul type="square">

Create a blueprint route from a decorated function.

<ul type="square">

**Parameters:**

- **uri** – endpoint at which the route will be accessible.
- **methods** – list of acceptable HTTP methods.

</ul></ul>

> **static** *(uri, file_or_directory,* * *args,* ** *kwargs)*

<ul type="square">

Create a blueprint static route from a decorated function.

<ul type="square">

**Parameters:**

- **uri** – endpoint at which the route will be accessible.
- **file_or_directory** – Static asset.

</ul></ul>

> **websocket** *(uri, host=None, strict_slashes=None, version=None, name=None)*

<ul type="square">

Create a blueprint websocket route from a decorated function.

<ul type="square">

**Parameters:	uri** – endpoint at which the route will be accessible

</ul></ul></ul>
  
</p>
</details>
  
</ul></ul></p>
</details>

# References

- https://sanic.readthedocs.io/en/latest/
- https://github.com/channelcat/sanic/
