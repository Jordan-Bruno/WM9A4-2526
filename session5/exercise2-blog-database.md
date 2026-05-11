# Student Guide: Upgrading the Blog to Use a Database

Welcome to Exercise 2! If you can remember last week in Session 3, you built a fully functional blog where you could write posts and view them. 

However, you probably noticed a **major flaw**: whenever you stopped your Flask server or restarted your computer, all your blog posts disappeared!

### Why did that happen?
Look at your `app.py` file. Near the top, you have this line:
```python
posts = []
```
This is a standard Python list. It stores your posts in your computer's temporary memory (RAM). When the Python program closes, that memory is wiped clean. 

To fix this, we need to replace our temporary list with a **Database** using Flask-SQLAlchemy!

---

## Step 1: Set up the Database Connection

First, we need to install the `flask-sqlalchemy` library and connect it to our Flask app.

1. Open your terminal and install the package:
   - **Windows:** `pip install flask-sqlalchemy`
   - **Mac:** `pip3 install flask-sqlalchemy`
2. Open your `app.py` file.
3. At the top of your file, add the SQLAlchemy import:
   ```python
   from flask import Flask, render_template, request, redirect, url_for
   from flask_sqlalchemy import SQLAlchemy  # <--- ADD THIS LINE
   ```
3. Below your `app = Flask(__name__)` line, configure the database:
   ```python
   app = Flask(__name__)
   
   # Add these three lines to configure the database!
   app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///blog.db'
   app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
   db = SQLAlchemy(app)
   
   # DELETE this line, we don't need it anymore!
   # posts = [] 
   ```

## Step 2: Define the `Post` Model

Now we need to tell SQLAlchemy what a blog post looks like so it can create a database table for it.

Below your database setup, define a `Post` class:

```python
class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100), nullable=False)
    content = db.Column(db.Text, nullable=False)
```
*This means every post will have a unique ID, a title (maximum 100 characters), and some text content.*

## Step 3: Initialize the Database

We need to create the actual `blog.db` file. 

Scroll to the very bottom of your `app.py` file and add the database creation code right before `app.run()`:

```python
if __name__ == "__main__":
    # Add this block to create the database tables
    with app.app_context():
        db.create_all()
        print("Database initialized!")
        
    app.run(debug=True)
```
*Run your `app.py` file now. You should see a new file called `blog.db` appear in your project folder!*

---

## Step 4: Saving Posts to the Database

Right now, your `/new` route tries to append data to the old `posts` list (which we deleted). Let's fix that.

Find your `new_post()` function and update the `POST` section to use the database instead:

**Change this old code:**
```python
        # OLD CODE:
        posts.append({"title": title, "content": content})
```

**To this new code:**
```python
        # NEW CODE:
        # Create a new Post object
        new_blog_post = Post(title=title, content=content)
        
        # Add it to the database session
        db.session.add(new_blog_post)
        
        # Commit (save) the changes
        db.session.commit()
```

## Step 5: Viewing All Posts

Now that posts are saved in the database, we need to fetch them when the user visits the homepage.

Find your `index()` function and change it to query the database:

**Change this:**
```python
@app.route("/")
def index():
    return render_template("index.html", posts=posts)
```

**To this:**
```python
@app.route("/")
def index():
    # Fetch all posts from the database!
    all_posts = Post.query.all()
    return render_template("index.html", posts=all_posts)
```

### Important Template Fix!
Because we are no longer using a simple list, we can't use the list's index numbers for our links. Our posts now have real Database IDs!

Open `templates/index.html`. Find the link that looks like this:
`<a href="{{ url_for('post', post_id=loop.index0) }}">`

Change `loop.index0` to `post.id`:
`<a href="{{ url_for('post', post_id=post.id) }}">`

## Step 6: Viewing a Single Post

Finally, when a user clicks on a post, we need to fetch that specific post from the database using its ID.

Find your `post(post_id)` function. We are going to completely rewrite it. 

**Replace the entire `post` function with this:**
```python
@app.route("/post/<int:post_id>")
def post(post_id):
    # Ask the database to find the Post with this specific ID.
    # If it doesn't exist, it will automatically show a 404 Error page!
    selected_post = Post.query.get_or_404(post_id)
    
    return render_template("post.html", post=selected_post)
```

---

## Test Your App!

1. Stop your Flask server if it is running, and start it again.
2. Go to your website and create a few blog posts.
3. Stop your server completely (Ctrl+C).
4. Start your server again.
5. Go to your website... **Your posts are still there!**

Congratulations! You have successfully upgraded a temporary web app into a robust, database-driven application!
