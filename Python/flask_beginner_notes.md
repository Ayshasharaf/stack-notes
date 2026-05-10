# Flask — Beginner Notes

> Python's simplest way to build a web server or API. Zero fluff.

---

## What is Flask?

Flask is a **Python web framework**. It lets you build websites and APIs with very little code.

> Think of it like a waiter. Someone makes a request (HTTP), Flask figures out who should handle it, runs your Python code, and sends back a response.

```
Browser / Postman
      |
      |  GET /hello
      ▼
   Flask app
      |
      |  runs your Python function
      ▼
   Returns "Hello World"
```

---

## Install & Run Your First App

```bash
pip install flask
```

```python
# app.py
from flask import Flask

app = Flask(__name__)          # create the app

@app.route("/hello")           # when someone visits /hello
def hello():
    return "Hello World!"      # send this back

if __name__ == "__main__":
    app.run(debug=True)        # start the server
```

```bash
python app.py
# Server running at http://127.0.0.1:5000
```

Visit `http://localhost:5000/hello` in your browser → you see "Hello World!"

---

## Routes — The Most Important Thing

A **route** maps a URL to a Python function.

```python
@app.route("/")          # homepage
def home():
    return "Home page"

@app.route("/about")     # /about page
def about():
    return "About page"
```

The `@app.route(...)` part is a **decorator** — it tells Flask "when this URL is hit, run this function."

---

## URL Variables — Dynamic Routes

```python
@app.route("/user/<name>")        # <name> is a variable
def greet(name):
    return f"Hello, {name}!"

# /user/Alice  →  "Hello, Alice!"
# /user/Bob    →  "Hello, Bob!"
```

With a type:

```python
@app.route("/product/<int:id>")   # int: forces it to be a number
def product(id):
    return f"Product ID: {id}"
```

---

## HTTP Methods — GET, POST, etc.

By default, routes only accept **GET**. To accept others:

```python
from flask import request

@app.route("/login", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        return "Processing login..."
    return "Show login form"
```

| Method | Use for |
|---|---|
| GET | Fetch data |
| POST | Send/create data |
| PUT | Update data |
| DELETE | Delete data |

---

## Reading Request Data

```python
from flask import request

# From URL query string:  /search?q=flask
@app.route("/search")
def search():
    query = request.args.get("q")       # gets ?q=...
    return f"Searching for: {query}"

# From JSON body (API):
@app.route("/data", methods=["POST"])
def data():
    body = request.get_json()           # parses JSON body
    name = body["name"]
    return f"Got name: {name}"

# From a form:
@app.route("/submit", methods=["POST"])
def submit():
    name = request.form.get("name")     # gets form field
    return f"Name: {name}"
```

---

## Returning Responses

```python
from flask import jsonify, make_response

# Return plain text
@app.route("/text")
def text():
    return "Hello"

# Return JSON (most common for APIs)
@app.route("/api/user")
def get_user():
    return jsonify({"id": 1, "name": "Alice"})

# Return with a status code
@app.route("/not-found")
def missing():
    return jsonify({"error": "Not found"}), 404

# Return with custom headers
@app.route("/custom")
def custom():
    response = make_response("OK")
    response.headers["X-Custom"] = "value"
    return response
```

Common status codes:

```
200  OK — everything worked
201  Created — new thing was made
400  Bad Request — something wrong with the input
401  Unauthorized — not logged in
403  Forbidden — logged in but not allowed
404  Not Found
500  Internal Server Error
```

---

## Returning HTML (Templates)

Flask uses **Jinja2** templates. Put `.html` files in a `templates/` folder.

```
your-project/
  app.py
  templates/
    index.html
```

```html
<!-- templates/index.html -->
<h1>Hello, {{ name }}!</h1>
<p>You have {{ count }} messages.</p>
```

```python
from flask import render_template

@app.route("/profile")
def profile():
    return render_template("index.html", name="Alice", count=5)
```

