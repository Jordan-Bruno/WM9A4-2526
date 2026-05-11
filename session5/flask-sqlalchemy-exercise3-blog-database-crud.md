# Student Guide: Building the Admin Panel (Update & Delete)

Welcome to Exercise 3! In our previous exercises, we learned how to Create and Read data from our database. But what happens if you make a typo in a blog post? What if you want to remove an old post?

Right now, you have no way to **Update** or **Delete** posts! 

To fix this, we are going to build a simulated "Admin Panel". In the real world, this page would require a username and password, but for this exercise, we will just create a hidden `/admin` URL that we can visit to manage our content.

---

## Step 1: The Admin Dashboard

First, we need a page that lists all of our posts and gives us options to edit or delete them.

1. **Create a new template file** in your `templates` folder called `admin.html`.
2. Copy and paste this code into `admin.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Admin Panel</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
</head>
<body>
    <main class="container">
        <h1>Admin Dashboard</h1>
        <hr>
        
        {% for post in posts %}
        <div class="post-card" style="display: flex; justify-content: space-between; align-items: center;">
            <h2>{{ post.title }}</h2>
            
            <div style="display: flex; gap: 10px;">
                <!-- We will add the Edit link here in Step 3 -->
                <a href="#" class="button">Edit</a>
                
                <!-- We will add the Delete form here in Step 2 -->
                <button class="button button-danger">Delete</button>
            </div>
        </div>
        {% else %}
        <p>No posts to manage!</p>
        {% endfor %}
        
        <hr>
        <a href="{{ url_for('index') }}">Back to Public Homepage</a>
    </main>
</body>
</html>
```

3. **Add the Route:** Open `app.py` and add this route near the bottom (before `if __name__ == "__main__":`):

```python
@app.route("/admin")
def admin():
    # Fetch all posts to display in the dashboard
    all_posts = Post.query.all()
    return render_template("admin.html", posts=all_posts)
```

4. **Update the CSS:** To make our new "Delete" button look appropriately dangerous (red), we need to add a new CSS class. Open `static/style.css` and add this to the very bottom of the file:

```css
.button-danger {
    background-color: #ef4444;
}

.button-danger:hover {
    background-color: #dc2626;
}
```

*Run your app and visit `http://127.0.0.1:5000/admin` to see your new dashboard! Notice the red delete buttons.*

---

## Step 2: Deleting a Post

When deleting data, we must be careful. If we just used a standard link (`<a href="/delete/1">`), a search engine like Google could accidentally click every delete link on your page while scanning your site! 

To prevent this, destructive actions should always use a **POST** request via an HTML form.

1. **Add the Delete Route in `app.py`**:

```python
@app.route("/delete/<int:post_id>", methods=["POST"])
def delete_post(post_id):
    # 1. Find the exact post in the database
    post_to_delete = Post.query.get_or_404(post_id)
    
    # 2. Tell the database session to delete it
    db.session.delete(post_to_delete)
    
    # 3. Commit (Checkout) to save the changes!
    db.session.commit()
    
    # 4. Send the user back to the admin panel
    return redirect(url_for("admin"))
```

2. **Update the Template**: Open `admin.html`. Find the placeholder `<button class="button button-danger">Delete</button>` and replace it with an actual form:

```html
                <form action="{{ url_for('delete_post', post_id=post.id) }}" method="POST" style="margin: 0;">
                    <button type="submit" class="button button-danger">Delete</button>
                </form>
```
*Try it out! You should now be able to permanently delete posts.*

---

## Step 3: Editing a Post (Viewing the Form)

Editing data is a two-step process:
1. Show the user a form that is **pre-filled** with the old data (`GET` request).
2. Accept the new data and save it to the database (`POST` request).

Let's do the first part.

1. **Create `edit.html`** in your `templates` folder and add this code:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Edit Post</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
</head>
<body>
    <main class="container">
        <h1>Edit Post</h1>
        
        <form method="POST">
            <label for="title">Title:</label>
            <!-- Notice how we use the 'value' attribute to pre-fill the input box! -->
            <input type="text" id="title" name="title" value="{{ post.title }}" required>

            <label for="content">Content:</label>
            <!-- Textareas don't have a 'value' attribute. The text goes between the tags. -->
            <textarea id="content" name="content" required>{{ post.content }}</textarea>

            <button type="submit" class="button">Save Changes</button>
        </form>
        
        <hr>
        <a href="{{ url_for('admin') }}">Cancel</a>
    </main>
</body>
</html>
```

2. **Add the Edit Route in `app.py`**:

```python
@app.route("/edit/<int:post_id>", methods=["GET", "POST"])
def edit_post(post_id):
    # Fetch the post we want to edit
    post_to_edit = Post.query.get_or_404(post_id)
    
    # If the user just clicked the link, show them the pre-filled form
    if request.method == "GET":
        return render_template("edit.html", post=post_to_edit)
```

3. **Update `admin.html`**: Find the placeholder `<a href="#" class="button">Edit</a>` and link it to our new route:

```html
                <a href="{{ url_for('edit_post', post_id=post.id) }}" class="button">Edit</a>
```
*Click the Edit button in your admin panel. You should see your form pre-filled with the old text!*

---

## Step 4: Saving the Edits

Right now, if you click "Save Changes", nothing happens because we haven't written the `POST` logic.

Update your `edit_post` route in `app.py` to handle the `POST` request:

```python
@app.route("/edit/<int:post_id>", methods=["GET", "POST"])
def edit_post(post_id):
    post_to_edit = Post.query.get_or_404(post_id)
    
    if request.method == "POST":
        # 1. Grab the new text from the form
        new_title = request.form["title"]
        new_content = request.form["content"]
        
        # 2. Overwrite the old properties with the new ones
        post_to_edit.title = new_title
        post_to_edit.content = new_content
        
        # 3. Commit the changes to the database!
        db.session.commit()
        
        # 4. Redirect back to the admin panel
        return redirect(url_for("admin"))
        
    # GET request code stays the same
    return render_template("edit.html", post=post_to_edit)
```

## Test Your Complete App!

You did it! You have built a full CRUD application.
- **C**reate: `/new` route
- **R**ead: `/` and `/post/<id>` routes
- **U**pdate: `/edit/<id>` route
- **D**elete: `/delete/<id>` route

This is the exact lifecycle that powers most of the web applications in the world. Great job!
