# Flask-Migrate, Raw SQL, and ORM with Flask-SQLAlchemy

This handout introduces three important concepts when working with Flask and databases:

1. **Flask-Migrate** – database migrations (schema versioning)
2. **Raw SQL** – executing custom SQL queries
3. **ORM (Object-Relational Mapping)** – interacting with tables using Python classes

---

## 1. Flask-Migrate Setup

Flask-Migrate is a wrapper around **Alembic**, allowing you to manage database schema changes.

### Installation
```bash
pip install flask-migrate
```

### Example Code
```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate

app = Flask(__name__)
app.config["SQLALCHEMY_DATABASE_URI"] = "sqlite:///mydb.sqlite3"
app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False

db = SQLAlchemy(app)
migrate = Migrate(app, db)

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), nullable=False)
    email = db.Column(db.String(120), nullable=False, unique=True)

if __name__ == "__main__":
    app.run(debug=True)
```

### Migration Commands
```bash
flask db init                     # create migrations/ folder
flask db migrate -m "init tables" # detect changes and generate migration
flask db upgrade                  # apply changes to DB
flask db downgrade                # rollback changes
```

---

## 2. Raw SQL with Flask-SQLAlchemy

Even with SQLAlchemy, sometimes you need raw SQL.

### Select All Users
```python
from sqlalchemy import text

@app.route("/raw_users")
def raw_users():
    sql = text("SELECT * FROM user")
    result = db.session.execute(sql)
    rows = [dict(row._mapping) for row in result]
    return {"users": rows}
```

### Safe Parameterized Query
```python
@app.route("/raw_user/<name>")
def raw_user(name):
    sql = text("SELECT * FROM user WHERE username = :username")
    result = db.session.execute(sql, {"username": name}).fetchone()
    if result:
        return dict(result._mapping)
    return {"error": "User not found"}
```

### Insert Record
```python
@app.route("/raw_add/<name>/<email>")
def raw_add(name, email):
    sql = text("INSERT INTO user (username, email) VALUES (:n, :e)")
    db.session.execute(sql, {"n": name, "e": email})
    db.session.commit()
    return {"status": "added"}
```

---

## 3. ORM Style (Recommended)

Instead of raw SQL, use ORM for cleaner and safer code.

### Select All Users
```python
@app.route("/orm_users")
def orm_users():
    users = User.query.all()
    return {"users": [{"id": u.id, "username": u.username, "email": u.email} for u in users]}
```

### Insert User
```python
@app.route("/orm_add/<name>/<email>")
def orm_add(name, email):
    u = User(username=name, email=email)
    db.session.add(u)
    db.session.commit()
    return {"status": "added"}
```

---

## 4. When to Use What?

| Approach       | Advantages | Disadvantages |
|----------------|------------|---------------|
| **ORM**        | Easy to use, Pythonic, integrates with migrations | Can hide SQL details |
| **Raw SQL**    | Great for custom queries, reports, performance tuning | Verbose, risk of SQL injection |
| **Migrations** | Handles schema changes safely | Requires migration workflow |

---

✅ **Best Practice:** Use **ORM** for most cases, **Raw SQL** for special needs, and always manage schema changes with **Flask-Migrate**.
