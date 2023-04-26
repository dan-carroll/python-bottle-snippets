# Python Bottle Snippets
Much of this information is gleaned from many online sources. I wanted to include as many code references as I could for my own use, and to share with others looking for code samples for Bottle. And I will link to appropriate documentation and tutorials (if possible).

## Bottle
[Bottle](https://bottlepy.org/docs/dev/) is a fast, simple and lightweight WSGI micro web-framework for Python.

### [“Hello World” in a bottle](https://bottlepy.org/docs/dev/tutorial.html#quickstart-hello-world)
A proper hello world:
```python
from bottle import route, run

@route('/hello')
def hello():
    return "Hello World!"

run(host='localhost', port=8080, debug=True)
```
Hello \<name\>:
```python
from bottle import route, run, template

@route('/hello/<name>')
def index(name):
    return template('<b>Hello {{name}}</b>!', name=name)

run(host='localhost', port=8080)
```
Multiple routes on a single callback function and a default keyword argument:
```python
@route('/')
@route('/hello/<name>')
def greet(name='Stranger'):
    return template('Hello {{name}}, how are you?', name=name)
```

### [Dynamic Routes](https://bottlepy.org/docs/dev/tutorial.html#dynamic-routes)
Dynamic routes with wildcards:
```python
@route('/wiki/<pagename>')            # matches /wiki/Learning_Python
def show_wiki_page(pagename):
    ...

@route('/<action>/<user>')            # matches /follow/defnull
def user_api(action, user):
    ...
```
Dynamic routes with filters:
```python
@route('/object/<id:int>')
def callback(id):
    assert isinstance(id, int)

@route('/show/<name:re:[a-z]+>')
def callback(name):
    assert name.isalpha()

@route('/static/<path:path>')
def callback(path):
    return static_file(path, ...)
```

### [HTTP Request Methods](https://bottlepy.org/docs/dev/tutorial.html#http-request-methods)
Get and Post methods, or just use Route:
```python
from bottle import get, post, request # or route

@get('/login') # or @route('/login')
def login():
    return '''
        <form action="/login" method="post">
            Username: <input name="username" type="text" />
            Password: <input name="password" type="password" />
            <input value="Login" type="submit" />
        </form>
    '''

@post('/login') # or @route('/login', method='POST')
def do_login():
    username = request.forms.get('username')
    password = request.forms.get('password')
    if check_login(username, password):
        return "<p>Your login information was correct.</p>"
    else:
        return "<p>Login failed.</p>"
```

### [Routing Static Files](https://bottlepy.org/docs/dev/tutorial.html#routing-static-files)
Route to static files by name:
```python
from bottle import static_file
@route('/static/<filename>')
def server_static(filename):
    return static_file(filename, root='/path/to/your/static/files')
```
Route to a static files directory:
```python
@route('/static/<filepath:path>')
def server_static(filepath):
    return static_file(filepath, root='/path/to/your/static/files')
```
Directly return [static files](https://bottlepy.org/docs/dev/tutorial.html#static-files) and provide mime type:
```python
from bottle import static_file
@route('/images/<filename:re:.*\.png>')
def send_image(filename):
    return static_file(filename, root='/path/to/image/files', mimetype='image/png')

@route('/static/<filename:path>')
def send_static(filename):
    return static_file(filename, root='/path/to/static/files')
```

### [Error Pages](https://bottlepy.org/docs/dev/tutorial.html#error-pages), [Abort and Redirect](https://bottlepy.org/docs/dev/tutorial.html#http-errors-and-redirects)
HTTP status code with the error() decorator:
```python
from bottle import error
@error(404)
def error404(error):
    return 'Nothing here, sorry'
```

Abort() function short cut:
```python
from bottle import route, abort
@route('/restricted')
def restricted():
    abort(401, "Sorry, access denied.")
```
And redirect:
```python
rom bottle import route, redirect
@route('/wrong/url')
def wrong():
    redirect("/right/url")
```

### [Content Generation](https://bottlepy.org/docs/dev/tutorial.html#generating-content)
Changing the Default Encoding:
```python
from bottle import response
@route('/iso')
def get_iso():
    response.charset = 'ISO-8859-15'
    return u'This will be sent with ISO-8859-15 encoding.'

@route('/latin9')
def get_latin():
    response.content_type = 'text/html; charset=latin9'
    return u'ISO-8859-15 is also known as latin9.'
```

Force a file download:
```python
@route('/download/<filename:path>')
def download(filename):
    return static_file(filename, root='/path/to/static/files', download=filename)
```
