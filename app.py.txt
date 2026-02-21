from flask import Flask, render_template, request, send_file
from PIL import Image
import os

app = Flask(__name__)
UPLOAD_FOLDER = "uploads"
os.makedirs(UPLOAD_FOLDER, exist_ok=True)

@app.route("/")
def home():
    return render_template("index.html")

@app.route("/convert", methods=["POST"])
def convert():
    files = request.files.getlist("files")

    images = []
    for file in files:
        path = os.path.join(UPLOAD_FOLDER, file.filename)
        file.save(path)
        img = Image.open(path).convert("RGB")
        images.append(img)

    if not images:
        return "No images uploaded"

    output = os.path.join(UPLOAD_FOLDER, "merged.pdf")
    images[0].save(output, save_all=True, append_images=images[1:])

    return send_file(output, as_attachment=True)

if __name__ == "__main__":
    app.run()