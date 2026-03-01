# Zomato Datebase Management System using SQL Project --P2

## Project Overview

**Project Title**: Zomato Datebase Management System
**Level**: Intermediate  
**Database**: `Zomato_P3`

This project demonstrates the implementation of a Zomato-style Food Delivery Management System using SQL. It focuses on designing and managing a relational database, performing CRUD operations, and executing SQL queries to analyze food ordering and delivery data. The project highlights skills in database design, data manipulation, and querying.

![Zomato_project](https://github.com/tanish-6126/Zomato_Project/blob/main/Zomato-bg.png)

## Objectives

**1.Set up the Food Delivery Management System database.**

**2.Create and manage tables for customers, restaurants, orders, riders, and deliveries.**

**3.Perform CRUD (Create, Read, Update, Delete) operations.**

**4.Execute SQL queries to retrieve and analyze operational data.**

## Project Structure

### 1. Database Setup
![ERD](https://github.com/tanish-6126/Zomato_Project/blob/main/EER_Zomato.png)

- **Database Creation**: Created a database named `Zomato_P3`.
### 2.Table Creation 
  1.Created tables for customers, restaurants, orders, riders, and deliveries.
  
  2.Defined relationships using primary and foreign keys to ensure data consistency.

```sql
CREATE DATABASE Zomato_P3;

CREATE TABLE customers
(
customer_id INT PRIMARY KEY,
customer_name VARCHAR(25),
reg_date DATE
);

CREATE TABLE restaurants
(
restaurant_id INT PRIMARY KEY,
restaurant_name VARCHAR(55),
city VARCHAR(15),
opening_hours VARCHAR(55)
);


CREATE TABLE orders
(
order_id INT PRIMARY KEY,
customer_id INT, -- this is coming from cx table
restaurant_id INT, -- this is coming from restaurants rable
order_item VARCHAR(55),
order_date DATE,
order_time TIME,
order_status VARCHAR(55),
total_amount FLOAT
);

CREATE TABLE riders
(
rider_id INT PRIMARY KEY,
rider_name VARCHAR(55),
sign_up DATE
);

CREATE TABLE deliveries
(
delivery_id INT PRIMARY KEY,
order_id int, -- this coming orders table
delivery_status VARCHAR(35),
delivery_time TIME,
rider_id INT -- this is coming riders
);

UPDATE deliveries
SET delivery_status = 'Not Delivered'
WHERE delivery_id % 4 = 0;

-- Adding constraints..
ALTER TABLE orders
ADD CONSTRAINT fk_customers
FOREIGN KEY (customer_id)
REFERENCES customers(customer_id);

ALTER TABLE orders
ADD CONSTRAINT fk_restaurants
FOREIGN KEY (restaurant_id)
REFERENCES restaurants(restaurant_id);

Alter table deliveries
add CONSTRAINT fk_orders
FOREIGN KEY (order_id)
REFERENCES orders (order_id);

Alter table deliveries
add CONSTRAINT fk_riders
FOREIGN KEY (rider_id)
REFERENCES riders(rider_id);
```

### 3. CRUD Operations

- **Create**: Inserted sample records into the database tables.
- **Read**:Retrieved data to analyze customer orders, restaurant activity, and delivery status.
- **Update**: Updated order and delivery-related information as required.
- **Delete**: Deleted records such as cancelled orders or inactive entries.

**Task 1.Write a query to find the top 5 most frequently ordered dishes by customer called "Arjun Mehta" in the last 2.5 year.**

```sql
select * from(
select 
c.customer_name,o.order_item,count(*) as cnt ,dense_rank() over (order by count(*) desc)top
from customers c 
inner join 
orders o 
on o.customer_id = c.customer_id 
where o.order_date >=current_date() - interval 30 month
and c.customer_name ='Arjun Mehta'
group by c.customer_id,
o.order_item) a
where top <= 5;
```
**Task 2:Popular Time Slots
-- Identify the time slots during which the most orders are placed. based on 2-hour intervals.**

```sql
SELECT
    CASE
        WHEN HOUR(order_time) BETWEEN 0 AND 1 THEN '00:00 - 02:00'
        WHEN HOUR(order_time) BETWEEN 2 AND 3 THEN '02:00 - 04:00'
        WHEN HOUR(order_time) BETWEEN 4 AND 5 THEN '04:00 - 06:00'
        WHEN HOUR(order_time) BETWEEN 6 AND 7 THEN '06:00 - 08:00'
        WHEN HOUR(order_time) BETWEEN 8 AND 9 THEN '08:00 - 10:00'
        WHEN HOUR(order_time) BETWEEN 10 AND 11 THEN '10:00 - 12:00'
        WHEN HOUR(order_time) BETWEEN 12 AND 13 THEN '12:00 - 14:00'
        WHEN HOUR(order_time) BETWEEN 14 AND 15 THEN '14:00 - 16:00'
        WHEN HOUR(order_time) BETWEEN 16 AND 17 THEN '16:00 - 18:00'
        WHEN HOUR(order_time) BETWEEN 18 AND 19 THEN '18:00 - 20:00'
        WHEN HOUR(order_time) BETWEEN 20 AND 21 THEN '20:00 - 22:00'
        WHEN HOUR(order_time) BETWEEN 22 AND 23 THEN '22:00 - 00:00'
    END AS time_slot,
    COUNT(order_id) AS order_count
FROM orders
GROUP BY time_slot
ORDER BY order_count DESC;
```

**Task 3: Order Value Analysis
--  Find the average order value per customer who has placed more than 250 orders.
-- Return customer_name, and aov(average order value)**

```sql
select c.customer_name,c.customer_id,count(order_id)orders ,avg(total_amount)Average
from customers c 
inner Join 
orders o
on o.customer_id = c.customer_id 
group by c.customer_id
having orders > 250;
```

**Task 4:High-Value Customers
--  List the customers who have spent more than 100K in total on food orders.
-- return customer_name, and customer_id!**

```sql
select c.customer_name,c.customer_id,sum(total_amount) total
from customers c
inner join 
orders o
on c.customer_id = o.customer_id 
group by c.customer_id
having total >100000
order by total;
```

**Task 5: Orders Without Delivery
-- Write a query to find orders that were placed but not delivered.
-- Return each restuarant name, city and number of not delivered orders..**

```sql
SELECT
    r.restaurant_name,
    r.city,
    COUNT(o.order_id) AS not_delivered_orders
FROM orders o
LEFT JOIN deliveries d
    ON o.order_id = d.order_id
JOIN restaurants r
    ON o.restaurant_id = r.restaurant_id 
WHERE d.delivery_status = 'Not Delivered'
   OR d.delivery_id IS NULL
GROUP BY
    r.restaurant_name,
    r.city
ORDER BY not_delivered_orders DESC;
```

- **Task 6: Restaurant Revenue Ranking:
-- Rank restaurants by their total revenue from the last 2.5 year, including their name,
-- total revenue, and rank within their city.**:
  
```sql
select * from 
(select r.city, r.restaurant_id, r.restaurant_name,sum(total_amount)total_revenue,
rank() over(partition by city order by sum(total_amount) desc)rank_city 
from orders o
left Join 
restaurants r 
on r.restaurant_id = o.restaurant_id 
where  o.order_date >= current_date() - interval 30 month
group by r.restaurant_id, r.restaurant_name)a 
where rank_city =1;

```

### 4. Data Analysis & Findings

**Task 7. Most Popular Dish by City:
-- Mentify the most popular dish in each city based on the number of orders.**:

```sql
select city,order_item from (
select r.city,o.order_item,count(order_item)no_of_orders,
rank() over(partition by city order by count(order_item) desc)popular_dish
from restaurants r
inner join 
orders o 
on o.restaurant_id = r.restaurant_id
group by r.city,o.order_item) a 
where popular_dish = 1;
```

8. **Task 8: Customer Churn:
-- Find customers who haven't placed an order in 2024 but did in 2023.**:

```sql
select distinct customer_id from orders 
where year(order_date) = 2023 and
customer_id not in
(select distinct customer_id from orders where year(order_date) = 2024);
```

9. **Rider Average Delivery Time:
-- Determine each rider's average delivery time.**:
   
```sql
SELECT
    d.rider_id,
    ROUND(AVG(CASE
            WHEN d.delivery_time < o.order_time
            THEN TIMESTAMPDIFF(SECOND, o.order_time, d.delivery_time) + 86400
            ELSE TIMESTAMPDIFF(SECOND, o.order_time, d.delivery_time)
        END
    ) / 60, 2) AS avg_delivery_time_minutes
FROM orders o
JOIN deliveries d
    ON o.order_id = d.order_id
GROUP BY d.rider_id
ORDER BY avg_delivery_time_minutes;
```

10. **Order Item Popularity:
-- Track the popularity of specific order items over time and identify seasonal demand spikes.**:

```sql
Select Order_item,Seasons,Sum(Total_amount) as Sales
From
(Select
	*,
    CASE
		WHEN Month(Order_date) Between 4 and 6 then "Spring"
        WHEN Month(Order_date) > 6 AND Month(Order_date) < 9 then "Summer"
        ELSE "Winter"
	END as Seasons
from Orders) as X
Group by 1,2
order by 1, 3 Desc;
```

Task 11. **Monthly Restaurant Growth Ratio:
-- Calculate each restaurant's growth ratio based on the total number of delivered orders since its joining..**:

```sql
select *,round((cnt_orders - previous_month)/previous_month ,2) as growth_ratio from (
select *,
lag(cnt_orders) over (partition by restaurant_id order by month) previous_month from (
select r.restaurant_id,
date_format(o.order_date,'%y-%m') as month,
count(o.order_id) as cnt_orders
from orders o
Join 
deliveries d 
on o.order_id = d.order_id
join 
restaurants r 
on r.restaurant_id = o.restaurant_id
where d.delivery_status = 'Delivered'
group by r.restaurant_id,
date_format(o.order_date,'%y-%m')
order by restaurant_id,
month)t)a;
```

## Advanced SQL Operations

**Task 12: Customer Segmentation: Segment customers into 'Gold' or 'Silver' groups based on their total spending
-- compared to the average order value per customer. If a customer's total spending exceeds the Average value,
-- label them as 'Gold'; otherwise, label them as 'Silver'.Write an SQL query to determine each segment's
-- total number of orders and total revenue**

```sql
select segment,
count(total_orders) as total_orders,
sum(total_spending) as total_revenue 
from (
select customer_id,sum(total_amount) as total_spending,
count(order_item) as total_orders,
case when sum(total_amount) > (select sum(total_amount)/count(distinct customer_id) from orders) 
then 'Gold' else 'Silver' end as segment 
from orders 
group by customer_id) a
group by segment;
```

**Task 13: Rider Monthly Earnings:
-- Calculate each rider's total monthly earnings, assuming they earn 8% of the order amount.**  

```sql

select rider_id,month,round((total_earning*1.08),2) as earning_net from (
SELECT
    a.rider_id,
    DATE_FORMAT(a.order_date, '%Y-%m') AS month,
    SUM(a.total_amount) AS total_earning
FROM (SELECT o.order_id,o.order_date,o.total_amount,d.rider_id
    FROM orders o
    JOIN deliveries d
        ON o.order_id = d.order_id
    WHERE d.delivery_status = 'Delivered'
) a
GROUP BY a.rider_id,DATE_FORMAT(a.order_date, '%Y-%m')
ORDER BY a.rider_id,month) b;
```

**Task 14: Rider Ratings Analysis:
-- Find the number of 5-star, 4-star, and 3-star ratings each rider has.
-- riders receive this rating based on delivery time.
-- If orders are delivered less than 60 minutes of order received time the rider get 5 star rating,
-- if they deliver 60 and 100 minute they get 4 star rating
-- if they deliver after 100 minute they get 3 star rating.**  

```sql
select *,(case when delivered_time <=200 then '5-Star'
               when delivered_time <=300 then '4-Star'
			   else '3-Star' end)rating from (
select rider_id,
(case when delivery_time > order_time then timestampdiff(second,order_time,delivery_time)
else timestampdiff(second,order_time,delivery_time) + 86400 end) /60 as delivered_time 
from orders o
join
deliveries d 
on o.order_id = d.order_id
where d.delivery_status ='Delivered'
order by rider_id)a;
```

**Task 15: Order Frequency by Day:
-- Analyze order frequency per day of the week and identify the peak day for each nestaurant.**

```sql

select * from (
select r.restaurant_id,dayname(order_date)Day,count(dayname(order_date))as frequency ,
rank() over(partition by r.restaurant_id order by count(dayname(order_date)) desc)peak_day
from orders o
join 
restaurants r 
on r.restaurant_id = o.restaurant_id 
group by r.restaurant_id,dayname(order_date)) a
where peak_day = 1;

```

## Reports

- **Database Schema**: Designed the database schema based on the ER diagram.
- **Data Analysis**:<br>
  1.Insights into customer ordering behavior.
 
  2.Restaurant performance analysis.
 
  3.Delivery status and operational efficiency

## Conclusion

This project demonstrates the practical use of SQL to build and manage a food delivery management database. It covers database creation, table relationships, CRUD operations, and analytical queries. The project reflects how platforms like Zomato handle structured data for daily operations and analysis.

## Author - Tanish

