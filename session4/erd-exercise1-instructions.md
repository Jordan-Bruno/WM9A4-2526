# Database Modeling Challenge: Bruno's Burgers

## The Scenario
You have been hired as a consultant to build a database for "Bruno's Burgers", a popular local burger joint. Jordan Bruno is moving his business into the digital age and needs a backend database to power his new online ordering system.

## The Task
Download and open the provided online menu website from the Moodle page. Your goal is to design an Entity-Relationship (ER) diagram (or list out the SQL table schemas) on the large paper provided to support this exact menu and customer orders.

### Requirements:
1. **The Menu Structure:** Your database must store the categories and the menu items, along with their prices.
2. **Customer Orders:** Your database must be able to record a customer's order, tracking exactly which items they bought and calculating a total.
3. **The Customisation Engine:** Look closely at the "Burger Customisation Options" box on the menu. Your database *must* be able to handle these choices dynamically. You need to support required choices (Bread), optional add-ons that change the price, and free removals.

---

## Test Your Design

Once you think your database schema is complete, put it to the test! Try to mentally "insert" this exact order into your tables on paper:

**Order #101:**
* 1x Cheeseburger
  * Choice: Brown Bun
  * Add-on: Bacon
  * Removal: No Tomatoes
* 1x Loaded Fries
* 1x Strawberry Milkshake

*Can your database store this without breaking? Does it know to charge extra for the bacon but nothing for the brown bun? Can it calculate the final correct total?*
