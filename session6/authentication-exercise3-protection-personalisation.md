# Student Guide: Route Protection, Flash Messages, & personalised Content

Welcome to **Session 6, Exercise 3**—the final exercise of our authentication module!

In Exercise 2, we built full stateful sessions so users can log in and out. However, our application still has two **critical flaws**:
1. **Unprotected Pages:** Even if you aren't logged in, you could technically type a secret URL into your browser address bar and view it directly!
2. **Anonymous Records:** When a note is created, it isn't linked to the logged-in user. Every record is completely anonymous.

In this exercise, we will implement **Route Protection** to block unauthenticated visitors, introduce **Flash Messages** to give users visual feedback, and **personalise Data** by associating notes with their creator.

---

## Step 1: Open the Starter Files

This exercise continues on from where you left off in Exercise 2, you may wish to copy your `app.py1 and your templates/static folder into a new project before beginning this exercise.

---

## Step 2: Import Flash and Update Database Models

Open `app.py`.

1. At the very top of your file, update your imports to include `flash` from the core `flask` library:

```python
from flask import Flask, render_template, request, redirect, url_for, session, flash # <--- ADD flash
```

2. We need to update our database models so that every `Note` belongs to a specific `User`. We use a **Foreign Key** just like we learned in our database modeling sessions.

Find your `User` and `Note` models and replace them with this updated code:

```python
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(50), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password_hash = db.Column(db.String(256), nullable=False)
    
    # Optional helpful relationship shortcut: 
    # Allows us to easily access a user's notes using user.notes, or a note's author using note.author
    notes = db.relationship('Note', backref='author', lazy=True)

