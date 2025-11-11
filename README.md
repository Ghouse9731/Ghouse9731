## Hi there 👋

<!--
**Ghouse9731/Ghouse9731** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->



from flask import Flask, render_template, request, redirect, url_for, jsonify
import sqlite3, os, cv2, qrcode, face_recognition
from datetime import datetime

app = Flask(__name__)
os.makedirs('face_data', exist_ok=True)

# Initialize database
def init_db():
    conn = sqlite3.connect('database.db')
    conn.execute('CREATE TABLE IF NOT EXISTS students (id INTEGER PRIMARY KEY, name TEXT, roll TEXT, dept TEXT)')
    conn.execute('CREATE TABLE IF NOT EXISTS attendance (id INTEGER PRIMARY KEY, roll TEXT, name TEXT, date TEXT, time TEXT)')
    conn.close()

init_db()

@app.route('/')
def login():
    return render_template('index.html')

@app.route('/admin')
def admin():
    conn = sqlite3.connect('database.db')
    students = conn.execute("SELECT * FROM students").fetchall()
    conn.close()
    return render_template('admin.html', students=students)

@app.route('/student/<roll>')
def student(roll):
    return render_template('student.html', roll=roll)

@app.route('/register', methods=['POST'])
def register():
    name = request.form['name']
    roll = request.form['roll']
    dept = request.form['dept']

    conn = sqlite3.connect('database.db')
    conn.execute("INSERT INTO students (name, roll, dept) VALUES (?, ?, ?)", (name, roll, dept))
    conn.commit()
    conn.close()

    qr = qrcode.make(f"{roll}-{name}")
    qr.save(f"static/{roll}.png")

    cam = cv2.VideoCapture(0)
    ret, frame = cam.read()
    if ret:
        cv2.imwrite(f"face_data/{roll}.jpg", frame)
    cam.release()

    return redirect(url_for('admin'))

@app.route('/mark', methods=['POST'])
def mark_attendance():
    roll = request.form['roll']
    name = request.form['name']
    now = datetime.now()
    conn = sqlite3.connect('database.db')
    conn.execute("INSERT INTO attendance (roll, name, date, time) VALUES (?, ?, ?, ?)",
                 (roll, name, now.strftime("%Y-%m-%d"), now.strftime("%H:%M:%S")))
    conn.commit()
    conn.close()
    return jsonify({"message": "Attendance marked"})

@app.route('/get_attendance')
def get_attendance():
    conn = sqlite3.connect('database.db')
    data = conn.execute("SELECT * FROM attendance").fetchall()
    conn.close()
    return jsonify(data)

if __name__ == "__main__":
    app.run(debug=True)