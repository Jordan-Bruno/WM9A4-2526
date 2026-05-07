# Data Analyst Onboarding: AI-Assisted SQL Challenges

## Welcome to Pop Box Collectibles!
Congratulations on your new role as a Junior Data Analyst for **Pop Box Collectibles**, the hottest Designer Toy & Blind Box retailer! We track everything: which series are dropping, the rarity of the figures (who pulled the 'Secret' figures?), and how much our collectors are spending.

Your manager has given you access to our database. Your first task is to answer 20 business questions. 

It's ok that you do not know SQL yet, In the modern business world, problem-solving is more important than memorising code. In this module you are expected and encouraged to use AI tools (like ChatGPT, Claude, or Gemini) to help you write the SQL code to answer these questions (But to do so responsibly).

---

## How to Use AI Responsibly (and Actually Learn!)

If you just copy and paste the questions into AI and then copy the answers, you will learn absolutely nothing. When you eventually build your own Flask Web Application for your Bubble Tea shop, you will get stuck. Use these principles:

### 1. Give the AI Context (The Schema)
The AI doesn't know what your database looks like. Before asking a question, copy and paste the SQL code from `database.sql` into the AI so it knows what tables and columns exist.
> **Prompt Example:** *"I am working on a SQLite database. Here is my schema: [PASTE SQL HERE]. Act as a senior database engineer and help me write queries."*

### 2. Break Down the Problem
Don't ask the AI to do everything at once. If a task is hard, ask the AI to explain the logic first.
> **Prompt Example:** *"I need to find the customer who spent the most money. Before giving me the code, can you explain what SQL commands I will need to use to combine the customers table and the purchases table?"*

### 3. Ask for Explanations, Not Just Code
If the AI gives you a query with a command you don't understand (like `JOIN` or `GROUP BY`), ask it to explain that specific word.
> **Prompt Example:** *"Your code used `LEFT JOIN`. Can you explain what a Left Join does in simple terms, and why you used it instead of a regular Join?"*

### 4. Relate it to Your Future Web App
Think about how these queries relate to your Bubble Tea shop assignment. 
* *"How would I write a query to show a user all their past bubble tea orders?"* (This will use a `JOIN` and a `WHERE` clause, just like tasks 16-20 below!).

---

## The Setup
1. Go to [DB Fiddle](https://www.db-fiddle.com/) (or your chosen SQL playground).
2. Copy all the text from the `database.sql` file and paste it into the **Schema Panel** (usually on the left).
3. Write your answers in the **Query Panel** (usually on the right) and hit "Run".

---

## The 20 Challenges

### Level 1: Warming Up (Basic SELECTs)
1. Write a query to view everything in the `series` table.
2. List the `username` and `join_date` of all our `customers`.
3. Show all the figures, but only show their `name` and `rarity_level`.
4. We need to audit our purchases. Retrieve the entire `purchases` table.
5. List all figures that have a `rarity_level` of 'Secret'.

### Level 2: Filtering & Sorting (WHERE and ORDER BY)
6. Find all customers who are VIPs (`is_vip = TRUE`).
7. We want to see our most expensive sales. List all purchases, but order them by `price_paid` from highest to lowest.
8. Find all purchases where the `price_paid` was exactly 12.99.
9. List the names of the figures in `series_id` 2.
10. Find all customers who joined after '2025-01-01'.

### Level 3: Basic Math & Aggregation (COUNT, SUM, AVG)
*Hint: You will need to ask your AI how aggregate functions work in SQL.*
11. How many total customers do we have?
12. What is the total revenue (sum of `price_paid`) from all purchases?
13. What is the average (`AVG`) price paid across all our purchases?
14. How many figures are classified as 'Secret' rarity?
15. What is the highest (`MAX`) price anyone has ever paid for a figure?

### Level 4: Bringing it Together (JOINs and GROUP BY)
*Hint: These require combining multiple tables or grouping data. Use your AI to learn about `JOIN`.*
16. We need a report showing the `username` of the customer alongside the `purchase_date` of their purchase. (Requires joining `customers` and `purchases`).
17. Show a list of all `figures` names and the `name` of the `series` they belong to. (Requires joining `figures` and `series`).
18. Find out how much total money *each* customer has spent. Show the `customer_id` and the total `price_paid`. (Requires `GROUP BY`).
19. We want to know which VIP customer spent the most in a single purchase. Find the highest `price_paid` by a customer where `is_vip = TRUE`.
20. **The Final Boss:** Generate a report that shows the `username`, the `name` of the figure they bought, and the `price_paid`. (Requires joining `customers`, `purchases`, and `figures`).
