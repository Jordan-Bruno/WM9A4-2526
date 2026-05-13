# Student Guide: User Login, Flask Sessions, & Logout

Welcome to **Session 6, Exercise 2**! 

In Exercise 1, we successfully registered users and safely encrypted their passwords using one-way hashing. But right now, registered users have no way to actually log into their accounts!

In this exercise, we will build the **Login** and **Logout** workflows. To understand how login works, we first need to understand a core rule of the internet: **HTTP is Stateless.**

### What does "Stateless" mean?
Every time your web browser sends a request to a server (like clicking a link or submitting a form), the server treats it as a completely independent, isolated event. The server has **zero memory** of your previous requests. 
- If you log in on page 1, and then click a link to visit page 2, the server instantly forgets who you are!

### The Solution: Cookies and Sessions
To make our application "remember" a logged-in user as they browse around, we use **Sessions**.
1. When a user types in their correct username and password, our Flask server creates a tiny data packet called a **Cookie** containing their User ID.
2. The server sends this cookie back to the user's web browser.
3. From that moment on, every time the browser requests a new page, it automatically attaches that cookie to the request.
4. Flask reads the cookie, recognizes the User ID, and knows exactly who is browsing!

To prevent hackers from faking or modifying these cookies, Flask cryptographically **signs** them using a secret password called a **Secret Key**.

Let's implement this!

---

## Step 1: Open the Starter Files

This exercise continues on from where you left off in Exercise 1, you may wish to copy your `app.py` and your templates/static folder into a new project before beginning this exercise.

---

## Step 2: Import Tools and Set the Secret Key

Open `app.py`. 

1. At the top of your file, update your imports to include `session` from `flask`, and `check_password_hash` from `werkzeug.security`:

```python
from flask import Flask, render_template, request, redirect, url_for, session # <--- ADD session
from flask_sqlalchemy import SQLAlchemy
# Import check_password_hash alongside generate_password_hash:
from werkzeug.security import generate_password_hash, check_password_hash # <--- ADD check_password_hash
```

2. Below your `app = Flask(__name__)` initialization, configure your application's **Secret Key**:

```python
app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///campus.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

# ADD THIS LINE TO ENABLE SECURE SESSIONS:
# This signs our browser cookies so users cannot tamper with them.
app.secret_key = 'super_secret_campus_key_change_in_production'
```

> **Warning:** If you try to use `session` without setting `app.secret_key`, your Flask application will crash with a `RuntimeError`!

---

## Step 3: Create the Login HTML Form

Let's create the web page where users enter their credentials.

1. Inside the `templates/` folder, create a new file named `login.html`.
2. Copy and paste the login form structure below:

```html
{% extends "base.html" %}

{% block title %}Login - Campus Confessions{% endblock %}

{% block content %}
<div class="card auth-card">
    <h1 style="text-align: center; margin-bottom: 0.5rem; color: var(--primary-color);">Welcome Back</h1>
    <p style="text-align: center; color: var(--text-muted); margin-bottom: 2rem; font-size: 0.95rem;">
        Log into your account to post notes.
    </p>

    <!-- Display error alert if login fails -->
    {% if error %}
    <div class="flash-alert" style="background-color: #fef2f2; color: #991b1b; border-color: #fecaca;">
        <span>{{ error }}</span>
    </div>
    {% endif %}

    <form method="POST" action="{{ url_for('login') }}">
        <div class="form-group">
            <label for="username">Username</label>
            <input type="text" id="username" name="username" placeholder="e.g. coolstudent99" required>
        </div>

        <div class="form-group">
            <label for="password">Password</label>
            <input type="password" id="password" name="password" placeholder="••••••••" required>
        </div>

        <button type="submit" class="btn">Log In</button>
    </form>

    <div style="text-align: center; margin-top: 1.5rem; font-size: 0.9rem; color: var(--text-muted);">
        Don't have an account yet? <a href="{{ url_for('register') }}" style="color: var(--primary-color); font-weight: 500;">Register here</a>
    </div>
</div>
{% endblock %}
```

---

