# Student Guide: Introduction to Flask-SQLAlchemy

Welcome! In our previous exercises, we learned how to write raw SQL commands to manage data. But as Python developers, writing SQL strings directly in our Python code can get messy. 

Enter **Flask-SQLAlchemy**, an Object-Relational Mapper (ORM). This tool lets us interact with our database using Python classes and objects instead of raw SQL strings. It translates Python into SQL for us!

In this exercise, we will build the database layer for a **Module Review** application.

---

## 1. Setup the Application

First, we need to install the library, import our tools, initialize our Flask app, and tell it where our database file will be stored.

Before writing code, open your terminal and install `flask-sqlalchemy`:

**Windows:**
`pip install flask-sqlalchemy`

**Mac:**
`pip3 install flask-sqlalchemy`

Next, create a new file called `app.py` and copy the following setup code:

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
# We are telling Flask to use a local SQLite database named 'reviews.db'
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///reviews.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

# Initialize the SQLAlchemy object
db = SQLAlchemy(app)
```

## 2. Define the Models

In SQLAlchemy, a **Model** is a Python class that represents a table in your database. Let's create our three tables: `Student`, `Module`, and `Review`.

Paste these classes below your setup code:

```python
class Student(db.Model):
    id = db.Column(db.Integer, primary_key=True) # e.g., 1234567
    first_name = db.Column(db.String(50), nullable=False)
    surname = db.Column(db.String(50), nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)

class Module(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    code = db.Column(db.String(10), unique=True, nullable=False) # e.g., BUS01-1
    name = db.Column(db.String(100), nullable=False)
    year = db.Column(db.String(5), nullable=False) # e.g., 23/24

class Review(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    student_id = db.Column(db.Integer, db.ForeignKey('student.id'), nullable=False)
    module_id = db.Column(db.Integer, db.ForeignKey('module.id'), nullable=False)
    score_teaching = db.Column(db.Integer, nullable=False) # 1-10
    score_content = db.Column(db.Integer, nullable=False) # 1-10
    score_workload = db.Column(db.Integer, nullable=False) # 1-10
    review_text = db.Column(db.Text, nullable=True)
```

*(Notice how we use `db.ForeignKey` to link the review to a specific student and module, just like in our relational diagrams!)*

## 3. Initialize the Database

Our classes define the *structure*, but we need to actually create the `reviews.db` file and its tables.

At the bottom of your file, add this code to create the tables when the app runs:

```python
with app.app_context():
    db.create_all()
    print("Database tables created successfully!")
```

Run your `app.py` script once. You should see "Database tables created successfully!" in your terminal, and a new file `reviews.db` will appear in your project folder.

---

## 4. Insert Data (Create)

Now let's add some data. With SQLAlchemy, we create Python objects and add them to our database **session**. Think of the session as a "staging area" for your changes. Nothing is permanently saved until you `commit()` the session.

Add this inside your `app.app_context()` block (after `db.create_all()`):

```python
    # 1. Create a Student
    student1 = Student(id=1234567, first_name='Jane', surname='Doe', email='jane.doe@student.ac.uk')
    
    # 2. Create a Module
    module1 = Module(code='BUS01-1', name='Introduction to Business', year='25/26')
    
    # 3. Add to session
    db.session.add(student1)
    db.session.add(module1)
    
    # 4. Save to the database
    db.session.commit()
    print("Student and Module added!")
```

Run your script again.

> **Wait, I got an error!** If you run it a second time, you will get an `IntegrityError` because the student ID `1234567` already exists! This is our database protecting our data rules. Let's comment out the insertion code or add new IDs for subsequent runs.

Now let's add a review by adding this code (and making sure your previous insert code is commented out or only run once!):

```python
    # Let's add a review for our existing student and module
    review1 = Review(
        student_id=1234567,
        module_id=1, # We assume the module was the first one added and got ID 1
        score_teaching=9,
        score_content=8,
        score_workload=5,
        review_text="Great module, but lots of reading."
    )
    db.session.add(review1)
    db.session.commit()
    print("Review added!")
```
*(Run your script again to insert the review!)*

## 5. Query Data (Read)

How do we retrieve the data we just saved? SQLAlchemy makes queries easy. 

*Remember to comment out your INSERT code from above so it doesn't crash, then add this:*

```python
    # Find a module by its code
    bus01 = Module.query.filter_by(code='BUS01-1').first()
    print(f"Found Module: {bus01.name}")

    # Find all reviews for a specific module ID
    module_reviews = Review.query.filter_by(module_id=bus01.id).all()
    print(f"There are {len(module_reviews)} reviews for this module.")

    # Find out who wrote the first review
    first_review = module_reviews[0]
    reviewer = Student.query.get(first_review.student_id)
    print(f"Review written by: {reviewer.first_name} {reviewer.surname}")
```

*(Run this and verify the terminal output!)*

## 6. Update Data

Updating data involves querying the object, changing its Python attributes, and committing the session.

```python
    # Oh no, Jane meant to give a 10 for teaching!
    jane_review = Review.query.filter_by(student_id=1234567).first()
    
    print(f"Old Teaching Score: {jane_review.score_teaching}")
    
    # Change the value in Python
    jane_review.score_teaching = 10
    
    # Save the changes to the database
    db.session.commit()
    print(f"New Teaching Score: {jane_review.score_teaching}")
```

*(Run your script again to verify the update!)*

## 7. Delete Data

Deleting data requires passing the object to `db.session.delete()`.

```python
    # Let's say we need to remove that review
    review_to_delete = Review.query.filter_by(student_id=1234567).first()
    
    db.session.delete(review_to_delete)
    db.session.commit()
    
    print("Review deleted successfully.")
    
    # Verify it is gone
    check_review = Review.query.filter_by(student_id=1234567).first()
    if check_review is None:
        print("Confirmed: The review is gone.")
```

*(Run your script one last time!)*

**Congratulations!** You've just performed Create, Read, Update, and Delete operations using an ORM. This is exactly how we will manage data in our final Flask web applications, replacing hardcoded Python dictionaries with persistent, professional databases!
