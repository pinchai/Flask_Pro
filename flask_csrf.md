# Flask CSRFProtect with Custom Form
(https://media.geeksforgeeks.org/wp-content/uploads/20221219133429/CSRF-Diagram-(1).png)
## 1. Install dependency

```bash
pip install flask-wtf
```

---

## 2. Flask Setup

```python
from flask import Flask, render_template, request
from flask_wtf.csrf import CSRFProtect

app = Flask(__name__)

app.config["SECRET_KEY"] = "super-secret-key"

csrf = CSRFProtect(app)
```

---

## 3. Custom HTML Form (No FlaskForm)

```html
<form method="POST" action="/submit">

    <!-- CSRF token -->
    <input type="hidden" name="csrf_token" value="{{ csrf_token() }}">

    <input type="text" name="username" placeholder="Enter name">

    <button type="submit">Submit</button>
</form>
```

---

## 4. Flask Routes

```python
@app.route("/")
def index():
    return render_template("index.html")


@app.route("/submit", methods=["POST"])
def submit():
    username = request.form.get("username")
    return f"Hello {username}"
```

---

## 5. How CSRF Works

- Validates POST/PUT/DELETE requests
- Checks token from form or header
- Rejects request if token is missing or invalid

---

## 6. AJAX Example

```javascript
fetch("/submit", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "X-CSRFToken": "{{ csrf_token() }}"
  },
  body: JSON.stringify({ name: "chai" })
})
```

---

## 7. Common Error

CSRF token missing or incorrect:

- Ensure SECRET_KEY is set
- Ensure csrf_token() is in form
- Ensure POST method is used
