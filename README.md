# SQL Project: Data analysis for Zomato, a food delivery company   

<p align="center">
  <img src="https://www.zomato.com/deliver-food/images/ladies.png" alt="Zomato Food Delivery" width="1000">
</p>



## **Project overview**:  

**Project title** : Data analysis for Zomato, a food delivery company  
**Author** : LABHALLA Halima    
**Level** : Advanced      
**Database** :  zoomato_db  
This project demonstrates my SQL problem_solving skills through the analysis of data for zomato, a popular food delivery company in India. This projct involves setting up the database, 
importing data, handeling null values, and solving a variety of business problems using complex SQL queries.   

## **Project structure** :   

- **Database Setup** :  creation of the `zoomato_db` and the required tables.   
- **Data import** : inserting sample data into the tables.   
- **Data cleaning** : Handling null values and insuring data integrity.   
- **Business problems** : solving 20 specific business problem using SQL queries.

### Database setup  
```sql
CREATE DATABASE zoomato_db;
```
1. **Dropping existing tables**
```sql
DROP TABLE IF EXISTS customers;
DROP TABLE IF EXISTS restaurants;
DROP TABLE IF EXISTS orders;
DROP TABLE IF EXISTS riders;
DROP TABLE IF EXISTS deliveries;  
```
2. **Creating tables**
```sql
DROP TABLE IF EXISTS customers;
CREATE TABLE customers(
customer_id INT PRIMARY KEY, 
customer_name VARCHAR(25), 
reg_date DATE
); 

--restaurants table
DROP TABLE IF EXISTS restaurants; 
CREATE TABLE restaurants(
restaurant_id INT PRIMARY KEY, 
restaurant_name VARCHAR(55), 
city VARCHAR(15), 
opening_hours VARCHAR(55) 
); 

--orders table
DROP TABLE IF EXISTS orders; 
CREATE TABLE orders(
order_id INT PRIMARY KEY, 
customer_id INT,--this is coming from customers table 
restaurant_id INT, --this is coming from restaurant table
order_item VARCHAR(55),
order_date DATE,
order_time TIME,
order_status VARCHAR(55), 
order_amount FLOAT
); 
--Adding fk CONSTRAINTS 
ALTER TABLE orders
ADD CONSTRAINT fk_customers
FOREIGN KEY (customer_id) 
REFERENCES customers(customer_id)

ALTER TABLE orders 
ADD CONSTRAINT fk_restaurants
FOREIGN KEY (restaurant_id)
REFERENCES restaurants(restaurant_id)

--riders table
DROP TABLE IF EXISTS riders;
CREATE TABLE riders(
rider_id INT PRIMARY KEY, 
rider_name VARCHAR(55), 
sign_up DATE
); 

--delivery table
DROP TABLE IF EXISTS deliveries; 
CREATE TABLE deliveries(
delivery_id INT PRIMARY KEY, 
order_id INT, 
delivery_status VARCHAR(35), 
delivery_time TIME, 
rider_id INT,
CONSTRAINT fk_orders FOREIGN KEY(order_id) REFERENCES orders(order_id), 
CONSTRAINT fk_riders FOREIGN KEY(rider_id) REFERENCES riders(rider_id)

);

--End of schemas 

```
### Data import  

**Data cleaning and handling null values**   

