# Ben-s-Pizzeria-Project

This project demonstrates essential skills for a data analyst, including database design, SQL querying, and integrating a SQL database with a BI tool for interactive dashboards. This README will guide you through the project's goals, setup, and key components.
#







## Project Overview

This project involves:

- Designing and building an SQL database.
- Writing custom SQL queries.
- Connect the database to a BI tool to create interactive dashboards.



## Scenario

Our client, Ben, is opening a new pizzeria that offers takeout and delivery services. He needs a bespoke relational database to capture and store all the critical business data. This database will help Ben monitor business performance through interactive dashboards.
## Key Focus Areas

- Customer Orders: Capture and store order details.
- Stock Levels: Manage inventory and stock control.
- Staff: Monitor staff schedules and calculate labor costs.
## Tools and Technologies

- **Data Organization:** Excel
- **Design:** Quick Database Diagrams
- **Database Management:** MySQL Workbench
- **Visualization:** Looker Studio
- **Storage:** Google Cloud SQL
- **Data Processing:** PowerShell
## Database Design

The database consists of several interconnected tables to minimize redundancy and ensure efficient data management. Below is an outline of the main tables and their relationships. This was achieved using [QuickDBD](https://www.quickdatabasediagrams.com/).

### Tables

| **Table Name**       | **Fields**                                                                                   |
|----------------------|----------------------------------------------------------------------------------------------|
| **Orders Table**     | Order ID, Created At, Item ID, Quantity, Customer ID, Delivery Method, Address ID            |
| **Customers Table**  | Customer ID, First Name, Last Name                                                           |
| **Addresses Table**  | Address ID, Address 1, Address 2, City, Zip Code                                             |
| **Items Table**      | Item ID, SKU, Item Name, Category, Size, Price                                               |
| **Ingredients Table**| Ingredient ID, Name, Weight, Price                                                           |
| **Recipes Table**    | Recipe ID, Ingredient ID, Quantity                                                           |
| **Inventory Table**  | Inventory ID, Item ID, Quantity                                                              |
| **Staff Table**      | Staff ID, First Name, Last Name, Position, Hourly Rate                                       |
| **Shifts Table**     | Shift ID, Day of the Week, Start Time, Finish Time                                           |
| **Rota Table**       | Rota ID, Shift ID, Date, Staff ID                                                            |



### Key Relationships

- Orders to Items: Orders.ItemID = Items.ItemID
- Orders to Addresses: Orders.AddressID = Addresses.AddressID
- Orders to Customers: Orders.CustomerID = Customers.CustomerID
- Recipes to Ingredients: Recipes.IngredientID = Ingredients.IngredientID
- Inventory to Items: Inventory.ItemID = Items.ItemID
- Rota to Staff: Rota.StaffID = Staff.StaffID
- Rota to Shifts: Rota.ShiftID = Shifts.ShiftID
#

### Data Model

![image](https://github.com/user-attachments/assets/ccfeff42-33bc-4887-90bc-b9b6f728784e)
## SQL Queries

### Dashboard 1 - Order activities:
For this, we will need a dashboard with the following data:
-	Total orders
-	Total sales
-	Total items
-	Average order value
-	Sales by category
-	Top selling items
-	Orders by hour
-	Sales by hour
-	Orders by address
-	Orders by delivery/pick up

To get all the relevant data for analyzing the orders activities, the following SQL Queries were defined:

```SQL
use `pizza_db`;

SELECT 
    o.order_id,
    i.item_price,
    o.quantity,
    i.item_cat,
    i.item_name,
    o.created_at,
    a.delivery_address1,
    a.delivery_address2,
    a.delivery_city,
    a.delivery_zipcode
FROM
    orders o
        LEFT JOIN
    item i ON i.item_id = o.item_id
        LEFT JOIN
    address a ON a.add_id = o.add_id

```
#

### Dashboard 2 Stock activities
The client requires the following data to track stock activities:
-	Total quantity by ingredient
-	Total cost of ingredients
-	Calculated cost of pizza
-	Percentage stock remaining by ingredients
-	List of ingredients to re-order based on remaining inventory

To achieve this, we are going to create 2 views. The relevant tables are orders, items, recipe, and ingredients. The query for view 1 is:

```SQL

CREATE VIEW `stock 1` AS
SELECT 
    s1.item_name,
    s1.ing_id,
    s1.ing_name,
    s1.ing_weight,
    s1.ing_price,
    s1.order_quantity,
    s1.recipe_quantity,
    s1.order_quantity*s1.recipe_quantity as ordered_weight,
    s1.ing_price/s1.ing_weight as unit_cost,
    (s1.order_quantity*s1.recipe_quantity)*(s1.ing_price/s1.ing_weight) as ingredient_cost
FROM
    (SELECT 
        o.item_id,
            i.sku,
            i.item_name,
            r.ing_id,
            ing.ing_weight,
            ing.ing_name,
            ing.ing_price,
            r.quantity AS recipe_quantity,
            SUM(o.quantity) AS order_quantity
    FROM
        orders o
    LEFT JOIN item i ON i.item_id = o.item_id
    LEFT JOIN recipe r ON r.recipe_id = i.sku
    LEFT JOIN ingredients ing ON ing.ing_id = r.ing_id
    GROUP BY o.item_id , i.sku , i.item_name , r.ing_id , r.quantity , ing.ing_name , ing.ing_weight , ing.ing_price) s1

```
#

- The query for view 2 is:
#
```SQL

CREATE VIEW `stock 1` AS
SELECT 
    s2.ing_name,
    s2.ordered_weight,
    ing.ing_weight * inv.quantity_int as total_inv_weight,
    (ing.ing_weight * inv.quantity_int)-s2.ordered_weight as remaining_weight
FROM
    (SELECT 
        ing_id, ing_name, SUM(ordered_weight) AS ordered_weight
    FROM
        stock1
    GROUP BY ing_id , ing_name) s2
    
    left join inventory inv on inv.item_id = s2.ing_id
    left join ingredients ing on ing.ing_id = s2.ing_id

```

### Dashboard 3 Staff activities
The client want to be able to efficiently manage staff data, by seeing the following:
- staff name 
- Total Hours worked
- Hourly rates
This information can then be compared with Total Cost of Ingredients and Total Sales to determine if they are running at a profit or loss. To get that information, the following queries were determined:
#

```SQL
SELECT
r.date,
s.staff_first_name,
s.staff_first_name,
s.hourly_rate,
sh.start_time,
sh.end_time,
(hour(timediff(sh.end_time, sh.start_time)) + (minute(timediff(sh.end_time, sh.start_time)))) as hours_in_shift,
(hour(timediff(sh.end_time, sh.start_time)) + (minute(timediff(sh.end_time, sh.start_time)))) * s.hourly_rate as staff_cost

from rota r
left join staff s on s.staff_id = r.staff_id
left join shift sh on sh.shift_id = r.shift_id
```
#
## Hosting Database in the Cloud: Google Cloud SQL Console

To host and manage our MySQL database in the cloud, and to facilitate seamless integration with Looker Studio, we set up an account with Google Cloud. Utilizing the Google Cloud SQL Console, we provisioned a MySQL instance, ensuring that our database is securely hosted in the cloud. This setup allows for robust data management, easy scalability, and reliable performance. With the database hosted on Google Cloud SQL, we can efficiently connect it to Looker Studio for advanced data visualization and analysis. Alternatively, you can set up the database in the cloud using PowerShell and then connect to it using MySQL Workbench.

#

![image](https://github.com/user-attachments/assets/5223a709-6477-4ffb-bd63-3fb38724ae7a)

#


## Business Intelligence Visualization

### Google Looker Studio

[Click To View Full Dashboard](https://lookerstudio.google.com/reporting/1cd383b9-53a7-4a89-bddf-94ebb2472e4a)


![image](https://github.com/user-attachments/assets/27b99dbc-5a6a-440f-9b67-f859b32e1f98)

![image](https://github.com/user-attachments/assets/1229d58b-a1cd-4a7b-b377-6d0e599033fe)

![image](https://github.com/user-attachments/assets/fcab713c-99c4-4ea9-b4e9-1d691a1da7bd)
## Author

- [@drsNdamah](https://www.github.com/drsNdamah)


## Appendix

Project files and queries are included in the folders included in this repository.

