````python
from flask import Flask, render_template, request, redirect, url_for, flash
from werkzeug.utils import secure_filename
import os

app = Flask(__name__)
app.secret_key = "secret"
UPLOAD_DIR = os.path.join("static", "uploads")
os.makedirs(UPLOAD_DIR, exist_ok=True)

ALLOWED_EXT = {"png", "jpg", "jpeg", "gif"}

def allowed(name):
    return "." in name and name.rsplit(".", 1)[-1].lower() in ALLOWED_EXT

@app.route("/", methods=["GET", "POST"])
def upload():
    if request.method == "POST":
        file = request.files["image"]
        if file and allowed(file.filename):
            filename = secure_filename(file.filename)
            file.save(os.path.join(UPLOAD_DIR, filename))
            flash("Uploaded!")
            return redirect(url_for("upload"))
        flash("Invalid file")
    files = os.listdir(UPLOAD_DIR)
    return render_template("gallery.html", files=files)
````

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>Image Upload with Preview</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.6.2/dist/css/bootstrap.min.css">
    <style>
        #preview {
            margin-top: 1rem;
            max-width: 300px;
            max-height: 300px;
            border: 1px solid #ccc;
            display: none;
        }
    </style>
</head>
<body class="p-4">
<div class="container" style="max-width:640px">
    <h3 class="mb-3">Upload Image</h3>

    {% with messages = get_flashed_messages(with_categories=true) %}
    {% if messages %}
    {% for c,m in messages %}
    <div class="alert alert-{{ c }}">{{ m }}</div>
    {% endfor %}
    {% endif %}
    {% endwith %}

    <form method="post" enctype="multipart/form-data">
        <div class="form-group">
            <label>Choose image</label>
            <input type="file" class="form-control-file" name="image" accept="image/*" id="imageInput" required>
        </div>
        <img id="preview" alt="Preview">
        <div class="mt-3">
            <button class="btn btn-primary">Upload</button>
        </div>
    </form>

    <hr>
    <h4>Gallery</h4>
    <div class="row">
        {% for f in files %}
        <div class="col-md-3 mb-3">
            <img src="{{ url_for('static', filename='uploads/' + f) }}" class="img-fluid img-thumbnail" alt="{{ f }}">
        </div>
        {% endfor %}
    </div>
</div>

<script>
    const input = document.getElementById("imageInput");
    const preview = document.getElementById("preview");

    input.addEventListener("change", () => {
        const file = input.files[0];
        if (file) {
            const reader = new FileReader();
            reader.onload = e => {
                preview.src = e.target.result;
                preview.style.display = "block";
            };
            reader.readAsDataURL(file);
        } else {
            preview.style.display = "none";
        }
    });
</script>
</body>
</html>

```
