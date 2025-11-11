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
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Login | Attendance System</title>
<link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
</head>
<body>
<div class="login-container">
  <h1>Login</h1>
  <select id="role">
    <option value="">Select Role</option>
    <option value="admin">Admin</option>
    <option value="student">Student</option>
  </select>
  <input type="text" id="username" placeholder="Username">
  <input type="password" id="password" placeholder="Password">
  <button onclick="login()">Login</button>
</div>
<script src="{{ url_for('static', filename='script.js') }}"></script>
</body>
</html>



<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Admin Dashboard</title>
<link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
</head>
<body>
<header>
<h2>Admin Dashboard</h2>
<button onclick="logout()">Logout</button>
</header>
<main class="container">
  <section class="card">
    <h3>Register Student</h3>
    <form action="{{ url_for('register') }}" method="post">
      <input type="text" name="name" placeholder="Name" required>
      <input type="text" name="roll" placeholder="Roll Number" required>
      <input type="text" name="dept" placeholder="Department" required>
      <button type="submit">Register</button>
    </form>
  </section>
  <section class="card">
    <h3>Student List</h3>
    <table>
      <thead><tr><th>Name</th><th>Roll</th><th>Dept</th></tr></thead>
      <tbody>
      {% for s in students %}
        <tr><td>{{ s[1] }}</td><td>{{ s[2] }}</td><td>{{ s[3] }}</td></tr>
      {% endfor %}
      </tbody>
    </table>
  </section>
</main>
<script src="{{ url_for('static', filename='script.js') }}"></script>
</body>
</html>

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Student Portal</title>
<link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
</head>
<body>
<header>
<h2>Student Portal</h2>
<button onclick="logout()">Logout</button>
</header>
<main class="container">
  <section class="card">
    <h3>Mark Attendance</h3>
    <video id="camera" autoplay></video>
    <button onclick="startCamera()">Start Camera</button>
    <button onclick="markAttendance('{{ roll }}')">Mark Attendance</button>
    <table id="myAttendance">
      <thead><tr><th>Date</th><th>Time</th><th>Status</th></tr></thead>
      <tbody></tbody>
    </table>
  </section>
</main>
<script src="{{ url_for('static', filename='script.js') }}"></script>
</body>
</html>



body{font-family:'Poppins',sans-serif;background:#eef2ff;margin:0;}
header{background:#1e3a8a;color:white;padding:15px;display:flex;justify-content:space-between;align-items:center;}
button{background:#1e3a8a;color:white;padding:8px 14px;border:none;border-radius:6px;cursor:pointer;}
button:hover{background:#2563eb;}
.container{display:flex;flex-wrap:wrap;gap:20px;padding:20px;justify-content:center;}
.card{background:white;border-radius:10px;padding:20px;width:350px;box-shadow:0 3px 8px rgba(0,0,0,0.1);}
input,select{width:100%;padding:8px;margin-bottom:8px;border:1px solid #ccc;border-radius:6px;}
table{width:100%;border-collapse:collapse;margin-top:10px;}
th,td{border:1px solid #ccc;padding:8px;text-align:center;}
th{background:#1e3a8a;color:white;}
.login-container{width:320px;margin:100px auto;background:white;padding:25px;border-radius:12px;box-shadow:0 4px 10px rgba(0,0,0,0.1);text-align:center;}







function login(){
  const role = document.getElementById("role").value;
  if(role==="admin") window.location="/admin";
  else if(role==="student") window.location="/student/123";
  else alert("Select role!");
}
function logout(){ window.location="/"; }
function startCamera(){
  const video=document.getElementById('camera');
  navigator.mediaDevices.getUserMedia({video:true})
    .then(stream=>video.srcObject=stream)
    .catch(err=>alert("Camera error:"+err));
}
function markAttendance(roll){
  fetch('/mark', {
    method:'POST',
    headers:{'Content-Type':'application/x-www-form-urlencoded'},
    body:`roll=${roll}&name=Student`
  }).then(res=>res.json()).then(data=>alert(data.message));
}



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