Before performing analysis, i ensured that the data was cleaned and free from null values where necessary, for instance :   
```sql
--checking for null values in the restaurants table
SELECT COUNT(*) FROM restaurants 
WHERE 
restaurant_id IS NULL 
OR 
restaurant_name IS NULL 
OR 
city IS NULL 
OR
opening_hours IS NULL 
```
### Business problem solved   
**Q1. Write a Query to find the top 5 most frequently ordered dishes by customer called 'Arjun Mehta' in the last 1 year**    
```sql

SELECT
customer_name, 
dishes, 
total_orders
FROM
(SELECT 
c.customer_id,
c.customer_name, 
o.order_item as dishes, 
COUNT(*) as total_orders,-- count(*) va s'appliquer sur chaque combinaison unique de customer_id, customer_name, order_item
DENSE_RANK() OVER(ORDER BY COUNT (*) desc) as rank--cette ligne sert à classer les lignes du résultat par nombre de lignes COUNT, de la plus grande à la plus petite sans sauter de rang en cas d'égalité
FROM orders o
JOIN customers c ON c.customer_id = o.customer_id
WHERE
o.order_date >= CURRENT_DATE - INTERVAL' 2 Year'
AND 
c.customer_name = 'Arjun Mehta'
GROUP BY 1,2,3 
ORDER BY 1,4 DESC) as t1 -- this t1 is a table 

WHERE rank <= 5  
```
**Q2. Identify the time slots during which the most orders are placed based on 2 hour intervales**  
```sql
-----Approach 1 
SELECT
    CASE
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 0 AND 1  THEN '00:00 - 02:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 2 AND 3  THEN '02:00 - 04:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 4 AND 5  THEN '04:00 - 06:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 6 AND 7  THEN '06:00 - 08:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 8 AND 9  THEN '08:00 - 10:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 10 AND 11 THEN '10:00 - 12:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 12 AND 13 THEN '12:00 - 14:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 14 AND 15 THEN '14:00 - 16:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 16 AND 17 THEN '16:00 - 18:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 18 AND 19 THEN '18:00 - 20:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 20 AND 21 THEN '20:00 - 22:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 22 AND 23 THEN '22:00 - 00:00'
    END AS time_slot,
    COUNT(order_id) AS order_count
FROM orders
GROUP BY time_slot
ORDER BY order_count DESC;  

--Approach 2 
SELECT 
FLOOR (EXTRACT (HOUR FROM order_time) / 2)*2  as start_time, -- la fonction floor permet en cas de virgule dans la sortie de division de rendre uniquement le chiffre avant la virgule 
FLOOR (EXTRACT (HOUR FROM order_time) / 2)*2 + 2 as end_time, 
COUNT(*) as total_orders
FROM orders
GROUP BY 1, 2
ORDER BY 3 DESC
```
**Q3. find the average order value per customers who have placed more than 750 orders, return customer name and order average value**    
```sql
SELECT 
c.customer_name,
o.customer_id, 
AVG(o.order_amount) as avg,
COUNT(*) as customers_with_more_than_750_order
FROM orders o JOIN customers c ON c.customer_id = o.customer_id
GROUP BY 1, 2 
HAVING COUNT(*) >= 750
ORDER BY COUNT(*) DESC
```
**Q4. list the customers who have spent more than 100 k in total on food_order, return customer name, customer_id**   
```sql
SELECT
c.customer_name, 
c.customer_id, 
SUM(o.order_amount) as customers_with_orders_more_than_100k
FROM orders o 
JOIN customers c ON c.customer_id =o.customer_id
GROUP BY 1,2 
HAVING SUM(o.order_amount) > 100000 
ORDER BY SUM(o.order_amount) DESC
```
**Q5. write a query to find orders that  were placed but not delivered, return each Restaurant name, city and number of not delivered orders**   
```sql
SELECT 
r.restaurant_name, 
r.city, 
COUNT(o.order_id) as cnt.not_delivered_orders
FROM Restaurants r 
LEFT JOIN orders o ON o.restaurant_id = r.restaurant_id
LEFT JOIN deliveries d ON d.order_id = o.order_id
WHERE d.delivery_id  IS NULL 
GROUP BY 1,2 
ORDER BY 3 DESC
```
**Q6. rank restaurants by their total revenue from the last year, including their name, total revenu and rank within their city**  
```sql
--(this time we will use a CTE )WITH ranking_table AS 

WITH ranking_table AS
(
SELECT 
r.city, 
r.restaurant_name, 
SUM(o.order_amount) as revenu, 
RANK()OVER(PARTITION BY r.city ORDER BY SUM(o.order_amount) DESC) as rank
FROM Restaurants r 
JOIN orders o ON o.restaurant_id = r.restaurant_id 
WHERE O.order_date >= CURRENT_DATE - INTERVAL'2 years'
GROUP BY 1,2 
ORDER BY 3 DESC 
) 
SELECT 
* FROM  ranking_table 
WHERE rank = 1
```
**Q7. identify the most popular dish in each city based on the number of orders  (this time we will use a subquery)**   
```sql
SELECT*
FROM
(SELECT 
r.city,
o.order_item,
COUNT(o.order_id) as total_orders, 
RANK()OVER(PARTITION BY r.city ORDER BY COUNT(o.order_id) DESC) as rank 
FROM Restaurants r JOIN orders o ON o.restaurant_id = r.restaurant_id
GROUP BY 1,2) as t1
WHERE rank = 1  
```
**Q8. Find customers who haven't placed an order in 2024 but did in 2023**   
```sql
SELECT DISTINCT customer_id FROM orders 
WHERE EXTRACT (YEAR FROM order_date) = 2023 
AND 
customer_id NOT IN (SELECT DISTINCT customer_id FROM orders 
WHERE EXTRACT (YEAR FROM order_date) = 2024)
```
**Q9. calculate and compare the order cancellation rate for each restaurant between the current date and the previous year**  
```sql
WITH cancel_ratio_2023 AS (
    SELECT 
        o.restaurant_id, 
        COUNT(o.order_id) AS total_orders,
        COUNT(CASE WHEN d.delivery_id IS NULL THEN 1 END) AS not_delivered
    FROM orders AS o 
    LEFT JOIN deliveries AS d 
        ON o.order_id = d.order_id 
    WHERE EXTRACT(YEAR FROM o.order_date) = 2023
    GROUP BY o.restaurant_id
), 

cancel_ratio_2024 AS (
    SELECT 
        o.restaurant_id, 
        COUNT(o.order_id) AS total_orders,
        COUNT(CASE WHEN d.delivery_id IS NULL THEN 1 END) AS not_delivered
    FROM orders AS o 
    LEFT JOIN deliveries AS d 
        ON o.order_id = d.order_id 
    WHERE EXTRACT(YEAR FROM o.order_date) = 2024
    GROUP BY o.restaurant_id
), 

last_year_data AS (
    SELECT 
        restaurant_id,
        total_orders,
        not_delivered, 
        ROUND((not_delivered::numeric / total_orders::numeric) * 100, 2) AS cancelation_ratio
    FROM cancel_ratio_2023
), 

current_year_data AS (
    SELECT 
        restaurant_id,
        total_orders,
        not_delivered, 
        ROUND((not_delivered::numeric / total_orders::numeric) * 100, 2) AS cancelation_ratio
    FROM cancel_ratio_2024
)

SELECT 
    c.restaurant_id, 
    c.cancelation_ratio AS current_year_cancel_ratio, 
    l.cancelation_ratio AS last_year_cancel_ratio
FROM current_year_data AS c 
JOIN last_year_data AS l 
    ON c.restaurant_id = l.restaurant_id;
```
**Q10. determine each riders average delivery time**  
```sql
SELECT
  o.order_id,
  d.rider_id,
  EXTRACT(EPOCH FROM (
    d.delivery_time - o.order_time
    + CASE 
        WHEN d.delivery_time < o.order_time THEN INTERVAL '1 day'
        ELSE INTERVAL '0 day'
      END
  )) / 60 AS time_difference_in_min
FROM orders o
JOIN deliveries d ON o.order_id = d.order_id
WHERE d.delivery_status = 'Delivered';  
```
**Q11. calculate each Restaurant Growth Ratio based On the total number of delivered orders since its joining**   
```sql
WITH growth_ratio 
AS 
(SELECT 
o.Restaurant_id, 
TO_CHAR(order_date, 'mm-yy') as Month, 
COUNT(o.order_id) as cr_month_orders, 
LAG(COUNT(o.order_id), 1) OVER(PARTITION BY O. restaurant_id ORDER BY TO_CHAR(order_date, 'mm-yy') ) as prev_month_orders
FROM orders o 
JOIN 
deliveries as d
ON o.order_id = d.order_id
WHERE d.delivery_status = 'Delivered' 
GROUP BY 1,2)
SELECT
Restaurant_id, 
month, 
cr_month_orders,
prev_month_orders, 
ROUND(
        CASE 
            WHEN prev_month_orders IS NOT NULL AND prev_month_orders != 0 THEN 
                (cr_month_orders::numeric - prev_month_orders::numeric) / prev_month_orders::numeric * 100
            ELSE NULL
        END, 
        2
    ) AS growth_ratio
FROM
growth_ratio --in this question i'll have each restaurant with month column, current_month_sales, last month_sales and growth ratio 
ORDER BY restaurant_id, month;
```
**Q.12 Segment customers into 'Gold', 'Silver' groups based on their total spendings, label them as golds otherwise label them as silvers write a segment query to determine each segment's total number of orders and total revenu**  
```sql
SELECT 
cx_category, 
COUNT(order_id) as Total_orders, 
SUM(total_spent) as total_revenu
FROM
(
SELECT 
order_id,
customer_id, 
SUM(order_amount) as total_spent, 
CASE 
   WHEN SUM(order_amount) > (SELECT AVG(order_amount) FROM orders )THEN 'Gold' -- this is called an inner query : (SELECT AVG(order_amount) FROM orders )
   ELSE 'Silver'
END AS cx_category
FROM orders 
GROUP BY 1) as t1
GROUP BY 1 
```
**Q13. calculate each rider's total monthly earnings, assuming they earn 8% of the order amount**   
```sql
SELECT 
d.rider_id, 
TO_CHAR(o.order_date, 'mm-yy') as month, 
SUM(o.order_amount) as revenu, 
SUM(o.order_amount) * 0.08  as riders_earning
FROM orders o 
JOIN deliveries d
ON o.order_id = d.order_id 
GROUP BY 1,2
ORDER BY 1,2 DESC --(Donc, la base de données va trier d’abord par restaurant_id (ordre croissant par défaut), puis à l’intérieur de chaque restaurant, elle va trier les lignes par month (ordre croissant aussi).)
```
**Q14. Rider ratings analysis : 
--Find the number of 5-star, 4-star, 3-star ratings each rider has 
--riders recieve this rating based on delivery time 
-- if orders are delivered less than 15 minutes of order recieved time the rider get 5 star  rating 
--if they deliver 15 and 20 minute they get 4 star rating 
-- if they deliver after 20 minute they get 3 star rating**    
```sql
SELECT 
rider_id, 
stars, 
COUNT(*) as total_stars 

FROM
(
SELECT 
rider_id, 
delivery_took_time, 
CASE
WHEN delivery_took_time < 15 THEN '5 stars'
WHEN delivery_took_time BETWEEN 15 AND 20 THEN '4 stars'
ELSE '3 stars'
END as stars 
FROM(
SELECT 
   o.order_id, 
   o.order_time, 
   d.delivery_time,
   EXTRACT (EPOCH FROM (  d.delivery_time -   o.order_time  +
   CASE WHEN d.delivery_time < o.order_time THEN INTERVAL '1 day'
   ELSE  INTERVAL '0 day'
   END )) / 60  as delivery_took_time, 
   d.rider_id
FROM orders as o 
JOIN deliveries as d 
ON o.order_id = d.order_id 
WHERE delivery_status ='Delivered') as t1 ) as t2 
GROUP BY 1,2 
ORDER BY 1,3
```
**Q15. Analyze order frequency per day of the week and identify the peak week for each restaurant**  
```sql
SELECT *
FROM(
SELECT 
r.restaurant_name, 
o.order_date, 
COUNT(o.order_id) as total_orders,
RANK() OVER(PARTITION BY r.restaurant_name ORDER BY COUNT(o.order_id) DESC) as rank, 
TO_CHAR(o.order_date, 'Day') as  day 
FROM orders o 
JOIN Restaurants r ON o.restaurant_id = r.restaurant_id 
GROUP BY 1,2 
ORDER BY 1, 3 DESC)--tirer par la première colonne du select en ordre ascendant par défaut puis par la colonne 3 DESC
as t1 
WHERE rank = 1
```
**Q16. Calculate the total revenu generated by each customer over all their orders 'customer lifetime value(CLV)'**  
```sql
SELECT
o.customer_id,
c.customer_name, 
SUM(o.order_amount) as clv 
FROM orders o
JOIN customers as c
ON o.customer_id = c.customer_id
GROUP BY 1, 2 
```
**Q17. identify monthly sales trends by comparing each month total sales to the previous month**  
```sql
SELECT 
      EXTRACT(YEAR FROM order_date) as year,  
   EXTRACT(MONTH FROM order_date) as month, 
   SUM(order_amount) as total_sale, 
   LAG(SUM(order_amount), 1) OVER(ORDER BY EXTRACT(YEAR FROM order_date), EXTRACT(MONTH FROM order_date)) as prev_month_sale
FROM orders   
   
   GROUP BY 1,2
   ORDER BY 1,2 
```
**Q18.  evaluate rider efficiency by determining average delivery times and identifying those with the lowest and highest average**
```sql
WITH new_table 
AS(
SELECT 
d.rider_id, 
EXTRACT (EPOCH FROM (  d.delivery_time -   o.order_time  +
   CASE WHEN d.delivery_time < o.order_time THEN INTERVAL '1 day'
   ELSE  INTERVAL '0 day'
   END )) / 60  as delivery_took_time 
FROM orders o 
JOIN deliveries d 
ON o.order_id = d.order_id
WHERE d.delivery_status = 'Delivered'), 

riders_time 
AS 
(
SELECT 
rider_id,  
AVG(delivery_took_time)  avg_time
FROM new_table 
GROUP BY 1
)
SELECT
   ROUND(MIN(avg_time), 2),
   ROUND(MAX(avg_time), 2)
FROM riders_time    
```
**Q19. order Item Popularity : track the popularity of specific order items over time and identify seasonal demand spikes**   
```sql
SELECT 
order_item, 
seasons, 
COUNT(order_id) as total_orders
FROM
(SELECT
*, 
EXTRACT(MONTH FROM order_date) as month, 
CASE
WHEN EXTRACT(MONTH FROM order_date) BETWEEN 4 AND 6 THEN 'spring'
WHEN EXTRACT(MONTH FROM order_date) > 6 AND  EXTRACT(MONTH FROM order_date) < 9 THEN 'summer'
ELSE 'winter'
END as seasons  
FROM orders) as t1	 
GROUP BY 1,2
ORDER BY 1 DESC  
```







