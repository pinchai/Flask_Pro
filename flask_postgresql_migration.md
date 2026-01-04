# Flask + PostgreSQL + Database Migration

A practical setup for using **Flask**, **PostgreSQL**, and **database migrations** in real-world applications such as POS systems.

---

## Architecture Overview

- **Flask** – Web framework  
- **PostgreSQL** – Relational database  
- **SQLAlchemy** – ORM (Object Relational Mapper)  
- **Flask-Migrate (Alembic)** – Database schema migrations  

---

## 1. Install Required Packages

```bash
pip install flask
pip install flask-sqlalchemy 
pip install flask-migrate
pip install psycopg2-binary
```

---

## 2. Configure PostgreSQL Connection

```python
# config.py
SQLALCHEMY_DATABASE_URI = (
    "postgresql://username:password@localhost:5432/pos_db"
)
SQLALCHEMY_TRACK_MODIFICATIONS = False
```

---

## 3. Initialize Flask, SQLAlchemy, and Migrate

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate

app = Flask(__name__)
app.config.from_object("config")

db = SQLAlchemy(app)
migrate = Migrate(app, db)
```

---

## 4. Create POS Models

```python
from app import db

class Category(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False, unique=True)

class Product(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(150), nullable=False)
    price = db.Column(db.Numeric(10, 2), nullable=False)

    category_id = db.Column(
        db.Integer,
        db.ForeignKey("category.id"),
        nullable=False
    )
```

---

## 5. Initialize Migrations

```bash
flask db init
```

---

## 6. Generate Migration

```bash
flask db migrate -m "create product and category tables"
```

---

## 7. Apply Migration

```bash
flask db upgrade
```

---

## 8. Common Migration Operations

### Add Column
```python
db.Column("stock", db.Integer, nullable=False, server_default="0")
```

### Rename Column
```python
op.alter_column("product", "cost", new_column_name="purchase_price")
```

### Drop Column
```python
op.drop_column("product", "old_field")
```

---

## 9. Migration Commands

```bash
flask db history
flask db current
flask db downgrade -1
```

---

## Best Practices

- Do not use `db.create_all()` in production
- Always use migrations
- Use `Numeric` for money
- Add unique constraints for SKU/barcode
- Test migrations before production
- Commit migration files to Git

---

## Recommended Stack

```
Flask
SQLAlchemy
Flask-Migrate
PostgreSQL
```