Flask replaces `{{ name }}` and `{{ count }}` with the values you pass.

---

## Blueprints — Organising Bigger Apps

When your app grows, don't put everything in `app.py`. Split into blueprints.

```
project/
  app.py
  routes/
    users.py
    products.py
```

```python
# routes/users.py
from flask import Blueprint

users_bp = Blueprint("users", __name__)

@users_bp.route("/users")
def get_users():
    return "All users"

@users_bp.route("/users/<int:id>")
def get_user(id):
    return f"User {id}"
```

```python
# app.py
from flask import Flask
from routes.users import users_bp

app = Flask(__name__)
app.register_blueprint(users_bp)    # plug it in
```

---

## Building a REST API — Full Example

This is the most common thing you'll actually build.

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

# Fake in-memory database
users = [
    {"id": 1, "name": "Alice"},
    {"id": 2, "name": "Bob"}
]

# GET all users
@app.route("/api/users", methods=["GET"])
def get_users():
    return jsonify(users)

# GET one user
@app.route("/api/users/<int:id>", methods=["GET"])
def get_user(id):
    user = next((u for u in users if u["id"] == id), None)
    if user is None:
        return jsonify({"error": "Not found"}), 404
    return jsonify(user)

# POST create user
@app.route("/api/users", methods=["POST"])
def create_user():
    data = request.get_json()
    new_user = {"id": len(users) + 1, "name": data["name"]}
    users.append(new_user)
    return jsonify(new_user), 201

# DELETE user
@app.route("/api/users/<int:id>", methods=["DELETE"])
def delete_user(id):
    global users
    users = [u for u in users if u["id"] != id]
    return jsonify({"message": "Deleted"}), 200

if __name__ == "__main__":
    app.run(debug=True)
```

Test it with curl:

```bash
# Get all users
curl http://localhost:5000/api/users

# Create user
curl -X POST http://localhost:5000/api/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Carol"}'

# Delete user
curl -X DELETE http://localhost:5000/api/users/1
```

---

## Error Handling

```python
from flask import jsonify

# Handle 404 globally
@app.errorhandler(404)
def not_found(e):
    return jsonify({"error": "Route not found"}), 404

# Handle 500 globally
@app.errorhandler(500)
def server_error(e):
    return jsonify({"error": "Internal server error"}), 500
```

---

## app.config — Settings

```python
app.config["DEBUG"] = True
app.config["SECRET_KEY"] = "your-secret-key"     # needed for sessions
app.config["DATABASE_URL"] = "sqlite:///db.sqlite3"
```

Or load from a file / environment:

```python
import os
app.config["SECRET_KEY"] = os.environ.get("SECRET_KEY", "fallback-key")
```

---

## Project Structure (Standard)

```
my-flask-app/
  app.py               ← entry point
  config.py            ← settings
  requirements.txt     ← pip dependencies
  routes/
    __init__.py
    users.py
    products.py
  models/
    user.py            ← database models
  templates/
    index.html
  static/
    style.css
    script.js
```

---

## Quick Reference

```
@app.route("/path")              Define a route
@app.route("/path", methods=[])  Specify HTTP methods
request.args.get("key")          Read query param  ?key=value
request.get_json()               Read JSON body
request.form.get("key")          Read form field
jsonify({})                      Return JSON response
render_template("file.html")     Return HTML page
return "text", 404               Return with status code
Blueprint(name, __name__)        Group routes into a module
app.run(debug=True)              Start dev server
```

---

## Useful Libraries to Know Next

| Library | What it adds |
|---|---|
| `flask-sqlalchemy` | Database ORM (talk to MySQL, PostgreSQL, SQLite) |
| `flask-jwt-extended` | JWT authentication / login tokens |
| `flask-cors` | Allow other domains to call your API |
| `flask-marshmallow` | Validate and serialize request/response data |
| `python-dotenv` | Load config from a `.env` file |

Install example:

```bash
pip install flask flask-sqlalchemy flask-cors python-dotenv
```
