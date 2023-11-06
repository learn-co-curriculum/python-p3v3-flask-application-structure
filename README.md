# Building Flask Applications : Code-Along

## Learning Goals

- Describe the components of a web application framework.
- Build and run a Flask application on your computer.
- Manipulate and test the structure of a request object.

---

## Key Vocab

- **Routing**: the association of a URL and the code that should execute when a
  request comes in for that URL.
- **View**: a function that maps to a URL.
- **Endpoint**: an identifier to associate with a routing rule, usually having
  the name of the view function.

---

## Introduction

It's finally time to create our first Flask application. We'll start this lesson
with your standard "text output in the browser" application, but we'll also
extend a bit further to explore **routing**, creating Flask development servers,
and debugging in the browser.

---

## Setup

> This lesson is a code-along, so please fork and clone the repo.

`Pipfile` has a new dependency that we'll use in this lesson: `flask`.

Run `pipenv install && pipenv shell` to generate and enter your virtual
environment.

```console
$ pipenv install && pipenv shell
```

The `server` folder contains a file named `app.py`. We will edit this file to
create our first Flask web server.

```text
.
├── CONTRIBUTING.md
├── LICENSE.md
├── Pipfile
├── README.md
├── pytest.ini
└── server
    ├── app.py
    └── testing
        └── codegrade_test.py
```

## Initializing a Flask Application

Flask is a web framework, but it is first and foremost a _Python_ framework. To
start any Flask application, you need to create an instance of the `Flask`
class. Your web server- whether you run it from Werkzeug, Flask, or another
library entirely- interacts with this `Flask` instance using WSGI.

Open `server/app.py` and enter the following code:

```py
#!/usr/bin/env python3

from flask import Flask

app = Flask(__name__)
```

The `Flask` class constructor only requires the name of the primary module or
package to be interpreted as the application. Flask uses this to figure out
where the application is and where its important files will be. As some of these
will not be `.py` files and might not have any `.py` files in their directory,
Flask needs to set up an application structure that allows it to see everything.

For the purposes of our curriculum, this argument will always be `__name__`,
which refers to the name of the current module.

<details>
  <summary>
    <em>What is the value of <code>__name__</code> when we run
        <code>python server/app.py</code>?</em>
  </summary>

  <h3><code>'__main__'</code></h3>
  <p><code>__name__</code> is equal to the name of the module in question
     <em>unless</em> it is the module being run from the command line. In this
     case, it is set to <code>'__main__'</code>. This is very helpful for
     writing scripts!</p>
</details>
<br/>

---

## Routing Views

### Routing

When clients send requests to our application's server, they are forwarded to
our Flask application instance. This instance will receive requests for many
different resources, located at different Uniform Resource Locations (URLs). To
map a URL to a Python function, we need to define a **route**.

**Routing** is the association of a URL and the function that should execute
when a request comes in for that URL. Routing isn't just a Python concept-
JavaScript, Java, Ruby, and even newer languages like Rust and Go use routes to
direct requests to the appropriate backend code.

The easiest way to define routes with Flask is through use of the `@app.route`
decorator. Edit `server/app.py` to add an initial route for the base URL `/` as
shown:

```py
#!/usr/bin/env python3

from flask import Flask

app = Flask(__name__)


@app.route('/')
def index():
    return '<h1>Welcome to my page!</h1>'
```

Remember that **decorators** are functions that take functions as arguments and
return them decorated with new features. `@app.route` registers the `index()`
function with the Flask application instance `app`. The `@app.route()` decorator
is an instance method that modifies `app`, creating a rule that requests for the
base URL (`/`) should execute the `index()` function, resulting in a page with a
heading "Welcome to my page!"

An **endpoint** is the name to associate with the routing rule and view
function, and usually has the same name as the view function. So in the example
above, we are defining the `index` endpoint.

## The Flask Development Web Server

In the last lesson, we ran our development server through Werkzeug. We could do
that here, but we would have to reconfigure it to work specifically with our
Flask application. For now, there's an easier way: `flask run`.

`flask run` is a console command that looks for the name of the Python module
with our Flask application instance.

First, make sure you are in the `server` folder.

```console
$ cd server
```

Set the `FLASK_APP` and `FLASK_RUN_PORT` environment variables, then launch the
server using `flask run`:

```console
$ export FLASK_APP=app.py
$ export FLASK_RUN_PORT=5555
$ flask run
```

- By default, `flask run` will look for a file named `app.py`. However, you may
  choose to name your file something other than `app.py`, in which case you need
  to set the `FLASK_APP` environment variable to refer to that file. We'll set
  `FLASK_APP` to demonstrate how to do so, though this step is unnecessary as
  our file is named `app.py`.
- Flask's default port is 5000, which conflicts with AirPlay on MacOS. We need
  to set the port to 5555 to avoid this conflict.

After executing `flask run`, you should see console output similar to the
following:

```console

# => * Serving Flask app 'app.py'
# => * Debug mode: off
# => WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
# => * Running on http://127.0.0.1:5555
# =>Press CTRL+C to quit
```

Navigate to `http://127.0.0.1:5555` and confirm you see the welcome heading:

