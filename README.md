# Pizza Sales Analysis using SQL

SQL portfolio project analysing one year (2015) of pizza restaurant sales data to find what drives orders and revenue.

**Author:** Anushree Agarwal | SQL, Excel

## Dataset
4 related tables: `orders` (21,350 rows), `orders_details` (48,620 rows), `pizzas` (96 rows), `pizza_types` (32 rows).

## Key results
- Total revenue: **$817,860** from **21,350** orders
- Average **138** pizzas ordered per day
- Peak hours: 12-1 PM and 5-7 PM
- Large (L) is the most ordered size; Classic is the top category
- The Thai Chicken Pizza earns the most revenue

## SQL concepts used
Joins, GROUP BY, subqueries, window functions (`SUM() OVER`, `RANK() OVER`)

## Questions, queries and results

### Basic

#### Q1. Retrieve the total number of orders placed.

```sql
SELECT 
    COUNT(order_id) AS total_orders
FROM
    orders;
```

**Result**

`total_orders` = **21,350**

#### Q2. Calculate the total revenue generated from pizza sales.

```sql
SELECT 
    ROUND(SUM(orders_details.quantity * pizzas.price),
            2) AS total_sales
FROM
    orders_details
        JOIN
    pizzas ON pizzas.pizza_id = orders_details.pizza_id;
```

**Result**

`total_sales` = **$817,860.05**

#### Q3. Identify the highest-priced pizza.

```sql
SELECT 
    pizza_types.name, pizzas.price
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
ORDER BY pizzas.price DESC
LIMIT 1;
```

**Result**

| name | price |
|---|---|
| The Greek Pizza | 35.95 |

#### Q4. Identify the most common pizza size ordered.

```sql
SELECT 
    pizzas.size,
    COUNT(orders_details.order_details_id) AS orders_count
FROM
    pizzas
        JOIN
    orders_details ON pizzas.pizza_id = orders_details.pizza_id
GROUP BY pizzas.size
ORDER BY orders_count DESC;
```

**Result**

| size | orders_count |
|---|---|
| L | 18526 |
| M | 15385 |
| S | 14137 |
| XL | 544 |
| XXL | 28 |

#### Q5. List the top 5 most ordered pizza types along with their quantities.

```sql
SELECT 
    pizza_types.name, SUM(orders_details.quantity) AS quantity
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    orders_details ON orders_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY quantity DESC
LIMIT 5;
```

**Result**

| name | quantity |
|---|---|
| The Classic Deluxe Pizza | 2453 |
| The Barbecue Chicken Pizza | 2432 |
| The Hawaiian Pizza | 2422 |
| The Pepperoni Pizza | 2418 |
| The Thai Chicken Pizza | 2371 |

### Intermediate

#### Q6. Join the necessary tables to find the total quantity of each pizza category ordered.

```sql
SELECT 
    pizza_types.category,
    SUM(orders_details.quantity) AS quantity
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    orders_details ON orders_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category
ORDER BY quantity DESC;
```

**Result**

| category | quantity |
|---|---|
| Classic | 14888 |
| Supreme | 11987 |
| Veggie | 11649 |
| Chicken | 11050 |

#### Q7. Determine the distribution of orders by hour of the day.

```sql
SELECT 
    HOUR(order_time) AS HOUR, COUNT(order_id) AS orders_count
FROM
    orders
GROUP BY HOUR(order_time);
```

**Result**

| HOUR | orders_count |
|---|---|
| 9 | 1 |
| 10 | 8 |
| 11 | 1231 |
| 12 | 2520 |
| 13 | 2455 |
| 14 | 1472 |
| 15 | 1468 |
| 16 | 1920 |
| 17 | 2336 |
| 18 | 2399 |
| 19 | 2009 |
| 20 | 1642 |
| 21 | 1198 |
| 22 | 663 |
| 23 | 28 |

#### Q8. Join relevant tables to find the category-wise distribution of pizzas.

```sql
SELECT 
    category, COUNT(name)
FROM
    pizza_types
GROUP BY category;
```

**Result**

| category | COUNT(name) |
|---|---|
| Supreme | 9 |
| Veggie | 9 |
| Classic | 8 |
| Chicken | 6 |

