# Flask-CORS

Flask-CORS is a Flask extension that makes it easy to enable Cross-Origin Resource Sharing (CORS) in Flask applications.

CORS allows a frontend running on one origin (for example, `http://localhost:3000`) to access a backend API on another origin (for example, `http://localhost:5000`).

---

## Install

```bash
pip install flask-cors
```

---

## Basic Usage

```python
from flask import Flask
from flask_cors import CORS

app = Flask(__name__)
CORS(app)

@app.route("/api/data")
def data():
    return {"message": "CORS enabled"}
```

This enables CORS for all routes and origins.

---

## Restrict Allowed Origins

```python
from flask import Flask
from flask_cors import CORS

app = Flask(__name__)

CORS(app, origins=["http://localhost:3000"])

@app.route("/api/data")
def data():
    return {"message": "Only localhost:3000 can access"}
```

---

## Enable CORS for Specific Routes

```python
CORS(app, resources={
    r"/api/*": {"origins": "http://localhost:3000"}
})
```

---

## Using `@cross_origin`

```python
from flask_cors import cross_origin

@app.route("/api/private")
@cross_origin(origins="http://localhost:3000")
def private():
    return {"status": "allowed"}
```

---

## Common Options

| Option | Description |
|---|---|
| `origins` | Allowed domains |
| `methods` | Allowed HTTP methods |
| `allow_headers` | Allowed request headers |
| `supports_credentials` | Allow cookies/auth |
| `max_age` | Cache preflight response |

Example:

```python
CORS(app,
     origins=["http://localhost:3000"],
     methods=["GET", "POST"],
     allow_headers=["Content-Type"],
     supports_credentials=True)
```

---

## Typical Frontend Error Without CORS

```text
Access to fetch at 'http://localhost:5000/api'
from origin 'http://localhost:3000'
has been blocked by CORS policy
```

---

## Official Documentation

- Flask-CORS Documentation:
  https://flask-cors.readthedocs.io/en/latest/

- Flask-CORS GitHub Repository:
  https://github.com/corydolphin/flask-cors
