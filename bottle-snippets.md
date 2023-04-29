# Python Bottle Snippets
Much of this information is gleaned from many online sources. I wanted to include as many code references as I could for my own use, and to share with others looking for code samples for Bottle. And I will link to appropriate documentation and tutorials (if possible).

## Bottle
[Bottle](https://bottlepy.org/docs/dev/) is a fast, simple and lightweight WSGI micro web-framework for Python.

**<ins>Table of Contents</ins>**
- [Hello World](#hello-world-in-a-bottle)
- [Simple Examples(#simple-examples)
- [Dynamic Routes](#dynamic-routes)
- [HTTP Request Methods](#http-request-methods)
- [Routing Static Files](#routing-static-files)
- [Error Pages](#error-pages)
- [Content Generation](#content-generation)
- [Cookies](#cookies)
- [Request Data](#request-data)
- [Formsdict](#formsdict)
- [HTTP Headers](#http-headers)
- [Query Variables](#query-variables)
- [HTML Form Handling](#html-form-handling)
- [File Uploads](#file-uploads)
- [WSGI Environment](#wsgi-environment)
- [Templates](#templates)
- [Plugins](#plugins)
- [Plugins and Sub-Applications](#plugins-and-sub-applications)
- [List of Available Plugins](#list-of-available-plugins)
- [Example Apps](#example-apps)
  - [python bottle and ajax](#python-bottle-and-ajax)

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

Simple message display (instead of hello):
```python
from bottle import route, run

@route('/message')
def hello():
    return "Today is a beautiful day"
```

### Simple Examples

#### Passing JSON Data
Bottle automatically transforms Python dictionaries into JSON:
```python
from bottle import route, run

@route('/cars')
def getcars():

    cars = [ {'name': 'Audi', 'price': 52642},
        {'name': 'Mercedes', 'price': 57127},
        {'name': 'Skoda', 'price': 9000},
        {'name': 'Volvo', 'price': 29000},
        {'name': 'Bentley', 'price': 350000},
        {'name': 'Citroen', 'price': 21000},
        {'name': 'Hummer', 'price': 41400},
        {'name': 'Volkswagen', 'price': 21600} ]

    return dict(data=cars)
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

### [Cookies](https://bottlepy.org/docs/dev/tutorial.html#cookies)
Set and Get cookies:
```python
@route('/hello')
def hello_again():
    if request.get_cookie("visited"):
        return "Welcome back! Nice to see you again"
    else:
        response.set_cookie("visited", "yes")
        return "Hello there! Nice to meet you"
```

Cryptographically sign your cookies:
```python
@route('/login')
def do_login():
    username = request.forms.get('username')
    password = request.forms.get('password')
    if check_login(username, password):
        response.set_cookie("account", username, secret='some-secret-key')
        return template("<p>Welcome {{name}}! You are now logged in.</p>", name=username)
    else:
        return "<p>Login failed.</p>"

@route('/restricted')
def restricted_area():
    username = request.get_cookie("account", secret='some-secret-key')
    if username:
        return template("Hello {{name}}. Welcome back.", name=username)
    else:
        return "You are not logged in. Access denied."
```

A simple cookie-based view counter:
```python
from bottle import route, request, response
@route('/counter')
def counter():
    count = int( request.cookies.get('counter', '0') )
    count += 1
    response.set_cookie('counter', str(count))
    return 'You visited this page %d times' % count
```

### [Request Data](https://bottlepy.org/docs/dev/tutorial.html#request-data)
Cookies, HTTP header, HTML \<form\> fields and other request data is available through the global request object.
```python
from bottle import request, route, template

@route('/hello')
def hello():
    name = request.cookies.username or 'Guest'
    return template('Hello {{name}}', name=name)
```

#### [Formsdict](https://bottlepy.org/docs/dev/tutorial.html#introducing-formsdict)
Bottle uses a special type of dictionary to store form data and cookies.

Attribute access:
```python
name = request.cookies.name

# is a shortcut for:

name = request.cookies.getunicode('name') # encoding='utf-8' (default)

# which basically does this:

try:
    name = request.cookies.get('name', '').decode('utf-8')
except UnicodeError:
    name = u''
```
Multiple values per key:
```python
for choice in request.forms.getall('multiple_choice'):
    do_something(choice)
```
WTForms support:
```python
>>> request.query['city']
'GÃ¶ttingen' # An utf8 string provisionally decoded as ISO-8859-1 by the server
>>> request.query.city
'Göttingen'  # The same string correctly re-encoded as utf8 by bottle
```

### [HTTP Headers](https://bottlepy.org/docs/dev/tutorial.html#http-headers)
WSGIHeaderDict is basically a dictionary with case-insensitive keys:
```python
from bottle import route, request
@route('/is_ajax')
def is_ajax():
    if request.headers.get('X-Requested-With') == 'XMLHttpRequest':
        return 'This is an AJAX request'
    else:
        return 'This is a normal request'
```

### [Query Variables](https://bottlepy.org/docs/dev/tutorial.html#query-variables)
The query string (as in /forum?id=1&page=5) is commonly used to transmit a small number of key/value pairs to the server.
```python
from bottle import route, request, response, template
@route('/forum')
def display_forum():
    forum_id = request.query.id
    page = request.query.page or '1'
    return template('Forum ID: {{id}} (page {{page}})', id=forum_id, page=page)
```

### [HTML \<form\> Handling](https://bottlepy.org/docs/dev/tutorial.html#html-form-handling)
A typical \<form\> looks something like this:
```html
<form action="/login" method="post">
    Username: <input name="username" type="text" />
    Password: <input name="password" type="password" />
    <input value="Login" type="submit" />
</form>
```
Server side code may look like this:
```python
from bottle import route, request

@route('/login')
def login():
    return '''
        <form action="/login" method="post">
            Username: <input name="username" type="text" />
            Password: <input name="password" type="password" />
            <input value="Login" type="submit" />
        </form>
    '''

@route('/login', method='POST')
def do_login():
    username = request.forms.get('username')
    password = request.forms.get('password')
    if check_login(username, password):
        return "<p>Your login information was correct.</p>"
    else:
        return "<p>Login failed.</p>"
```

### [File Uploads](https://bottlepy.org/docs/dev/tutorial.html#file-uploads)
Here is an example:
```html
<form action="/upload" method="post" enctype="multipart/form-data">
  Category:      <input type="text" name="category" />
  Select a file: <input type="file" name="upload" />
  <input type="submit" value="Start upload" />
</form>
```
Let us assume you just want to save the file to disk:
```python
@route('/upload', method='POST')
def do_upload():
    category   = request.forms.get('category')
    upload     = request.files.get('upload')
    name, ext = os.path.splitext(upload.filename)
    if ext not in ('.png','.jpg','.jpeg'):
        return 'File extension not allowed.'

    save_path = get_save_path_for_category(category)
    upload.save(save_path) # appends upload.filename automatically
    return 'OK'
```

### [WSGI Environment](https://bottlepy.org/docs/dev/tutorial.html#wsgi-environment)
Access WSGI environ variables directly:
```python
@route('/my_ip')
def show_ip():
    ip = request.environ.get('REMOTE_ADDR')
    # or ip = request.get('REMOTE_ADDR')
    # or ip = request['REMOTE_ADDR']
    return template("Your IP is: {{ip}}", ip=ip)
```

## [Templates](https://bottlepy.org/docs/dev/tutorial.html#templates)
Bottle comes with a fast and powerful built-in template engine called [SimpleTemplate Engine](https://bottlepy.org/docs/dev/stpl.html). To render a template you can use the [template() function](https://bottlepy.org/docs/dev/api.html#bottle.template) or the [view() decorator](https://bottlepy.org/docs/dev/api.html#bottle.view).

Simple example of how to render a template:
```python
@route('/hello')
@route('/hello/<name>')
def hello(name='World'):
    return template('hello_template', name=name)
```

view() decorator example:
```python
@route('/hello')
@route('/hello/<name>')
@view('hello_template')
def hello(name='World'):
    return dict(name=name)
```

Example template syntax:
```html
%if name == 'World':
    <h1>Hello {{name}}!</h1>
    <p>This is a test.</p>
%else:
    <h1>Hello {{name.title()}}!</h1>
    <p>How are you?</p>
%end
```

## [Plugins](https://bottlepy.org/docs/dev/tutorial.html#plugins)
The SQLitePlugin plugin for example:
```python
from bottle import route, install, template
from bottle_sqlite import SQLitePlugin

install(SQLitePlugin(dbfile='/tmp/test.db'))

@route('/show/<post_id:int>')
def show(db, post_id):
    c = db.execute('SELECT title, content FROM posts WHERE id = ?', (post_id,))
    row = c.fetchone()
    return template('show_post', title=row['title'], text=row['content'])

@route('/contact')
def contact_page():
    ''' This callback does not need a db connection. Because the 'db'
        keyword argument is missing, the sqlite plugin ignores this callback
        completely. '''
    return template('contact')
```

Plugin installed application-wide:
```python
from bottle_sqlite import SQLitePlugin
install(SQLitePlugin(dbfile='/tmp/test.db'))
```

Uninstall Plugins:
```python
sqlite_plugin = SQLitePlugin(dbfile='/tmp/test.db')
install(sqlite_plugin)

uninstall(sqlite_plugin) # uninstall a specific plugin
uninstall(SQLitePlugin)  # uninstall all plugins of that type
uninstall('sqlite')      # uninstall all plugins with that name
uninstall(True)          # uninstall all plugins at once
```

Route-Specific Installation:
```python
sqlite_plugin = SQLitePlugin(dbfile='/tmp/test.db')

@route('/create', apply=[sqlite_plugin])
def create(db):
    db.execute('INSERT INTO ...')
```

Blacklisting Plugins:
```python
sqlite_plugin = SQLitePlugin(dbfile='/tmp/test1.db')
install(sqlite_plugin)

dbfile1 = '/tmp/test1.db'
dbfile2 = '/tmp/test2.db'

@route('/open/<db>', skip=[sqlite_plugin])
def open_db(db):
    # The 'db' keyword argument is not touched by the plugin this time.

    # The plugin handle can be used for runtime configuration, too.
    if db == 'test1':
        sqlite_plugin.dbfile = dbfile1
    elif db == 'test2':
        sqlite_plugin.dbfile = dbfile2
    else:
        abort(404, "No such database.")

    return "Database File switched to: " + sqlite_plugin.dbfile
```

### [Plugins and Sub-Applications](https://bottlepy.org/docs/dev/tutorial.html#plugins-and-sub-applications)
Should not affect sub-applications mounted with [Bottle.mount()](https://bottlepy.org/docs/dev/api.html#bottle.Bottle.mount):
```python
root = Bottle()
root.mount('/blog', apps.blog)

@root.route('/contact', template='contact')
def contact():
    return {'email': 'contact@example.com'}

root.install(plugins.WTForms())
```

### [List of Available Plugins](https://bottlepy.org/docs/dev/plugins/index.html#list-of-available-plugins)
This is a list of third-party plugins that add extend Bottles core functionality or integrate other libraries with the Bottle framework.

Have a look at Plugins for general questions about plugins (installation, usage). If you plan to develop a new plugin, the Plugin Development Guide may help you.

#### [Bottle-Beaker](https://pypi.org/project/bottle-beaker/)
    Beaker to session and caching library with WSGI Middleware
#### [Bottle-Cork](https://pypi.org/project/bottle-cork/)
##### [Cork - read the docs](https://cork.readthedocs.io/en/latest/)
    Cork provides a simple set of methods to implement Authentication and Authorization in web applications based on Bottle.
#### [Bottle-Cors-plugin](https://pypi.org/project/bottle-cors-plugin/)
    Cors-plugin is the easiest way to implement cors on your bottle web application
#### [Bottle-Extras](https://pypi.org/project/bottle-extras/)
    Meta package to install the bottle plugin collection.
#### [Bottle-Flash](https://pypi.org/project/bottle-flash/)
    flash plugin for bottle
#### [Bottle-Hotqueue](https://pypi.org/project/bottle-hotqueue/)
    FIFO Queue for Bottle built upon redis
#### [Macaron](https://pypi.org/project/macaron/)
##### [Macaron documentation](https://nobrin.github.io/macaron/)
    Macaron is an object-relational mapper (ORM) for SQLite.
#### [Bottle-Memcache](https://pypi.org/project/bottle-memcache/)
    Memcache integration for Bottle.
#### [Bottle-Mongo](https://pypi.org/project/bottle-mongo/)
    MongoDB integration for Bottle
#### [Bottle-OAuthlib](https://pypi.org/project/bottle-oauthlib/)
    Adapter for oauthlib - create your own OAuth2.0 implementation
#### [Bottle-Redis](https://pypi.org/project/bottle-redis/)
    Redis integration for Bottle.
#### [Bottle-Renderer](https://pypi.org/project/bottle-renderer/)
    Renderer plugin for bottle
#### [Bottle-Servefiles](https://pypi.org/project/bottle-servefiles/)
    A reusable app that serves static files for bottle apps
#### [Bottle-Sqlalchemy](https://pypi.org/project/bottle-sqlalchemy/)
    SQLAlchemy integration for Bottle.
#### [Bottle-Sqlite](https://pypi.org/project/bottle-sqlite/)
    SQLite3 database integration for Bottle.
#### [Bottle-Web2pydal](https://pypi.org/project/bottle-web2pydal/)
    Web2py Dal integration for Bottle.
#### [Bottle-Werkzeug](https://pypi.org/project/bottle-werkzeug/)
    Integrates the werkzeug library (alternative request and response objects, advanced debugging middleware and more).
#### [bottle-smart-filters](https://github.com/agile4you/bottle-smart-filters/)
    Bottle Querystring smart guessing.
#### [bottle-jwt](https://github.com/agile4you/bottle-jwt/)
    JSON Web Token authentication plugin for bottle.py
#### [Bottle-jwt](https://github.com/agalera/bottlejwt)
    JWT integration for bottle
#### [canister](https://github.com/dagnelies/canister)
    a bottle wrapper to provide logging, sessions and authentication
#### [bottle-cerberus](https://github.com/agalera/bottle-cerberus)
    Cerberus integration for bottle
#### [Bottle-errorsrest](https://github.com/agalera/bottle-errorsrest)
    All errors generated from bottle are returned in json
#### [Bottle-tools](https://github.com/theSage21/bottle-tools)
    Decorators that auto-supply function arguments using POST/query string data.

Plugins listed here are not part of Bottle or the Bottle project, but developed and maintained by third parties.

## Example Apps

### [python bottle and ajax](https://enricorossi.org/blog/2012/python_bottle_and_ajax/)
Renders a page template which sends a js async request (ajax), gets the result in a json form and displays it:
```python
from bottle import route, run, template, get, debug

debug(True)

# this will be the dictionary returned by the ajax call.
# Bottle converts this into a json compatibile string.

items = {1: 'first item', 2: 'second item'}

# a simple json test main page
@route('/')
def jsontest():
    return template('json')

@route('/getallitems.json')
def shop_aj_getallitems():
    return (items)
```
json.tpl template file:
```html
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />

<script type="text/javascript">
var xmlhttp;

// Are we using a modern browser or ...
if (window.XMLHttpRequest) {
  // code for IE7+, Firefox, Chrome, Opera, Safari
  xmlhttp=new XMLHttpRequest();
} else {
  // code for IE6, IE5
  xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
}

// This will render the two output which substitute the
// elements id="raw" and id="forin"
function GetItems()
{
  if (xmlhttp.readyState==4 && xmlhttp.status==200) {
    // var jsonobj = eval ("(" + xmlhttp.responseText + ")"); 
    var jsonobj = JSON.parse(xmlhttp.responseText); 

    var output = xmlhttp.responseText;
    document.getElementById("raw").innerHTML = output;

    output = "";

    for (i in jsonobj) {
      output += '<p>';
      output += i + " : " + jsonobj[i];
      output += '</p>';
    }

    document.getElementById("forin").innerHTML = output;

  } else {
    alert("data not available");
  }
}

// xmlhttp.onreadystatechange = GetArticles;
// the GetItems function will be triggered once the ajax
// request is terminated.
xmlhttp.onload = GetItems;

// send the request in an async way
xmlhttp.open("GET", "/getallitems.json", true);
xmlhttp.send();
</script>
</head>

<body>
  <p>The raw result from the ajax json request is:</p>
  <div id="raw"></div>
  <br />
  <p>The for cycle produces :</p>
  <div id="forin"></div>
</body>
</html>
```

