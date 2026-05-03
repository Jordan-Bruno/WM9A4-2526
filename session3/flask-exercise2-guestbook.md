# Building a Guestbook

Welcome! In this exercise, you are going to learn how to capture information that a user types into a form and send it to your Python code. By the end, you'll have a working guestbook!

---

## Step 1: Setting up your folders

As always, Flask expects your project to be organised in a very specific way. 
1. Open your code editor and create a new folder for your project.
2. Inside that project folder, create **one new file** and **two new folders**:
   - Create a file named `app.py` (This is where all your Python code will go).
   - Create a folder named `templates` (This is where all your HTML files will go).
   - Create a folder named `static` (This is where your CSS styling will go).

---

## Step 2: The Magic "Skeleton"

Let's write the absolute minimum code needed to start our web app.

1. Open `app.py` and write the following code:

```python
# Import the tools we need from Flask
from flask import Flask, render_template

# Create our web application
app = Flask(__name__)

# This is our home page route
@app.route("/")
def home():
    return render_template("index.html")

# This turns on the web server!
if __name__ == "__main__":
    app.run(debug=True)
```

> **Hint:** Keep your server running in the background (`python app.py`) as you work. If you visit your page now, you will get an error because we haven't created `index.html` yet! Let's do that next.

---

## Step 3: Writing the Form (HTML)

We need a page where users can type their names and messages.

1. Inside your **`templates`** folder, create a new file called `index.html`.
2. Add the following HTML code:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Guestbook</title>
  </head>
  <body>
    <main class="container">
      <h1>Sign Our Guestbook</h1>

      <!-- The form uses method="POST" to securely send data to Python -->
      <form method="POST">
        <label>Name:</label>
        <input type="text" name="name" required />

        <label>Message:</label>
        <textarea name="message" required></textarea>

        <button type="submit">Submit</button>
      </form>
    </main>
  </body>
</html>
```

> **Hint:** Pay close attention to `name="name"` inside the `<input>` tag and `name="message"` inside the `<textarea>`. These names act like labels on a box. Python will use these exact labels to open the box and get the data!

---

## Step 4: Catching the Data (Python)

Right now, if you click the submit button, nothing happens. We need to tell Python to catch the data that the form just sent!

1. Open `app.py`. Update the very first line so we import a new tool called `request`:
```python
from flask import Flask, render_template, request
```

2. Update your `@app.route` section so it looks exactly like this:

```python
# We have to add methods=["GET", "POST"] to allow the form to submit data here!
@app.route("/", methods=["GET", "POST"])
def home():
    # Did the user just click the submit button? (This is a POST request)
    if request.method == "POST":
        # Get the text they typed into the 'name' box
        name = request.form.get("name")
        # Get the text they typed into the 'message' box
        message = request.form.get("message")
        
        # Send them to a new page, and pass those variables to the HTML!
        return render_template("response.html", name=name, message=message)
    
    # If they are just visiting normally (a GET request), show the empty form
    return render_template("index.html")
```

---

## Step 5: The "Thank You" Page (HTML)

Python is now catching the data and trying to send it to `response.html`. Let's create that page.

1. Inside your **`templates`** folder, create a new file called `response.html`.
2. Add this code:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Guestbook</title>
  </head>
  <body>
    <main class="container">
      <!-- We use Jinja to print the variables we caught in Python! -->
      <h1>Thank You, {{ name | title }}!</h1>
      
      <p>Your message:</p>
      <blockquote>{{ message }}</blockquote>
      
      <a href="/" class="button">Sign Again</a>
    </main>
  </body>
</html>
```

> **Hint:** Remember that `{{ }}` tells the HTML to expect a Python variable! We also used `| title` next to the name variable. This is a neat trick that automatically capitalises the first letter of their name!

Try it out! You should be able to type a message and see it on the next screen.

---

## Step 6: Make it Beautiful! (CSS)

Your guestbook works, but it needs some style.

1. Inside your **`static`** folder, create a new file called `style.css`.
2. Copy the exercise 2 css code on Moodle and paste it into this file.
3. Finally, you need to tell your HTML files to use this new style. Open **both** `index.html` and `response.html`. 
4. Look for the `<head>` section in both files and add this line inside it:

```html
<link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
```

Refresh your page. Congratulations, you've built a fully functional Guestbook!
