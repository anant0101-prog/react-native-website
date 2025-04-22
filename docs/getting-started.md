pip install Flaskcricket_fantasy/
├── app.py
├── templates/
│   ├── index.html
│   └── team_selection.html
└── static/
    └── style.cssfrom flask import Flask, render_template, request, redirect, url_for, session
import sqlite3

app = Flask(__name__)
app.secret_key = 'your_secret_key'

# Database setup
def init_db():
    with sqlite3.connect('fantasy_cricket.db') as conn:
        cursor = conn.cursor()
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                username TEXT UNIQUE,
                password TEXT
            )
        ''')
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS players (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT,
                points INTEGER
            )
        ''')
        conn.commit()

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/register', methods=['GET', 'POST'])
def register():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        with sqlite3.connect('fantasy_cricket.db') as conn:
            cursor = conn.cursor()
            cursor.execute('INSERT INTO users (username, password) VALUES (?, ?)', (username, password))
            conn.commit()
        return redirect(url_for('index'))
    return render_template('register.html')

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        with sqlite3.connect('fantasy_cricket.db') as conn:
            cursor = conn.cursor()
            cursor.execute('SELECT * FROM users WHERE username=? AND password=?', (username, password))
            user = cursor.fetchone()
            if user:
                session['user_id'] = user[0]
                return redirect(url_for('team_selection'))
    return render_template('login.html')

@app.route('/team_selection', methods=['GET', 'POST'])
def team_selection():
    if 'user_id' not in session:
        return redirect(url_for('login'))
    
    if request.method == 'POST':
        # Handle team selection logic here
        pass

    # Fetch players from the database
    with sqlite3.connect('fantasy_cricket.db') as conn:
        cursor = conn.cursor()
        cursor.execute('SELECT * FROM players')
        players = cursor.fetchall()
    return render_template('team_selection.html', players=players)

if __name__ == '__main__':
    init_db()
    app.run(debug=True)<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
    <title>Cricket Fantasy</title>
</head>
<body>
    <h1>Welcome to Cricket Fantasy</h1>
    <a href="{{ url_for('register') }}">Register</a>
    <a href="{{ url_for('login') }}">Login</a>
</body>
</html><!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
    <title>Select Your Team</title>
</head>
<body>
    <h1>Select Your Team</h1>
    <form method="POST">
        {% for player in players %}
            <div>
                <input type="checkbox" name="players" value="{{ player[0] }}">
                <label>{{ player[1] }} - {{ player[2] }} points</label>
            </div>
        {% endfor %}
        <button type="submit">Submit
