# Flask-Limiter

## Overview

**Flask-Limiter** is a Flask extension used to limit the number of requests a client can make within a specific time period.

It helps protect applications from:

- Brute-force attacks
- DDoS attacks
- ......

---

## Installation

```bash
pip install Flask-Limiter
```

---

## Basic Example

```python
from flask import Flask
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

app = Flask(__name__)

limiter = Limiter(
    key_func=get_remote_address,
    app=app,
    default_limits=["100 per day", "10 per minute"]
)

@app.route("/")
def home():
    return "Hello Flask-Limiter"

if __name__ == "__main__":
    app.run(debug=True)
```

### Explanation

- `get_remote_address` gets the client IP address.
- `100 per day` allows 100 requests daily.
- `10 per minute` allows 10 requests every minute.

---

## Route-Specific Limit

```python
@app.route("/login")
@limiter.limit("5 per minute")
def login():
    return "Login Page"
```

---

## Multiple Limits

```python
@app.route("/api")
@limiter.limit("10 per minute")
@limiter.limit("100 per day")
def api():
    return "API Response"
```

---

## Limit Using User ID

```python
from flask import session

def get_user():
    return session.get("user_id")

limiter = Limiter(
    key_func=get_user,
    app=app
)
```

---

## Exempt Route

```python
@app.route("/public")
@limiter.exempt
def public():
    return "No rate limit"
```

---

## Custom Error Response

```python
from flask import jsonify

@app.errorhandler(429)
def ratelimit_handler(e):
    return jsonify({
        "error": "Too many requests"
    }), 429
```

---

## Redis Storage Backend

```python
limiter = Limiter(
    key_func=get_remote_address,
    app=app,
    storage_uri="redis://localhost:6379"
)
```

Install Redis support:

```bash
pip install Flask-Limiter[redis]
```

---

## Common Rate Limit Formats

| Limit           | Meaning                  |
| --------------- | ------------------------ |
| `5 per second`  | 5 requests every second  |
| `10 per minute` | 10 requests every minute |
| `100 per hour`  | 100 requests every hour  |
| `1000 per day`  | 1000 requests every day  |
| `1 per day`     | One request per day      |

---

## Login Protection Example

```python
from flask import Flask, request
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

app = Flask(__name__)

limiter = Limiter(
    key_func=get_remote_address,
    app=app
)

@app.route("/login", methods=["POST"])
@limiter.limit("5 per minute")
def login():
    username = request.form.get("username")
    password = request.form.get("password")

    return "Login Successful"

if __name__ == "__main__":
    app.run(debug=True)
```