class Note(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    content = db.Column(db.Text, nullable=False)
    
    # ADDED IN EXERCISE 3: Link every note to its creator's User ID!
    author_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
```
3. **Critical Concept: Updating Sample Data Seeding**
Because `author_id` is now a mandatory, non-nullable column, any dummy/sample notes we create when initializing the database **must** belong to a valid registered user! If we try to create sample notes without specifying an `author_id`, SQLAlchemy will block it and crash with an `IntegrityError`.

To fix this, let's update the database initialisation code at the very bottom of `app.py` to first create a sample admin teacher account, and then link our initial sample notes to that teacher's ID!

Find your `if __name__ == "__main__":` block at the bottom of `app.py` and replace it entirely with this updated code:

```python
# --- Database Initialization ---

if __name__ == "__main__":
    with app.app_context():
        db.create_all()
        
        # Seed dummy admin user and notes if empty to make UI look complete immediately
        if User.query.count() == 0:
            # Create a secure hash for our default teacher user
            dummy_hash = generate_password_hash("teacher123")
            admin_user = User(username="admin_teacher", email="teacher@university.ac.uk", password_hash=dummy_hash)
            db.session.add(admin_user)
            db.session.commit() # Commit immediately so the database generates admin_user.id!
            
            # Now seed sample notes linked directly to the admin_user's ID
            sample1 = Note(content="Did you hear? Trinity caught a bug in the tutorial, she is wicked smart!!!", author_id=admin_user.id)
            sample2 = Note(content="James has been particularly grumpy this morning :(.", author_id=admin_user.id)
            db.session.add_all([sample1, sample2])
            db.session.commit()
            print("Database initialized and sample user/notes seeded!")
            
    app.run(debug=True)
```
---

## Step 3: Implement Flash Alert Rendering in Base Template

A **Flash Message** is a one-time pop-up notification used to inform users of success or failure (e.g., *"Welcome back!"* or *"Access Denied"*). 

Open `templates/base.html`. Find the `<main class="main-container">` area and add the following code to retrieve and display flashed messages nicely:

```html
<!-- Main Content Area -->
<main class="main-container">
    <!-- RENDER FLASH MESSAGES INTRODUCED IN EXERCISE 3 -->
    <div class="flash-container">
        {% with messages = get_flashed_messages(with_categories=true) %}
            {% if messages %}
                {% for category, message in messages %}
                    <!-- Render success alerts in green, and danger alerts in red -->
                    <div class="flash-alert {% if category == 'success' %}flash-alert-success{% elif category == 'danger' %}flash-alert-danger{% endif %}">
                        <span>{{ message }}</span>
                    </div>
                {% endfor %}
            {% endif %}
        {% endwith %}
    </div>
    
    {% block content %}
    {% endblock %}
</main>
```

---

## Step 4: Adding Flash Messages to Login and Logout

Let's make our user experience professional by giving instant visual feedback.

Open `app.py`. Find your `login()` and `logout()` routes and add `flash()` commands right before returning redirects:

**Inside `login()` upon successful verification:**
```python
        if user and check_password_hash(user.password_hash, form_password):
            session['user_id'] = user.id
            session['username'] = user.username
            
            # Add this visual alert feedback!
            flash(f"Welcome back, {user.username}!", "success")
            return redirect(url_for("index"))
```

**Inside `logout()`:**
```python
@app.route("/logout")
def logout():
    session.pop('user_id', None)
    session.pop('username', None)
    
    # Add this visual alert feedback!
    flash("You have been logged out successfully.", "success")
    return redirect(url_for("index"))
```

*(You can also add `flash("Account created successfully! Please log in.", "success")` inside your `/register` route!)*

---

## Step 5: Route Protection — Secure Page to Post Notes

Now let's build a secure web page where logged-in users can write a secret note. If an unauthenticated guest tries to visit this URL, we will block them and redirect them to the login page!

1. Inside `templates/`, create a new file named `new_note.html` and add this code:

```html
{% extends "base.html" %}

{% block title %}Post Note - Campus Confessions{% endblock %}

{% block content %}
<div class="card auth-card" style="max-width: 600px;">
    <h1 style="color: var(--primary-color); margin-bottom: 0.5rem;">Post a Secret Note</h1>
    <p style="color: var(--text-muted); margin-bottom: 2rem; font-size: 0.95rem;">
        Posting securely as: <span style="font-weight: 600; color: var(--text-color);">{{ session['username'] }}</span>
    </p>

    <form method="POST" action="{{ url_for('new_note') }}">
        <div class="form-group">
            <label for="content">What's on your mind?</label>
            <textarea id="content" name="content" rows="5" placeholder="Share a confession, study strategy, or campus advice..." required></textarea>
        </div>

        <button type="submit" class="btn">Publish Securely</button>
    </form>
</div>
{% endblock %}
```

2. Open `app.py` and implement the protected route function below your other routes:

```python
@app.route("/note/new", methods=["GET", "POST"])
def new_note():
    # 1. ROUTE PROTECTION BOUNDARY CHECK
    # If the user's browser has no logged-in session key, deny access immediately!
    if 'user_id' not in session:
        flash("Access Denied: Please log in to post secret notes.", "danger")
        return redirect(url_for("login"))
        
    if request.method == "POST":
        note_content = request.form["content"]
        
        # 2. DATA PERSONALISATION
        # Read the logged-in user's ID from the session dictionary and assign it as the foreign key!
        current_user_id = session['user_id']
        new_secret_note = Note(content=note_content, author_id=current_user_id)
        
        db.session.add(new_secret_note)
        db.session.commit()
        
        flash("Your secret note was published successfully!", "success")
        return redirect(url_for("index"))
        
    return render_template("new_note.html")
```

3. Open `templates/index.html`. Link the **"+ Post a Secret Note"** button to our new secure route:
```html
<!-- Change the placeholder href="#" to use url_for: -->
<a href="{{ url_for('new_note') }}" class="btn" style="width: auto;">+ Post a Secret Note</a>
```

Also, update the recent notes section in `index.html` to display the real username of the author dynamically using our SQLAlchemy relationship:
```html
<div class="note-header">
    <span class="badge">#{{ note.id }}</span>
    <!-- Display the author's username dynamically! -->
    <span style="font-weight: 500; color: var(--primary-color);">Posted by {{ note.author.username }}</span>
</div>
```

---

## Step 6: Querying personalised Records — User Dashboard

Finally, let's create a personalised "My Dashboard" page that displays **only** the notes published by the currently logged-in user.

1. Create `templates/dashboard.html`:

```html
{% extends "base.html" %}

{% block title %}My Dashboard - Campus Confessions{% endblock %}

{% block content %}
<div style="display: flex; justify-content: space-between; align-items: center; border-bottom: 2px solid var(--border-color); padding-bottom: 1rem; margin-bottom: 2rem;">
    <div>
        <h1 style="color: var(--primary-color);">My Secret Notes</h1>
        <p style="color: var(--text-muted); font-size: 0.95rem;">Manage the secrets you've contributed to the campus.</p>
    </div>
    <a href="{{ url_for('new_note') }}" class="btn" style="width: auto;">+ Post a Secret Note</a>
</div>

<div class="notes-grid">
    {% for note in user_notes %}
    <div class="note-card" style="border-left: 4px solid var(--primary-color);">
        <div>
            <div class="note-header">
                <span class="badge">#{{ note.id }}</span>
                <span style="color: var(--success-color); font-weight: 500;">✓ Your Post</span>
            </div>
            <p class="note-content">{{ note.content }}</p>
        </div>
    </div>
    {% else %}
    <div class="card" style="grid-column: 1 / -1; text-align: center; color: var(--text-muted);">
        <p>You haven't posted any notes yet. Click the button above to publish your first one!</p>
    </div>
    {% endfor %}
</div>
{% endblock %}
```

2. Add the Dashboard Route in `app.py`:

```python
@app.route("/dashboard")
def dashboard():
    # 1. Route Protection
    if 'user_id' not in session:
        flash("Access Denied: Please log in to view your dashboard.", "danger")
        return redirect(url_for("login"))
        
    # 2. Query filtering by Author ID
    current_user_id = session['user_id']
    my_notes = Note.query.filter_by(author_id=current_user_id).all()
    
    return render_template("dashboard.html", user_notes=my_notes)
```

3. Open `templates/base.html` and add the link to `My Dashboard` inside the authenticated nav section:
```html
{% if 'user_id' in session %}
    <!-- Add this dashboard link: -->
    <a href="{{ url_for('dashboard') }}" class="nav-link" style="color: var(--primary-color); font-weight: 600;">My Dashboard</a>
```

---

## Step 7: Complete Verification Checklist

Let's test our secure web application end-to-end!

1. **Delete Old Database:** Because we added `author_id` to the `Note` table, delete `campus.db` from your file explorer to prevent schema mismatch errors, then run `python app.py`.
2. **Test Route Protection (Unauthorised):** Open your browser as a guest (logged out). Try typing `http://127.0.0.1:5000/dashboard` or `http://127.0.0.1:5000/note/new` directly into the address bar. Verify that you are instantly redirected to the login page with a red alert saying *"Access Denied"*.
3. **Register & Log in:** Create an account and log in. Observe your green welcome alert message.
4. **Post personalised Data:** Click **"+ Post a Secret Note"**. Publish a note. Verify that it appears on the homepage displaying your real username.
5. **View Dashboard:** Click **My Dashboard** in the navbar. Verify that your published notes are listed there.
6. **Multi-User Isolation:** Register a second account. Log in. Check the Dashboard... It should be completely empty for the new user! Only the notes created by the active session ID are fetched.

Well done, you have now built a fully authenticated, stateful, personalised, and robust web application. You now should be able to implement the account registration and login functionality for your bubble tea app, and able to have the personalsied order history for your users.
