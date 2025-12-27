```bash
pip install pillow
```

---

## config.py

```python
import os

BASE_DIR = os.path.dirname(os.path.abspath(__file__))

UPLOAD_FOLDER = os.path.join(BASE_DIR, 'static/uploads')
ALLOWED_EXTENSIONS = {'png', 'jpg', 'jpeg', 'gif'}
MAX_CONTENT_LENGTH = 2 * 1024 * 1024  # 2MB
```

---

## upload_service.py

```python
import os
from werkzeug.utils import secure_filename
from PIL import Image


def allowed_file(filename, allowed_extensions):
    return (
        '.' in filename and
        filename.rsplit('.', 1)[1].lower() in allowed_extensions
    )


def save_image(
    file,
    upload_folder,
    allowed_extensions,
    resize_to=(800, 800),
    thumb_size=(200, 200)
):
    if not file or file.filename == '':
        return None

    if not allowed_file(file.filename, allowed_extensions):
        return None

    filename = secure_filename(file.filename)
    name, ext = os.path.splitext(filename)

    original_path = os.path.join(upload_folder, filename)
    resized_path = os.path.join(upload_folder, f"{name}_resized{ext}")
    thumb_path = os.path.join(upload_folder, f"{name}_thumb{ext}")

    file.save(original_path)

    image = Image.open(original_path)

    resized = image.copy()
    resized.thumbnail(resize_to)
    resized.save(resized_path)

    thumb = image.copy()
    thumb.thumbnail(thumb_size)
    thumb.save(thumb_path)

    return {
        "original": filename,
        "resized": f"{name}_resized{ext}",
        "thumbnail": f"{name}_thumb{ext}"
    }
```

---

## app.py

```python
from flask import Flask, render_template, request
import os
import config
from upload_service import save_image

app = Flask(__name__)
app.config.from_object(config)

os.makedirs(app.config['UPLOAD_FOLDER'], exist_ok=True)


@app.route('/', methods=['GET', 'POST'])
def upload_image():
    images = None

    if request.method == 'POST':
        file = request.files.get('image')
        images = save_image(
            file,
            app.config['UPLOAD_FOLDER'],
            app.config['ALLOWED_EXTENSIONS']
        )

    return render_template('upload.html', images=images)


if __name__ == '__main__':
    app.run(debug=True)
```

---

## templates

```html
<!doctype html>
<html>
<head>
  <title>Upload Image</title>
</head>
<body>
  <h2>Upload Image</h2>

  <form method="POST" enctype="multipart/form-data">
    <input type="file" name="image" accept="image/*">
    <button type="submit">Upload</button>
  </form>

  {% if images %}
    <h3>Original</h3>
    <img src="{{ url_for('static', filename='uploads/' + images.original) }}" width="300">

    <h3>Resized</h3>
    <img src="{{ url_for('static', filename='uploads/' + images.resized) }}">

    <h3>Thumbnail</h3>
    <img src="{{ url_for('static', filename='uploads/' + images.thumbnail) }}">
  {% endif %}
</body>
</html>
```
---
