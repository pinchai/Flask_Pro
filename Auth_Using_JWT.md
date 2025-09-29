```python
from datetime import timedelta
from flask import Flask, request, jsonify
from flask_jwt_extended import (
    JWTManager, create_access_token, create_refresh_token,
    jwt_required, get_jwt_identity, get_jwt
)
from werkzeug.security import check_password_hash, generate_password_hash

app = Flask(__name__)

# --- JWT config ---
app.config["JWT_SECRET_KEY"] = "change-me"  # put in ENV in production
app.config["JWT_ACCESS_TOKEN_EXPIRES"] = timedelta(minutes=30)
app.config["JWT_REFRESH_TOKEN_EXPIRES"] = timedelta(days=7)

jwt = JWTManager(app)

# --- demo user store (replace with DB later) ---
USERS = {
    "admin": generate_password_hash("admin123")
}

# --- blocklist for revoked JTIs (in-memory demo) ---
REVOKED_JTIS = set()


@jwt.token_in_blocklist_loader
def check_if_token_revoked(jwt_header, jwt_data):
    return jwt_data["jti"] in REVOKED_JTIS


@app.post("/login")
def login():
    data = request.get_json(silent=True) or {}
    username = (data.get("username") or "").strip()
    password = data.get("password") or ""

    if username not in USERS or not check_password_hash(USERS[username], password):
        return jsonify({"msg": "Bad username or password"}), 401

    access = create_access_token(identity=username)
    refresh = create_refresh_token(identity=username)
    return jsonify(access_token=access, refresh_token=refresh)


@app.post("/refresh")
@jwt_required(refresh=True)
def refresh():
    user = get_jwt_identity()
    new_access = create_access_token(identity=user)
    return jsonify(access_token=new_access)


@app.post("/logout")
@jwt_required()  # revoke current access token
def logout():
    jti = get_jwt()["jti"]
    REVOKED_JTIS.add(jti)
    return jsonify(msg="Access token revoked")


@app.post("/logout_all")
@jwt_required()  # optional: also revoke refresh token if you pass it
def logout_all():
    # In practice, add *both* current access + refresh token JTIs to the blocklist.
    # Here we only revoke the current one for demo.
    REVOKED_JTIS.add(get_jwt()["jti"])
    return jsonify(msg="Logged out everywhere (demo)")


@app.get("/me")
@jwt_required()
def me():
    user = get_jwt_identity()
    return jsonify(user=user)


# --- Example protected business endpoint ---
@app.get("/secret-data")
@jwt_required()
def secret_data():
    return jsonify(data="🥷 protected payload")


if __name__ == "__main__":
    app.run(debug=True)

```
