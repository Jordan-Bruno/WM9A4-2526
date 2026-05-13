# Student Guide: Secure User Registration & Password Hashing

Welcome to **Session 6, Exercise 1**! 

Up until now, our web applications have been completely public. Anyone visiting the site could view, create, update, or delete records without telling us who they are. 

To build exclusive, personalized features, we need to introduce **User Accounts**. In this exercise, we will build a registration system for an exclusive student application: **Campus Confessions & Secret Notes**.

Before writing code, let's talk about the most important rule of web security: **Never store plain text passwords.**

### Why are Plain Text Passwords Dangerous?
Imagine you create a website and store your users' passwords exactly as they typed them into the database (e.g., `password123`). If a malicious hacker manages to steal your database file, they will instantly know the password of every single user! Because most people reuse passwords across different websites, those hackers could then log into your users' personal email or bank accounts. 

### The Solution: Password Hashing
Instead of saving the actual password, we run it through a mathematical one-way function called a **Hash**. 
- A hash function turns `password123` into a long, scrambled string of random characters, like: `scrypt:32768:8:1$k8zL...`
- It is **one-way**: it is mathematically impossible to take the scrambled string and reverse it back to `password123`.
- Even if a hacker steals your database, your users' passwords remain completely safe!

Let's upgrade our starter application to securely register users using password hashing.

---

## Step 1: Open the Starter Files

Download the starter files from Moodle and unzip them into a new project folder. 
Take a look at `app.py`. You will see a simple Flask setup with a `Note` database model and a single homepage route.

Let's install the tool we need for secure password hashing. Open your terminal and make sure your virtual environment is active, then install Flask-SQLAlchemy if you haven't already:

**Windows:** `pip install flask-sqlalchemy`  
**Mac:** `pip3 install flask-sqlalchemy`

