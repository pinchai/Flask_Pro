# Python-dotenv Setup Guide

This document explains how to install and use `python-dotenv` in a Flask
project.

## 1. Install python-dotenv

``` bash
pip install python-dotenv
```

## 2. Create `.env` File

``` env
DATABASE_URL=sqlite:///mydb.sqlite3
SECRET_KEY=my-secret-key
```

## 3. Create `config.py`

``` python
from dotenv import load_dotenv
import os

load_dotenv()

class Config:
    SECRET_KEY = os.getenv("SECRET_KEY")
    SQLALCHEMY_DATABASE_URI = os.getenv(
        "DATABASE_URL",
        "sqlite:///mydb.sqlite3"
    )
    SQLALCHEMY_TRACK_MODIFICATIONS = False
```

## 4. Create `app.py`

``` python
from flask import Flask
from config import Config

app = Flask(__name__)
app.config.from_object(Config)
```

## 5. Recommended Project Structure

``` text
project/
│
├── app.py
├── config.py
├── .env
├── requirements.txt
└── instance/
```

## 6. Notes

-   Add `.env` to `.gitignore`.
-   Store secrets in `.env`, not in source code.
-   Use `os.getenv()` to read environment variables.
