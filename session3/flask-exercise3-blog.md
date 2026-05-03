# Building Your Own Web Blog

Welcome! In this exercise, you are going to build a fully working blog from scratch using Python, Flask, and HTML. By the end of this tutorial, you will have a website where you can create new blog posts and read them.

---

## Step 1: Setting up your folders

Flask expects your project to be organised in a very specific way. 
1. Open your code editor and create a new folder for your project.
2. Inside that project folder, create **one new file** and **two new folders**:
   - Create a file named `app.py` (This is where all your Python code will go).
   - Create a folder named `templates` (This is where all your HTML files will go).
   - Create a folder named `static` (This is where your CSS styling will go).

> **Hint:** Make sure `templates` is spelled correctly (all lowercase!). If it's misspelled, Flask won't be able to find your web pages.

---

## Step 2: The Magic "Skeleton"

Let's write the absolute minimum code needed to start a web server.

1. Open `app.py` and write the following code:

```python
from flask import Flask

# This creates our web application
app = Flask(__name__)

# This checks if we are running this exact file
if __name__ == "__main__":
    # This turns on the web server!
    app.run(debug=True)
```

> **Hint:** Notice that `__name__` and `__main__` have **two** underscores on the left, and **two** underscores on the right! 
> 
> You can now run this file using your terminal (`python app.py`). If you go to the address it gives you (usually `http://127.0.0.1:5000`), you will see a "Not Found" error. This is normal because we haven't built any pages yet! Keep the server running in the background as you work.

---

## Step 3: Our Fake Database and the Home Page

We need a place to store our blog posts. Since we aren't using a real database today, we will just use a Python **list**.

1. Update your `app.py` file to look like this:

```python
from flask import Flask, render_template

app = Flask(__name__)

# This empty list is our fake database!
posts = []

# This tells Flask: "When someone visits the home page (/), run this function"
@app.route("/")
def index():
    # This sends our list of posts to a file called index.html
    return render_template("index.html", posts=posts)

if __name__ == "__main__":
    app.run(debug=True)
```

---

## Step 4: Writing the Home Page (HTML)

Flask is looking for a file called `index.html`. Let's create it!

1. Inside your **`templates`** folder, create a new file called `index.html`.
2. Add the following HTML code:

```html
<!DOCTYPE html>
<html>
<head>
    <title>My Awesome Blog</title>
</head>
<body>
    <main class="container">
        <h1>My Blog Posts</h1>
        
        <!-- We will add a link to create posts here later -->

        <hr>

        <!-- This is a Jinja Loop. It runs through our Python list! -->
        {% for post in posts %}
        <div class="post-card">
            <h2>{{ post.title }}</h2>
        </div>
        {% else %}
        <p>No posts yet! You should write one.</p>
        {% endfor %}

    </main>
</body>
</html>
```

> **Hint:** `{% %}` and `{{ }}` are special tags that let us mix Python directly into our HTML! Remember to use `{% endfor %}` to tell the loop where to stop.

Go to your browser and refresh the page. You should now see "My Blog Posts" and "No posts yet!".

---

## Step 5: How to Create a New Post (Python)

We need a new webpage where we can type out a blog post.

1. Open `app.py`. First, we need to import a few more tools at the very top. Update line 1 so it looks exactly like this:
```python
from flask import Flask, render_template, request, redirect, url_for
```

2. Now, add this new route **above** the `if __name__ == "__main__":` line. Make sure your indentation (spacing) matches exactly:

```python
@app.route("/new", methods=["GET", "POST"])
def new_post():
    # Did the user click the submit button? (POST request)
    if request.method == "POST":
        # Get the text they typed into the form
        title = request.form["title"]
        content = request.form["content"]

        # Save it to our Python list as a Dictionary
        posts.append({"title": title, "content": content})

        # Send them back to the home page
        return redirect(url_for("index"))

    # If they didn't click submit, just show them the empty form
    return render_template("new_post.html")
```

---

## Step 6: Creating the Form (HTML)

Now we need the HTML file with the form that the user can type into.

1. Inside your **`templates`** folder, create a new file called `new_post.html`.
2. Add this code:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Write a Post</title>
</head>
<body>
    <main class="container">
        <h1>Write a New Post</h1>

        <!-- The form uses method="POST" to securely send data to Python -->
        <form method="POST">
            <label for="title">Title of your post:</label>
            <input type="text" id="title" name="title" required>

            <label for="content">What do you want to say?</label>
            <textarea id="content" name="content" required></textarea>

            <button type="submit" class="button">Publish Post</button>
        </form>

        <hr>
        
        <!-- A handy link to go back -->
        <a href="{{ url_for('index') }}">Go Back Home</a>
    </main>
</body>
</html>
```

> **Hint:** Pay close attention to `name="title"` and `name="content"`. These names MUST match the names we used in `app.py` when we wrote `request.form["title"]`.

### Connecting the Pages
Right now, there is no way to click from the Home Page to the New Post page. Let's fix that.
1. Open `templates/index.html`
2. Underneath the `<h1>My Blog Posts</h1>`, add this link:
```html
<a href="{{ url_for('new_post') }}" class="button">Create a New Post</a>
```

Try it out in your browser! Click the link, type a post, and hit submit. It should appear on your home page!

---

## Step 7: Viewing a Single Post

We can see the titles on the home page, but we can't click them to read the full story. Let's fix that!

1. Open `app.py`.
2. Add this route just above your `new_post` route:

```python
# The <int:post_id> tells Flask to expect a number in the URL!
@app.route("/post/<int:post_id>")
def post(post_id):
    # Check if the post number actually exists in our list
    if post_id >= 0 and post_id < len(posts):
        # Grab that specific post
        selected_post = posts[post_id]
        # Show it on a new page
        return render_template("post.html", post=selected_post)
    
    # If the post doesn't exist, send them home
    return redirect(url_for("index"))
```

---

## Step 8: The Single Post Webpage (HTML)

1. Inside your **`templates`** folder, create a new file called `post.html`.
2. Add this code:

```html
<!DOCTYPE html>
<html>
<head>
    <title>{{ post.title }}</title>
</head>
<body>
    <main class="container">
        <!-- We use double curly brackets to display the Python variables! -->
        <h1>{{ post.title }}</h1>
        <hr>

        <p>{{ post.content }}</p>

        <hr>
        <a href="{{ url_for('index') }}" class="button">Go Back Home</a>
    </main>
</body>
</html>
```

### Making the Titles Clickable
The last step for functionality is making the titles on the home page clickable links.
1. Open `templates/index.html`.
2. Find the line that says `<h2>{{ post.title }}</h2>` and change it to look like this:

```html
<h2>
    <!-- loop.index0 gives us the number of the post (0, 1, 2...) -->
    <a href="{{ url_for('post', post_id=loop.index0) }}">
        {{ post.title }}
    </a>
</h2>
```

---

## Step 9: Make it Beautiful! (CSS)

Your blog works perfectly, but it looks a bit plain. Let's add some styling!

1. Inside your **`static`** folder, create a new file called `style.css`.
2. Copy the exercise 3 css code on Moodle and paste it into this file.
3. Finally, you need to tell your HTML files to use this new style. Open `index.html`, `new_post.html`, and `post.html`. 
4. In **all three files**, look for the `<head>` section and add this line inside it:

```html
<link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
```

Refresh your page. Congratulations, you've built a fully functional, beautiful web application!
