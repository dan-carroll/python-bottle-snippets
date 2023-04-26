# Python Bottle Snippets
Much of this information is gleaned from many online sources. I wanted to include as many code references as I could for my own use, and to share with others looking for code samples for Bottle.

## Bottle
[Bottle](https://bottlepy.org/docs/dev/) is a fast, simple and lightweight WSGI micro web-framework for Python.

### “Hello World” in a bottle
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
Dynamic routes with wildcards:
```python
@route('/wiki/<pagename>')            # matches /wiki/Learning_Python
def show_wiki_page(pagename):
    ...

@route('/<action>/<user>')            # matches /follow/defnull
def user_api(action, user):
    ...
```
