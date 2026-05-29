# Flask Configuration & Extension Initialization Documentation

## 1. Configuration Management (`config.py`)

Configuration variables are organized inside a class named `Config` within `config.py`.

### Source Code (`config.py`)
```python
class Config:
    # SQLALCHEMY_DATABASE_URI = "sqlite:///mydb.sqlite3"

    # pip install pymysql
    SQLALCHEMY_DATABASE_URI = "mysql+pymysql://root:@localhost/ssssss"

    SQLALCHEMY_TRACK_MODIFICATIONS = False

    SECRET_KEY = "super-secret-key"
```

### Loading Configuration (`app.py`)
In the main application entry point (`app.py`), the configuration class is imported and loaded onto the Flask application object:

```python
from config import Config

# Load configuration from the Config class object
app.config.from_object(Config)
```
---

## Extension Management (`extensions.py`)
### Source Code (`extensions.py`)
```python
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate
from flask_wtf import CSRFProtect

db = SQLAlchemy()
migrate = Migrate()
csrf = CSRFProtect()
```

### Source Code Segment (`app.py`)
```python
from extensions import db, migrate, csrf

# ... Flask app instantiation ...

# Initialize the extensions
db.init_app(app)
csrf.init_app(app)
migrate.init_app(app, db)
```
---