![Webpage that says "Welcome to my page!"](https://curriculum-content.s3.amazonaws.com/7391/python-p3v3-flask/welcome.png)

> **NOTE: "localhost" is the plain-English version of 127.0.0.1 on most
> machines. You will often see the two used interchangeably. Try navigating to
> `http://localhost:5555/` and you should see the same welcome page.**

### Views

We call functions that map to URLs **views**. This is easy to remember-
everything you _view_ in your application is generated by a **view** (though
there are certainly many invisible things going on in views as well).

A view returns the response that the client that made the request. Our `index()`
view returns a simple string of HTML code, but we will see that views can also
contain forms, code to ensure cybersecurity, and much more.

<details>
  <summary>
    <em>Which line of code tells Flask to show the returned data from
        <code>index()</code> in the web browser?</em>
  </summary>

  <h3>The <code>app.route</code> decorator registers the <code>index()</code>  view to
      the application instance.</h3>
  <p>"Registration" in Flask means that a view has been connected to an
     application instance's routes.</p>
  <p>When the instance receives a URL pointing
     to that route, the view function is called and the return value is added
     to the response by the instance.</p>
</details>
<br/>

### Variable Routes

Navigate to your favorite social media site and take a look at the URL. The base
will represent the index or homepage for the application. Navigate to a user
profile and take another look at the URL:

![Screenshot of NASA Twitter profile with URL twitter.com/NASA](https://curriculum-content.s3.amazonaws.com/python/twitter_nasa_screenshot.png)

"twitter.com" is clearly a fixed portion of the URL- it's everywhere! There are
other pieces, though, that it wouldn't make sense to hard-code into our
application. "NASA", for instance, is the username for one out of millions of
users on the site. Managing views for that many users would be impossible!

Lucky for us, Flask allows us to parameterize different parts of our routes.
When we interpolate these into strings or use them to retrieve records from a
database, we can create flexible, dynamic applications.

For example, we can define a route that takes a parameter to capture the user's
name.

```py
@app.route('/<username>')
def user(username):
    return f'<h1>Profile for {username}</h1>'
```

Anything included in the route passed to the `app.route` decorator with angle
brackets `<>` surrounding it will be passed to the decorated function as a
parameter. We can make sure that the username is a valid `string`, `int`,
`float`, or `path` (string with slashes) by specifying this in the route.

Update `app.py` to define two routes, one for the welcome page and one for a
page that mentions the user by name:

```py
#!/usr/bin/env python3

from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return '<h1>Welcome to my page!</h1>'

@app.route('/<string:username>')
def user(username):
    return f'<h1>Profile for {username}</h1>'
```

NOTE: You will have to stop (`ctrl-c`) and restart (`flask run`) the server to
use the new route.

Try testing the new route by appending a name to the url such as
`http://127.0.0.1:5555/happy-coder`. Now you should see something like this:

## ![Webpage that says "Profile for happy-coder"](https://curriculum-content.s3.amazonaws.com/7391/python-p3v3-flask/user-parameter-route.png)

<details>
  <summary>
    <em>Flask has to parse "string" and "username" from the route. How can you
        use Python to remove brackets and get parameters out of a string?</em>
  </summary>

  <h3><code>str</code> methods or the <code>re</code> module.</h3>

  <p>With <code>str</code> methods:</p>
  <p><code>url = '/&lt;string:username&gt;'</code></p>
  <p><code>url = url.replace('/<', '')</code></p>
  <p><code>url = url.replace('>', '')</code></p>
  <p><code>type, parameter = url.split(':')</code></p>
  <br/>
  <p>With <code>re</code>:</p>
  <p><code>exp = re.compile('[A-z]+')</code></p>
  <p><code>type, parameter = exp.findall(url)</code></p>

</details>
<br/>

### Running the server as a script

We can also run a development server by treating our application module as a
script with the `app.run()` method. This let's us avoid having to set the
`FLASK_APP` and `FLASK_RUN_PORT` environment variables:

```py
# append to server/app.py

if __name__ == '__main__':
    app.run(port=5555, debug=True)

```

Run the script with `python app.py` and you should see the same server as
before:

```console
$ python app.py

# => * Serving Flask app 'app'
# => * Debug mode: off
# => WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
# => * Running on http://127.0.0.1:5555
# =>Press CTRL+C to quit
```

`debug=True` provides us many benefits for development, but the nicest of these
is that the server will _automatically restart_ whenever we change `app.py`!

There are pros and cons to each approach. Configuring the `FLASK_APP` and
`FLASK_RUN_PORT` environment variables allows us to use tools like the Flask
shell, but it's easy to forget about environment variables since they don't show
up in your project directory.

Running from a script isn't quite as Flasky, but it keeps all of our
configuration in sight. We also still have access to these Flask tools as
written, because `flask run` and `flask shell` look for `app.py` by default!

---

## Conclusion

You've just built your first Flask web application! It's very simple, but you'll
use the Flask class and its decorator methods many times throughout Phase 4.
Next, we'll get some more practice with routing and views.

---

## Solution Code

```py
#!/usr/bin/env python3

from flask import Flask

app = Flask(__name__)


@app.route('/')
def index():
    return '<h1>Welcome to my page!</h1>'

@app.route('/<string:username>')
def user(username):
    return f'<h1>Profile for {username}</h1>'

if __name__ == '__main__':
    app.run(port=5555, debug=True)
```

---

## Resources

- [A Minimal Application - Flask](https://flask.palletsprojects.com/en/2.3.x/quickstart/#a-minimal-application)
- [Routing - Flask](https://flask.palletsprojects.com/en/2.3.x/quickstart/#routing)
- [Chapter 2. Basic Application Structure - Flask Web Development, 2nd Edition](https://learning.oreilly.com/library/view/flask-web-development/9781491991725/ch02.html#idm140583868985008)