#### Q9. Group the orders by date and calculate the average number of pizzas ordered per day.

```sql
SELECT 
    ROUND(AVG(quantity), 0) AS avg_pizza_ordered_per_day
FROM
    (SELECT 
        orders.order_date, SUM(orders_details.quantity) AS quantity
    FROM
        orders
    JOIN orders_details ON orders.order_id = orders_details.order_id
    GROUP BY orders.order_date) AS order_quantity;
```

**Result**

`avg_pizza_ordered_per_day` = **138**

#### Q10. Determine the top 3 most ordered pizza types based on revenue.

```sql
SELECT 
    pizza_types.name,
    SUM(orders_details.quantity * pizzas.price) AS revenue
FROM
    pizza_types
        JOIN
    pizzas ON pizzas.pizza_type_id = pizza_types.pizza_type_id
        JOIN
    orders_details ON orders_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY revenue DESC
LIMIT 3;
```

**Result**

| name | revenue |
|---|---|
| The Thai Chicken Pizza | 43434.25 |
| The Barbecue Chicken Pizza | 42768.00 |
| The California Chicken Pizza | 41409.50 |

### Advanced

#### Q11. Calculate the percentage contribution of each pizza category to total revenue.

```sql
SELECT 
    pizza_types.category,
    ROUND(SUM(orders_details.quantity * pizzas.price) / (SELECT 
                    ROUND(SUM(orders_details.quantity * pizzas.price),
                                2) AS total_sales
                FROM
                    orders_details
                        JOIN
                    pizzas ON pizzas.pizza_id = orders_details.pizza_id) * 100,
            2) AS revenue
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    orders_details ON orders_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category
ORDER BY revenue DESC;
```

**Result**

| category | revenue (%) |
|---|---|
| Classic | 26.91 |
| Supreme | 25.46 |
| Chicken | 23.96 |
| Veggie | 23.68 |

#### Q12. Analyze the cumulative revenue generated over time.

```sql
SELECT order_date,
SUM(revenue) over(ORDER BY order_date) AS cum_revenue
FROM (SELECT orders.order_date,
SUM(orders_details.quantity * pizzas.price) AS revenue FROM orders_details
JOIN pizzas
ON orders_details.pizza_id = pizzas.pizza_id
JOIN orders
ON orders.order_id = orders_details.order_id
GROUP BY orders.order_date) AS sales;
```

**Result**

First row: 2015-01-01 -> 2713.85 

Last row: 2015-12-31 -> **817860.05** (equals total sales from Q2)

#### Q13. Determine the top 3 most ordered pizza types based on revenue for each pizza category.

```sql
SELECT name, revenue FROM 
(SELECT category, name, revenue,
rank() over(partition by category ORDER BY revenue DESC)
AS rn FROM 
(SELECT pizza_types.category, pizza_types.name,
SUM((orders_details.quantity) * pizzas.price) AS revenue 
FROM pizza_types JOIN pizzas
ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN orders_details
ON orders_details.pizza_id = pizzas.pizza_id
GROUP by pizza_types.category, pizza_types.name) AS a) AS b 
WHERE rn <= 3;
```

**Result**

| category | name | revenue |
|---|---|---|
| Chicken | The Thai Chicken Pizza | 43434.25 |
| Chicken | The Barbecue Chicken Pizza | 42768.00 |
| Chicken | The California Chicken Pizza | 41409.50 |
| Classic | The Classic Deluxe Pizza | 38180.50 |
| Classic | The Hawaiian Pizza | 32273.25 |
| Classic | The Pepperoni Pizza | 30161.75 |
| Supreme | The Spicy Italian Pizza | 34831.25 |
| Supreme | The Italian Supreme Pizza | 33476.75 |
| Supreme | The Sicilian Pizza | 30940.50 |
| Veggie | The Four Cheese Pizza | 32265.70 |
| Veggie | The Mexicana Pizza | 26780.75 |
| Veggie | The Five Cheese Pizza | 26066.50 |

(Your query returns only `name` and `revenue`; category is shown here for readability.)

## Files
- `Pizza_Sales_SQL_Portfolio.pdf`: project presentation
- `pizza_sales.zip`: raw dataset (4 CSV files: orders, order_details, pizzas, pizza_types)
