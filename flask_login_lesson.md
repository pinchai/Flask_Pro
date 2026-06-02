# Flask-Login Lesson Outline

## 1. Installation

### Install Flask-Login

```bash
pip install flask-login
```

### Verify Installation

```bash
pip show flask-login
```

---

# 2. Configuring your Application

### Basic Setup

```python
from flask import Flask
from flask_login import LoginManager

app = Flask(__name__)
app.secret_key = "secret-key"

login_manager = LoginManager()
login_manager.init_app(app)
```

### Configure Login View

```python
login_manager.login_view = "auth.login"
```

When a user tries to access a protected page without logging in, Flask-Login redirects them to this route.

---

# 3. How it Works

Flask-Login manages user authentication using sessions.

### Main Workflow

1. User logs in
2. `login_user(user)` stores user ID in session
3. Flask-Login loads the user using `user_loader`
4. `current_user` becomes available
5. Protected routes use `@login_required`

### Important Components

| Component | Purpose |
|---|---|
| `login_user()` | Log in user |
| `logout_user()` | Log out user |
| `current_user` | Current logged-in user |
| `login_required` | Protect routes |
| `UserMixin` | Adds default user methods |

---

# 4. Your User Class

Your user class should inherit from `UserMixin`.

### Example

```python
from flask_login import UserMixin

class User(UserMixin):

    def __init__(self, id, email, profile, is_admin):
        self.id = id
        self.email = email
        self.profile = profile
        self.is_admin = is_admin
```

---

# 5. Login Example

## User Loader

Flask-Login reloads the user from the session.

```python
@login_manager.user_loader
def load_user(user_id):

    row = db.execute(
        "SELECT * FROM users WHERE id=?",
        (user_id,)
    ).fetchone()

    if row:
        return User(
            id=row["id"],
            email=row["email"],
            profile=row["profile"],
            is_admin=row["is_admin"]
        )

    return None
```

---

## Login Route

```python
from flask import request, redirect, url_for
from flask_login import login_user

@app.route("/login", methods=["POST"])
def do_login():

    email = request.form.get("email")
    password = request.form.get("password")

    row = db.execute(
        "SELECT * FROM users WHERE email=?",
        (email,)
    ).fetchone()

    if row and row["password"] == password:

        user = User(
            id=row["id"],
            email=row["email"],
            profile=row["profile"],
            is_admin=row["is_admin"]
        )

        login_user(user)

        return redirect(url_for("dashboard"))

    return "Invalid login"
```

---

## Protected Route

```python
from flask_login import login_required, current_user

@app.route("/dashboard")
@login_required
def dashboard():

    return f"Welcome {current_user.email}"
```

---

## Logout

```python
from flask_login import logout_user

@app.route("/logout")
@login_required
def logout():

    logout_user()

    return "Logged out"
```

---

# 6. Customizing the Login Process

## Change Login Message

```python
login_manager.login_message = "Please login first."
```

---

## Change Message Category

```python
login_manager.login_message_category = "warning"
```

---

## Unauthorized Handler

```python
@login_manager.unauthorized_handler
def unauthorized():

    return {
        "error": "Authentication required"
    }, 401
```

Useful for APIs.

---

# 7. Custom Login using Request Loader

Request loader allows login without sessions.

Useful for:

- API Tokens
- API Keys
- Custom Headers

---

## Example

```python
@login_manager.request_loader
def load_user_from_request(request):

    api_key = request.headers.get("API-KEY")

    if not api_key:
        return None

    row = db.execute(
        "SELECT * FROM users WHERE api_key=?",
        (api_key,)
    ).fetchone()

    if row:
        return User(
            id=row["id"],
            email=row["email"],
            profile=row["profile"],
            is_admin=row["is_admin"]
        )

    return None
```

---

# 8. Anonymous Users

When no user is logged in:

```python
current_user.is_authenticated
```

returns:

```python
False
```

---

## Example

```python
from flask_login import current_user

@app.route("/")
def home():

    if current_user.is_authenticated:
        return f"Hello {current_user.email}"

    return "Guest User"
```

---

# 9. Remember Me

Keeps users logged in after browser restart.

---

## Login with Remember Me

```python
login_user(user, remember=True)
```

---

## Set Remember Duration

```python
from datetime import timedelta

app.config["REMEMBER_COOKIE_DURATION"] = timedelta(days=7)
```

---

# 10. Alternative Tokens

Instead of storing real user IDs in cookies, use alternative IDs.

More secure.

---

## Example

```python
class User(UserMixin):

    def get_id(self):
        return self.email
```

Now Flask-Login stores email instead of numeric ID.

---

# 11. Fresh Logins

Fresh login means the user recently entered credentials.

Useful for:

- Password change
- Payment
- Security settings

---

## Fresh Login Example

```python
login_user(user, fresh=True)
```

---

## Protect Sensitive Route

```python
from flask_login import fresh_login_required

@app.route("/settings")
@fresh_login_required
def settings():
    return "Sensitive Settings"
```

---

# 12. Cookie Settings

Customize Flask-Login cookies.

---

## Example

```python
app.config["REMEMBER_COOKIE_NAME"] = "remember_token"

app.config["REMEMBER_COOKIE_HTTPONLY"] = True

app.config["REMEMBER_COOKIE_SECURE"] = True

app.config["REMEMBER_COOKIE_SAMESITE"] = "Lax"
```

---

## Common Settings

| Setting | Description |
|---|---|
| `REMEMBER_COOKIE_NAME` | Cookie name |
| `REMEMBER_COOKIE_DURATION` | Expiration |
| `REMEMBER_COOKIE_SECURE` | HTTPS only |
| `REMEMBER_COOKIE_HTTPONLY` | Disable JS access |
| `REMEMBER_COOKIE_SAMESITE` | CSRF protection |

---

# 13. Session Protection

Flask-Login can detect session hijacking.

---

## Enable Protection

```python
login_manager.session_protection = "strong"
```

---

## Modes

| Mode | Description |
|---|---|
| `"basic"` | Detect suspicious changes |
| `"strong"` | Force re-login on mismatch |

---

# 14. Disabling Session Cookie for APIs

APIs usually use tokens instead of sessions.

Disable session saving for API requests.

---

## Example

```python
from flask import request

@login_manager.request_loader
def load_user_from_request(request):

    token = request.headers.get("Authorization")

    if token == "secret-token":
        return User(1, "admin@gmail.com", "profile.png", True)

    return None
```

---

## Disable Session Creation

```python
from flask import g

@app.before_request
def disable_session_for_api():

    if request.path.startswith("/api/"):
        g.login_via_request = True
```

---

## Custom Session Interface

```python
from flask.sessions import SecureCookieSessionInterface

class CustomSessionInterface(SecureCookieSessionInterface):

    def save_session(self, *args, **kwargs):

        if g.get("login_via_request"):
            return

        return super().save_session(*args, **kwargs)

app.session_interface = CustomSessionInterface()
```
