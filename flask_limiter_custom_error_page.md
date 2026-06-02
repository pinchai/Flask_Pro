# Flask-Limiter with Custom Error Page

## Install Flask-Limiter

```bash
pip install Flask-Limiter
```

---

## Project Structure

```text
project/
│
├── app.py
└── templates/
    └── 429.html
```

---

# app.py

```python
from flask import Flask, render_template, request
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

app = Flask(__name__)

# Flask-Limiter setup
limiter = Limiter(
    key_func=get_remote_address,
    app=app,
    default_limits=["5 per minute"]
)


@app.route('/')
@limiter.limit("3 per minute")
def home():
    return "Welcome to KhmerFun"


# Custom 429 Error Page
@app.errorhandler(429)
def ratelimit_handler(e):
    return render_template(
        "429.html",
        ip=request.remote_addr,
        error=str(e)
    ), 429


if __name__ == '__main__':
    app.run(debug=True)
```

---

# templates/429.html

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Too Many Requests</title>

    <style>
        body{
            margin:0;
            padding:0;
            height:100vh;
            display:flex;
            justify-content:center;
            align-items:center;
            background:#f5f5f5;
            font-family:Arial, sans-serif;
        }

        .card{
            width:400px;
            background:white;
            padding:30px;
            border-radius:10px;
            box-shadow:0 0 10px rgba(0,0,0,0.1);
            text-align:center;
        }

        h1{
            color:#dc3545;
            margin-bottom:10px;
        }

        p{
            color:#555;
        }

        .ip{
            margin-top:20px;
            padding:10px;
            background:#eee;
            border-radius:5px;
            font-weight:bold;
        }

        a{
            display:inline-block;
            margin-top:20px;
            text-decoration:none;
            background:#007bff;
            color:white;
            padding:10px 20px;
            border-radius:5px;
        }
    </style>
</head>
<body>

<div class="card">
    <h1>429</h1>

    <p>Too many requests.</p>

    <p>Please wait a moment and try again.</p>

    <div class="ip">
        Your IP: {{ ip }}
    </div>

    <p>{{ error }}</p>

    <a href="/">Back Home</a>
</div>

</body>
</html>
```

---

# How It Works

When a user exceeds the limit:

```python
@limiter.limit("3 per minute")
```

Flask-Limiter automatically raises:

```python
429 Too Many Requests
```

Then Flask uses:

```python
@app.errorhandler(429)
```

to show your custom HTML page instead of the default error message.

---

# Example Limits

```python
@limiter.limit("10 per minute")
@limiter.limit("100 per day")
```

Or:

```python
default_limits=["200 per day", "50 per hour"]
```

---

# Skip Limit for Admin

```python
@limiter.exempt
@app.route('/admin')
def admin():
    return "Admin Page"
```

---

# JSON API Error Example

```python
@app.errorhandler(429)
def ratelimit_handler(e):
    return {
        "status": 429,
        "message": "Too many requests"
    }, 429
```
