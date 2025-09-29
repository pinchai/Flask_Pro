```python
from datetime import timedelta
from functools import wraps
from flask import (
    Flask, render_template, request, redirect,
    url_for, session, flash
)
from werkzeug.security import check_password_hash, generate_password_hash

app = Flask(__name__)
app.config["SECRET_KEY"] = "change-this"
app.config["PERMANENT_SESSION_LIFETIME"] = timedelta(minutes=30)  # session TTL

# --- demo user (replace with DB later) ---
DEMO_USER = {
    "id": 1,
    "username": "admin",
    "password_hash": generate_password_hash("admin123")  # password = admin123
}


def login_required(view):
    @wraps(view)
    def wrapped(*args, **kwargs):
        if not session.get("user_id"):
            flash("Please log in first.", "warning")
            return redirect(url_for("login", next=request.path))
        return view(*args, **kwargs)

    return wrapped


@app.get("/")
def home():
    # redirect to dashboard if logged in
    if session.get("user_id"):
        return redirect(url_for("dashboard"))
    return redirect(url_for("login"))


@app.route("/login", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        username = request.form.get("username", "").strip()
        password = request.form.get("password", "")

        # lookup demo user
        if username == DEMO_USER["username"] and check_password_hash(DEMO_USER["password_hash"], password):
            session.clear()
            session.permanent = bool(request.form.get("remember"))  # remember me
            session["user_id"] = DEMO_USER["id"]
            session["username"] = DEMO_USER["username"]
            flash("Welcome back!", "success")
            return redirect(request.args.get("next") or url_for("dashboard"))
        flash("Invalid username or password.", "danger")

    return render_template("login.html")


@app.get("/logout")
def logout():
    session.clear()
    flash("Logged out.", "info")
    return redirect(url_for("login"))


@app.get("/dashboard")
@login_required
def dashboard():
    return render_template("dashboard.html", username=session.get("username"))
```

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>Login</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.6.2/dist/css/bootstrap.min.css">
</head>
<body class="p-4">
<div class="container" style="max-width:420px">
    <h4 class="mb-3">Login (Session-based)</h4>

    {% with messages = get_flashed_messages(with_categories=true) %}
    {% if messages %}
    {% for c,m in messages %}
    <div class="alert alert-{{ c }}">{{ m }}</div>
    {% endfor %}
    {% endif %}
    {% endwith %}

    <form method="post">
        <div class="form-group">
            <label>Username</label>
            <input class="form-control" name="username" placeholder="admin" required>
        </div>
        <div class="form-group">
            <label>Password</label>
            <input class="form-control" type="password" name="password" placeholder="admin123" required>
        </div>
        <div class="form-check mb-3">
            <input class="form-check-input" type="checkbox" id="remember" name="remember">
            <label class="form-check-label" for="remember">Remember me</label>
        </div>
        <button class="btn btn-primary btn-block">Login</button>
    </form>
</div>
</body>
</html>

```
