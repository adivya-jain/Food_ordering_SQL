# Customer Sales Data Analysis
FOOD DELIVERING SQL
This document explains the SQL queries used for analyzing the Zomato customer sales data, including customer transactions, membership status, and product details. It also covers how to compute metrics such as total spending, points earned, and transaction rankings.

Prerequisites
Database Tables: The queries operate on the following tables:
users - Contains customer signup details.
sales - Records all sales transactions by customers.
goldusers_signup - Stores details of customers who signed up for Zomato Gold membership.
product - Holds information about the products available for purchase and their respective prices.
Queries Overview
1. Total Amount Spent by Each Customer
This query calculates the total amount spent by each customer on Zomato.

select a.userid, sum(b.price) total_amt_spent 
from sales a 
inner join product b on a.product_id = b.product_id
group by a.userid;

2. Number of Distinct Days Visited by Each Customer
This query counts the distinct days each customer made a transaction on Zomato.

select userid, count(distinct created_date) distinct_days 
from sales 
group by userid;

3. First Product Purchased by Each Customer
This query returns the first product purchased by each customer.

select * 
from (select *, rank() over (partition by userid order by created_date) rnk from sales) a 
where rnk = 1;


4. Most Purchased Product on the Menu & Number of Purchases
This query identifies the most purchased product on the menu and counts how many times it was purchased by all customers.

select userid, count(product_id) cnt 
from sales 
where product_id = (select top 1 product_id from sales group by product_id order by count(product_id) desc)
group by userid;


5. Most Popular Item for Each Customer
This query finds the most frequently purchased item for each customer.

select * 
from (select *, rank() over(partition by userid order by cnt desc) rnk from
(select userid, product_id, count(product_id) cnt from sales group by userid, product_id) a) b
where rnk = 1;

6. First Item Purchased After Becoming a Zomato Gold Member
This query retrieves the first product purchased by a customer after they joined the Zomato Gold program.

select * 
from (select c.*, rank() over (partition by userid order by created_date) rnk from
(select a.userid, a.created_date, a.product_id, b.gold_signup_date 
from sales a inner join goldusers_signup b on a.userid = b.userid and created_date >= gold_signup_date) c) d
where rnk = 1;

7. Item Purchased Just Before Becoming a Zomato Gold Member
This query identifies the last product a customer purchased before becoming a Zomato Gold member.

select * 
from (select c.*, rank() over (partition by userid order by created_date desc) rnk from
(select a.userid, a.created_date, a.product_id, b.gold_signup_date 
from sales a inner join goldusers_signup b on a.userid = b.userid and created_date <= gold_signup_date) c) d
where rnk = 1;


8. Total Orders and Amount Spent Before Becoming a Zomato Gold Member
This query calculates the total number of orders and the total amount spent by each customer before they became Zomato Gold members.

select userid, count(created_date) order_purchased, sum(price) total_amt_spent 
from (select c.*, d.price from
(select a.userid, a.created_date, a.product_id, b.gold_signup_date 
from sales a inner join goldusers_signup b on a.userid = b.userid and created_date <= gold_signup_date) c 
inner join product d on c.product_id = d.product_id) e
group by userid;


9. Points Collected by Each Customer Based on Product Purchases
This query calculates Zomato points earned based on product purchases, with different products having different point-earning rates.

select userid, sum(total_points) * 2.5 total_point_earned 
from (select e.*, amt / points total_points from 
(select d.*, case when product_id = 1 then 5 when product_id = 2 then 2 when product_id = 3 then 5 else 0 end as points 
from (select c.userid, c.product_id, sum(price) amt from 
(select a.*, b.price from sales a inner join product b on a.product_id = b.product_id) c 
group by userid, product_id) d) e) f 
group by userid;

This query also identifies the product for which most points have been earned.

select * from
(select *, rank() over (order by total_point_earned desc) rnk from
(select product_id, sum(total_points) total_point_earned from 
(select e.*, amt / points total_points from 
(select d.*, case when product_id = 1 then 5 when product_id = 2 then 2 when product_id = 3 then 5 else 0 end as points 
from (select c.userid, c.product_id, sum(price) amt from 
(select a.*, b.price from sales a inner join product b on a.product_id = b.product_id) c 
group by userid, product_id) d) e) f 
group by product_id) f) g 
where rnk = 1;

10. Points Earned in First Year of Zomato Gold Membership
This query computes the points earned by each customer in the first year after joining Zomato Gold, based on their spending.

select c.*, d.price * 0.5 total_points_earned 
from (select a.userid, a.created_date, a.product_id, b.gold_signup_date 
from sales a inner join goldusers_signup b on a.userid = b.userid and created_date >= gold_signup_date 
and created_date <= Dateadd(year, 1, gold_signup_date)) c 
inner join product d on c.product_id = d.product_id;

11. Rank All Transactions of Customers
This query ranks all transactions made by customers in the order they were created.

select *, rank() over (partition by userid order by created_date) rnk 
from sales;

12. Rank Transactions for Zomato Gold Members
This query ranks transactions for Zomato Gold members. Transactions made by non-Gold members are marked as "na."

select e.*, case when rnk = 0 then 'na' else rnk end as rnkk 
from (select c.*, cast((case when gold_signup_date is null then 0 else rank() over (partition by userid order by created_date desc) end) as varchar) as rnk 
from (select a.userid, a.created_date, a.product_id, b.gold_signup_date 
from sales a left join goldusers_signup b on a.userid = b.userid and created_date >= gold_signup_date) c) e;



The SQL queries provided for the Zomato customer sales data offer a detailed analysis of user behavior, transactions, and Zomato Gold membership impact. The key insights include:

Total spending and transaction activity: These queries allow tracking how much each customer spends and how frequently they make purchases, providing a clear view of customer engagement.
Product popularity: The analysis identifies the most purchased items and highlights which products contribute the most to customer points, giving valuable insights into product preferences.
Impact of Zomato Gold membership: The queries determine how Zomato Gold membership influences customer behavior by analyzing purchases before and after membership, and tracking points earned during the first year of membership.
Customer loyalty: The points system allows understanding of how customers accumulate rewards, while ranking transactions offers a view of customer purchase patterns over time.
