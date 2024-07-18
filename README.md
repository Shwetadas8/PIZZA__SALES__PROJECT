# PIZZA__SALES__PROJECT

-- 1. Retrieve the total number of orders placed.

select count(*) from pizzahut.order_details;

-- 2. Calculate the total revenue generated from pizza sales;
SELECT 
    ROUND(SUM(order_details.quantity * pizzas.price),2) AS Total_revenue
FROM
    order_details
        JOIN
    pizzas ON order_details.pizza_id = pizzas.pizza_id; 
    
    

-- 3. Identify the highest-priced pizza.
SELECT 
    MAX(price), pizza_type_id
FROM
    pizzas
GROUP BY pizza_type_id
ORDER BY MAX(price) DESC
LIMIT 1;

SELECT 
    pizza_types.name, pizzas.price
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
ORDER BY pizzas.price DESC
LIMIT 1;


-- 4. Identify the most common pizza size ordered.
SELECT 
    pizzas.size, COUNT(order_details.order_details_id) as SIZE_COUNT
FROM
    pizzas
        JOIN
    order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizzas.size order by pizzas.size; 


-- 5. List the top 5 most ordered pizza types along with their quantities.

SELECT 
    pizza_types.name, SUM(order_details.quantity) AS QUANTITY
FROM
    pizza_types
        JOIN
        pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
        order_details ON  order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY QUANTITY DESC LIMIT 5;

-- Join the necessary tables to find the total quantity of each pizza category ordered.
SELECT 
    pizza_types.category, SUM(order_details.quantity) AS TOTAL_QUANTITY
FROM
    order_details
        JOIN
    pizzas ON order_details.pizza_id = pizzas.pizza_id
        JOIN
    pizza_types ON pizzas.pizza_type_id = pizza_types.pizza_type_id
    group by pizza_types.category
    order by TOTAL_QUANTITY DESC;


-- 2. Determine the distribution of orders by hour of the day.
SELECT 
    HOUR(order_time) AS HR, COUNT(order_id) AS OID
FROM
    orders
GROUP BY HR;

-- Join relevant tables to find the category-wise distribution of pizzas.

select category, count(name) from pizza_types
group by category;

-- Group the orders by date and calculate the average number of pizzas ordered per day.
SELECT 
    ROUND(AVG(qty), 0) AS AVG_NO_PIZZAS
FROM
    (SELECT 
        orders.order_date, SUM(order_details.quantity) AS QTY
    FROM
        orders
    JOIN order_details ON orders.order_id = order_details.order_details_id
    GROUP BY orders.order_date) AS order_qty;

-- Determine the top 3 most ordered pizza types based on revenue.
SELECT 
    pizza_types.name,
    sum(order_details.quantity * pizzas.price) AS Revenue
FROM
    pizza_types 
        JOIN
    pizzas
    ON pizza_types.pizza_type_id = pizzas.pizza_type_id
    JOIN
order_details
    ON pizzas.pizza_id = order_details.pizza_id
    group by pizza_types.name
    order by Revenue desc LIMIT 3;
    
-- Advanced:
-- Calculate the percentage contribution of each pizza type to total revenue.

SELECT 
    pizza_types.category,
    ROUND((sum(order_details.quantity * pizzas.price)/(SELECT 
    ROUND(SUM(order_details.quantity * pizzas.price),2) AS Total_Sales
FROM
    order_details
        JOIN
   pizzas ON order_details.pizza_id = pizzas.pizza_id)) * 100,2) AS Revenue
FROM
    pizza_types 
        JOIN
    pizzas
    ON pizza_types.pizza_type_id = pizzas.pizza_type_id
    JOIN
order_details
    ON pizzas.pizza_id = order_details.pizza_id
    group by pizza_types.category
    order by Revenue desc LIMIT 3;
    
-- Analyze the cumulative revenue generated over time.

select order_date,
ROUND(sum(revenue) over(order by order_date),2) as cum_rev
from
(select orders.order_date,
sum(order_details.quantity * pizzas.price) as revenue 
from order_details join pizzas
on order_details.pizza_id = pizzas.pizza_id
join orders
on orders.order_id = order_details.order_id
group by orders.order_date) as sales;

-- Determine the top 3 most ordered pizza types based on revenue for each pizza category.
select name,revenue from 
(select category,name,revenue,
rank() over(partition by category order by revenue desc) as rn 
from (
select pizza_types.category,pizza_types.name,
sum((order_details.quantity) * pizzas.price) as revenue 
from pizza_types join pizzas
on pizza_types.pizza_type_id = pizzas.pizza_type_id
join order_details
on order_details.pizza_id = pizzas.pizza_id
group by pizza_types.category,pizza_types.name) as a) as b
where rn <=3;
