# Flask Basics

Welcome to your first Flask app! During this session, the module tutor will walk you through how to build your first web application step-by-step. Keep this document open to follow along and copy the code as instructed.

---

## Step 1: Create a Basic Flask App

We start by building a tiny web server. This app will show "Hello, World!" when we visit the homepage.

**Action:** Create a new file called `app.py` and add this code:

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return 'Hello, World!'

if __name__ == '__main__':
    app.run(debug=True)
```

**What this does:** 
We import Flask, create our app, create one "route" (the main page `/`), and run the server. `debug=True` means the server will automatically update when you save your file!

---

## Step 2: Add Another Route

Most websites have many pages. Let's add an About page.

**Action:** Add this new route to your `app.py` file (above the `if __name__` line):

```python
@app.route('/about')
def about():
    return 'This is the about page.'
```

**What this does:** 
When you visit `/about` in your browser, this new page will load and show a simple message.

---

## Step 3: Return HTML from a Route

Instead of sending plain text, we can return proper HTML formatting.

**Action:** Add this code to your `app.py`:

```python
@app.route('/html')
def html_page():
    return '<h1>Hello with HTML!</h1><p>This is a paragraph with HTML tags.</p>'
```

**What this does:** 
The browser understands the HTML tags (`<h1>` and `<p>`) and formats the text properly as a heading and a paragraph.

---

## Step 4: Update Imports to Use Templates

Writing long HTML inside Python is messy. Flask lets us store HTML files separately using "templates". 

**Action:** Go to the very top of `app.py` and update your import line to include `render_template`:

```python
from flask import Flask, render_template
```

**What this does:** 
Adding `render_template` gives Python the ability to find and load real HTML files saved in a folder.

---

## Step 5: Create and Render an HTML File

Let's use a real HTML file for our homepage.

**Action 1:** Create a new folder named `templates`. 
**Action 2:** Inside the `templates` folder, create a file called `index.html` with this code:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Home Page</title>
</head>
<body>
    <h1>Welcome to the Home Page!</h1>
    <p>This page is from a real HTML file.</p>
</body>
</html>
```

**Action 3:** Go back to `app.py` and update your very first route (the `/` route) to look like this:

```python
@app.route('/')
def home():
    return render_template('index.html')
```

**What this does:** 
Flask now loads your real HTML file when someone visits the homepage. This separates your design from your logic!

---

## Step 6: Create a Dynamic URL

Sometimes the page should change based on what the user types in the address bar. 

**Action:** Add this new route to `app.py`:

```python
@app.route('/user/<name>')
def user(name):
    return f'Hello, {name}!'
```

**What this does:** 
If someone visits `/user/Jordan`, it will say "Hello, Jordan!". Flask automatically grabs the name from the URL and passes it to the Python function.

---

## Step 7: Set Variable Types in URLs

We can enforce rules on what users can type in the URL.

**Action:** Add this new route to `app.py`:

```python
@app.route('/post/<int:post_id>')
def show_post(post_id):
    return f'Post ID is {post_id}'
```

**What this does:** 
If you visit `/post/5`, it works. If you type text instead of a number, it will throw an error because `<int:` tells Flask to *only* accept whole numbers (integers).

---

## Step 8: Use a Template with Dynamic Content

Let's combine everything! We will use a dynamic URL variable and pass it directly into an HTML template.

**Action 1:** Create a new file inside the `templates` folder called `user.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>User Page</title>
</head>
<body>
    <h1>{{ name }} says hello!</h1>
</body>
</html>
```

**Action 2:** Update your `/user/<name>` route in `app.py` to render this template instead of returning a string:

```python
@app.route('/user/<name>')
def user(name):
    return render_template('user.html', name=name)
```

**What this does:** 
Inside the `user.html` page, `{{ name }}` is a special placeholder. Flask takes the name from the URL and replaces the placeholder with the real name. This uses a templating language called **Jinja2**, which you will use a lot in the future!
