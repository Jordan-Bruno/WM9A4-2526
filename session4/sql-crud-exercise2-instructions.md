# Student Guide: Introduction to SQL & CRUD (Walkthrough)

Welcome to your first step into managing business data! As a Bubble Tea shop owner, your menu needs to live somewhere safe and structured. That place is a Database.

## What is SQL?
**SQL** (Structured Query Language) is the standard language used to communicate with databases. You use it to tell the database exactly what to do. For this exercise, you are going to copy, paste, and run SQL commands to build your very own digital menu.

## The 4 Pillars of Data: CRUD
Every app you use relies on four fundamental operations, known as **CRUD**:
* **C**reate: Adding new data (e.g., adding a new bubble tea). In SQL, this is `INSERT`.
* **R**ead: Viewing data (e.g., looking at your menu). In SQL, this is `SELECT`.
* **U**pdate: Modifying existing data (e.g., changing a price). In SQL, this is `UPDATE`.
* **D**elete: Removing data (e.g., taking an old drink off the menu). In SQL, this is `DELETE`.

---

## Let's Build Your Menu!

Open your SQL Playground. For each step below, read the business scenario, copy the SQL code provided, paste it into your playground, and click "Run". Look at the results!

### Step 1: Set Up the Structure (Create)
**The Scenario:** Before we can add drinks, we need an empty spreadsheet-like structure to hold them. 
**The Action:** We will create a table named `menu_items`.

```sql
CREATE TABLE menu_items (
    id INTEGER PRIMARY KEY,
    name TEXT,
    price DECIMAL(5,2),
    category TEXT
);
```
*(Notice how we tell the database exactly what type of data each column will hold, like TEXT for words and DECIMAL for money).*

### Step 2: Add Your First Drink (Create)
**The Scenario:** Your shop is open! Let's add the signature drink.
**The Action:** We use `INSERT INTO` to add 'Classic Milk Tea'.

```sql
INSERT INTO menu_items (id, name, price, category) 
VALUES (1, 'Classic Milk Tea', 3.50, 'Milk Tea');
```

### Step 3: Check Your Work (Read)
**The Scenario:** You want to make sure the drink was actually saved.
**The Action:** We use `SELECT *` (which means "select everything") to view our table.

```sql
SELECT * FROM menu_items;
```
*(Run this and you should see a table with your one drink!)*

### Step 4: Batch Add More Drinks (Create)
**The Scenario:** Adding drinks one by one is slow. Let's add the rest of the menu at once.
**The Action:** We can provide multiple rows of `VALUES`.

```sql
INSERT INTO menu_items (id, name, price, category) VALUES 
(2, 'Taro Milk Tea', 4.00, 'Milk Tea'),
(3, 'Matcha Latte', 4.50, 'Milk Tea'),
(4, 'Mango Passion Fruit', 3.80, 'Fruit Tea');
```

### Step 5: View the Full Menu (Read)
**The Scenario:** Let's look at the expanded menu.
**The Action:** Run the read command again.

```sql
SELECT * FROM menu_items;
```

### Step 6: Filter for Premium Drinks (Read)
**The Scenario:** You want to see which drinks are your most expensive (costing more than £4.00).
**The Action:** We add a `WHERE` clause to our `SELECT` statement to filter the results.

```sql
SELECT * FROM menu_items 
WHERE price > 4.00;
```

### Step 7: Deal with Inflation (Update)
**The Scenario:** The cost of black tea leaves has gone up. You need to raise the price of the 'Classic Milk Tea' to £3.80.
**The Action:** We use `UPDATE`. **Important:** Notice the `WHERE` clause. If we didn't include `WHERE id = 1`, it would change the price of EVERY drink!

```sql
UPDATE menu_items 
SET price = 3.80 
WHERE id = 1; 
```

### Step 8: Re-categorize a Drink (Update)
**The Scenario:** You realise 'Matcha Latte' shouldn't just be 'Milk Tea', it should be in a 'Specialty' category.
**The Action:** Update the category for drink ID 3.

```sql
UPDATE menu_items 
SET category = 'Specialty' 
WHERE id = 3;
```

### Step 9: Remove a Discontinued Drink (Delete)
**The Scenario:** 'Taro Milk Tea' just isn't selling well, and the powder expired. You need to remove it completely.
**The Action:** We use `DELETE`. Again, the `WHERE` clause is critical so we don't accidentally delete the whole menu!

```sql
DELETE FROM menu_items 
WHERE id = 2; 
```

### Step 10: Final Verification (Read)
**The Scenario:** Let's look at your finalized, updated, and corrected menu.
**The Action:** Run one last check.

```sql
SELECT * FROM menu_items;
```

**Check your final table!**
* Is Taro gone? 
* Is Classic Milk Tea 3.80? 
* Is Matcha Latte a Specialty? 

**Congratulations!** You just controlled a database using SQL. You now understand how the software you use every day manages your data behind the scenes. Some of the above commands will be the ones you would use for the functions in your Bubble Tea application.