## Step 4: Implement the `/login` Route

Now we need the Python logic to verify the user's password and start their session. 

Because we hashed the password in Exercise 1, we can't just check `if user.password_hash == form_password`. We must use `check_password_hash(stored_hash, form_password)` to safely compare them!

In `app.py`, paste this route below your `register()` function:

```python
@app.route("/login", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        form_username = request.form["username"].strip()
        form_password = request.form["password"]
        
        # Step 1: Query the database to find the user by username
        user = User.query.filter_by(username=form_username).first()
        
        # Step 2: Verify if the user exists AND the password matches the hash!
        if user and check_password_hash(user.password_hash, form_password):
            # Step 3: START THE SESSION!
            # We store their ID and username inside the session dictionary.
            # Flask automatically packages this into a secure browser cookie.
            session['user_id'] = user.id
            session['username'] = user.username
            
            print(f"Login successful for: {user.username}")
            return redirect(url_for("index"))
        else:
            # If the user doesn't exist, or they typed the wrong password:
            return render_template("login.html", error="Invalid username or password.")
            
    # If request.method is GET, show the blank login form
    return render_template("login.html")
```

*(Optional Polish: Update your `/register` route so that when a user successfully creates an account, it redirects them to `login` instead of `index`! Change `return redirect(url_for("index"))` inside `register()` to `return redirect(url_for("login"))`).*

---

## Step 5: Implement the `/logout` Route

Logging out is extremely simple: we just delete the user's data from the `session` dictionary! Once removed, the server no longer recognizes their browser cookie.

Paste this short route function at the bottom of your routes in `app.py`:

```python
@app.route("/logout")
def logout():
    # Remove user_id and username from the session
    session.pop('user_id', None)
    session.pop('username', None)
    
    print("Session cleared. User logged out.")
    return redirect(url_for("index"))
```

---

## Step 6: Make the Navigation Bar Dynamic!

Right now, your website always displays links to "Register" and "Login", even if you are already logged in. That's confusing UX. 

Let's use Jinja templating to read our `session` dictionary and change the navigation bar dynamically!

Open `templates/base.html`. Find the `<div class="nav-links">` container and replace its contents with this conditional block:

```html
<div class="nav-links">
    <a href="{{ url_for('index') }}" class="nav-link">Home</a>
    
    <!-- Check if 'user_id' exists inside the server session: -->
    {% if 'user_id' in session %}
        <!-- Show personalized greeting and Logout link to authenticated users -->
        <span style="color: var(--text-color); font-weight: 600; background-color: #f1f5f9; padding: 0.3rem 0.8rem; border-radius: 6px;">
            {{ session['username'] }}
        </span>
        <a href="{{ url_for('logout') }}" class="nav-link" style="color: var(--danger-color);">Logout</a>
    {% else %}
        <!-- Show public links to unauthenticated guests -->
        <a href="{{ url_for('register') }}" class="nav-link">Register</a>
        <a href="{{ url_for('login') }}" class="nav-link">Login</a>
    {% endif %}
</div>
```

---

## Step 7: Test Your End-to-End Workflow!

Let's see our stateful application in action.

1. **Start your server:** Run `app.py`.
2. **Open your browser:** Visit `http://127.0.0.1:5000/`. Notice the top navigation bar shows **Register** and **Login**.
3. **Register an account:** Click Register, fill out the details, and submit.
4. **Log in:** Enter your newly created credentials on the Login page. 
5. **Watch the magic:** Upon successful login, you are redirected to the homepage. Look at the top right navigation bar... It now greets you personally (`yourusername`) and displays a **Logout** button!
6. **Browse around:** Click the Home link or refresh the page. Notice the server remembers you perfectly across requests.
7. **Log out:** Click the **Logout** button. You are instantly returned to the public guest view.
8. **Test failure:** Try logging in with the wrong password to ensure your error alert displays correctly.

You now understand stateless HTTP, session cookies, and dynamic interfaces. In our final exercise (**Exercise 3**), we will use these sessions to restrict access to secure web pages and let users create their own personalised records!