*(Note: Flask automatically includes a built-in library called `werkzeug` which contains our secure hashing tools, so we don't need to install any extra packages for hashing!)*

---

## Step 2: Import the Hashing Utility

Open `app.py`. At the very top of your file, add the import statement for `generate_password_hash`:

```python
from flask import Flask, render_template, request, redirect, url_for
from flask_sqlalchemy import SQLAlchemy
from werkzeug.security import generate_password_hash  # <--- ADD THIS LINE
```

---

## Step 3: Define the `User` Database Model

We need a database table to store our registered users. Every user will have a unique ID, a unique username, a unique email address, and their encrypted password hash.

Find your `Note` model in `app.py` and paste the new `User` class directly below it:

```python
class Note(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    content = db.Column(db.Text, nullable=False)

# ADD THIS NEW CLASS BELOW Note:
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(50), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    # Notice we name the column password_hash, not password!
    # We allocate 256 characters because generated hashes are very long strings.
    password_hash = db.Column(db.String(256), nullable=False)
```

---

## Step 4: Create the Registration HTML Form

Now we need a web page where users can type in their details. 

1. Inside the `templates/` folder, create a new file named `register.html`.
2. Copy and paste the following registration form structure:

```html
{% extends "base.html" %}

{% block title %}Register - Campus Confessions{% endblock %}

{% block content %}
<div class="card auth-card">
    <h1 style="text-align: center; margin-bottom: 0.5rem; color: var(--primary-color);">Create an Account</h1>
    <p style="text-align: center; color: var(--text-muted); margin-bottom: 2rem; font-size: 0.95rem;">
        Join your campus community to post notes.
    </p>

    <!-- If there is an error message passed from Python, display it here! -->
    {% if error %}
    <div class="flash-alert" style="background-color: #fef2f2; color: #991b1b; border-color: #fecaca;">
        <span>{{ error }}</span>
    </div>
    {% endif %}

    <!-- Submit the data using a secure POST request -->
    <form method="POST" action="{{ url_for('register') }}">
        <div class="form-group">
            <label for="username">Username</label>
            <input type="text" id="username" name="username" placeholder="e.g. coolstudent99" required>
        </div>

        <div class="form-group">
            <label for="email">Student Email</label>
            <input type="email" id="email" name="email" placeholder="student@university.ac.uk" required>
        </div>

        <div class="form-group">
            <label for="password">Password</label>
            <input type="password" id="password" name="password" placeholder="••••••••" required>
        </div>

        <button type="submit" class="btn">Register Account</button>
    </form>
</div>
{% endblock %}
```

### Link the Navigation Bar
Open `templates/base.html`. Find the navigation links section and update the `Register` link so users can click it to reach our form:

**Change this placeholder link:**
```html
<a href="#" class="nav-link">Register</a>
```

**To use `url_for`:**
```html
<a href="{{ url_for('register') }}" class="nav-link">Register</a>
```

---

## Step 5: Implement the `/register` Route Logic

Let's wire up the Python logic to process the form submission. We must handle both viewing the blank form (`GET` request) and processing the user's submitted data (`POST` request).

In `app.py`, paste this route below your `index()` route function:

```python
@app.route("/register", methods=["GET", "POST"])
def register():
    # If the user submitted the form:
    if request.method == "POST":
        # 1. Extract data typed into the input boxes
        form_username = request.form["username"].strip()
        form_email = request.form["email"].strip()
        form_password = request.form["password"]
        
        # 2. Check if a user with this username or email already exists.
        # If we skip this, our database will throw a fatal IntegrityError crash!
        existing_user = User.query.filter((User.username == form_username) | (User.email == form_email)).first()
        if existing_user:
            # Re-render the form and pass a helpful error alert to the user
            return render_template("register.html", error="Username or email is already registered.")
            
        # 3. SECURE HASHING: Scramble the password!
        hashed_password = generate_password_hash(form_password)
        
        # 4. Create the new user object using the HASHED password
        new_user = User(username=form_username, email=form_email, password_hash=hashed_password)
        
        # 5. Save to the database
        db.session.add(new_user)
        db.session.commit()
        
        print(f"Account created successfully for: {form_username}")
        # Send the user back to the homepage
        return redirect(url_for("index"))
        
    # If request.method is GET, simply display the blank registration form
    return render_template("register.html")
```

> **Stop and Think:** Look closely at **Step 3 and 4** in the code above. Notice how we pass `form_password` into `generate_password_hash()`, and then we save `hashed_password` into our database class. If you accidentally wrote `password_hash=form_password`, you would be storing plain text passwords again! Always double-check your hashing logic.

---

## Step 6: Test and Verify!

Let's prove our registration system works securely.

1. **Delete any old database file:** If you ran `app.py` before adding the `User` class, delete the `campus.db` file from your project explorer so SQLAlchemy can recreate it with both tables.
2. **Run your server:** Run `app.py`.
3. **Open your browser:** Visit `http://127.0.0.1:5000/`. Click the **Register** link in the navigation bar.
4. **Create an account:** Type in a test username, email, and password (e.g., `supersecret123`), then submit.
5. **Verify the Hash:** Let's look inside the actual database to confirm the password is encrypted!
   - Make sure you have the VSCode extension ** SQLite3 Editor** installed, open `campus.db` and browse the `user` table. Look at the `password_hash` column.
   - Alternatively, add this quick query check at the bottom of your `app.py` script (inside `app.app_context()`) to print the stored string to your terminal:
     ```python
     # Quick check to view the encrypted hash:
     first_user = User.query.first()
     if first_user:
         print(f"Stored Hash for {first_user.username}: {first_user.password_hash}")
     ```
   *(Notice how the stored text looks absolutely nothing like your original password!)*

6. **Test Duplicate Prevention:** Try registering a second account using the exact same email or username. Verify that the form reloads nicely and displays your red alert warning instead of crashing your Python server.

**Awesome job!** You have securely captured user data and protected their passwords. In **Exercise 2**, we will learn how to verify these hashes so users can actually log into their accounts!
