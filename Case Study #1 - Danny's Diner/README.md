# Case Study #1 - Danny's Diner

<img src="https://github.com/user-attachments/assets/637b1318-0483-4dd5-93b5-b9fec2982aad" width="500" height="500">

## Problem statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

## Entity Relationship Diagram
<img width="536" alt="image" src="https://github.com/user-attachments/assets/c0580227-bcb2-49bb-ad10-030710d1e0ff">

## Questions and solutions

**1. What is the total amount each customer spent at the restaurant?**
````sql
#--Method 1
select *
from menu
;

select *
from sales
;

#--I added a new column for pricing
alter table sales
add price int
;

update sales
set price = case product_id
when 1 then 10
when 2 then 15
when 3 then 12
else 'nil'
end;

select customer_id, sum(price) 
from sales
group by customer_id
;

#--Method 2 - using join
select customer_id, sum(menu.price)
from sales
join menu 
on sales.product_id = menu.product_id
group by customer_id
````
**2. How many days has each customer visited the restaurant?**
````sql
select customer_id, count(order_date)
from sales
group by customer_id
;

#--use distinct so that repetitive days are removed
select customer_id, count(distinct order_date)
from sales
group by customer_id
;
````
**3. What was the first item from the menu purchased by each customer?**
````sql
with cte_sales as (
select customer_id, order_date, product_name, 
rank () over (partition by customer_id order by order_date) as first_item,
row_number () over (partition by customer_id order by order_date) as first_item2,
dense_rank () over (partition by customer_id order by order_date) as first_item3
from sales
join menu 
on sales.product_id = menu.product_id
)
select customer_id, product_name
from cte_sales
where first_item3 = 1
# -- you can use firstitem 1 2 or 3 depending on what you want to achieve
;
````

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**
````sql
select product_name, count(sales.product_id) as mostpurchased
from sales
join menu 
on sales.product_id = menu.product_id
group by product_name
order by mostpurchased desc
limit 1
;
````

**5. Which item was the most popular for each customer?**
````sql
with cte_sale as (
select customer_id, product_name, count(sales.product_id),
rank () over (partition by customer_id order by count(sales.product_id)) as first_item,
row_number () over (partition by customer_id order by count(sales.product_id)) as first_item2
from sales
join menu 
on sales.product_id = menu.product_id
group by product_name, customer_id
)

select customer_id, product_name
from cte_sale
where first_item = 1
;
````

**6. Which item was purchased first by the customer after they became a member?**
````sql
with cte_sale as (
select sales.customer_id, order_date, product_name, join_date,
row_number () over (partition by customer_id order by order_date) as first_item2
from sales
join menu
on sales.product_id = menu.product_id
join members
on sales.customer_id = members.customer_id
where order_date >= join_date
)
select customer_id, product_name
from cte_sale
where first_item2 = 1
;
````

**7. Which item was purchased just before the customer became a member?**
````sql
with cte_sale as (
select sales.customer_id, order_date, product_name, join_date,
rank () over (partition by customer_id order by order_date desc) as first_item,
row_number () over (partition by customer_id order by order_date desc) as first_item2
from sales
join menu
on sales.product_id = menu.product_id
join members
on sales.customer_id = members.customer_id
where order_date < join_date
)
select customer_id, product_name
from cte_sale
where first_item2 = 1
;
````

**8. What is the total items and amount spent for each member before they became a member?**
````sql
select sales.customer_id, count(order_date) as total_items, sum(menu.price) as amount_spent 
from sales
join menu
on sales.product_id = menu.product_id
join members
on sales.customer_id = members.customer_id
where order_date < join_date
group by sales.customer_id
order by sales.customer_id 
;
````

**9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**
````sql
select customer_id, #menu.price, product_name,
sum(case  
when  product_name = 'sushi' then menu.price*10*2
else menu.price*10
end) as points
from sales
join menu
on sales.product_id = menu.product_id  
Group by customer_id
;
````

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi -  how many points do customer A and B have at the end of January?**
````sql
select * ,
case  
when  product_name = 'sushi' then menu.price*10*2
else menu.price*10
end as points
from sales
join menu
on sales.product_id = menu.product_id
join members
on sales.customer_id = members.customer_id
where order_date >= join_date
;
````

### BONUS QUESTION
**A. Recreate table with: customer_id, order_date, product_name, price, member (Y/N)**
````sql
select sales.customer_id, order_date, product_name, sales.price, 
case
when order_date >= join_date then 'Y'
else 'N'
end as 'member'
from sales
join menu
on sales.product_id = menu.product_id
left join members
on sales.customer_id = members.customer_id
;
````

**B. Recreate the table with: customer_id, order_date, product_name, price, member (Y/N) with ranking**
````sql
with cte_table as (
select sales.customer_id, order_date, product_name, sales.price, 
case
when order_date >= join_date then 'Y'
else 'N'
end as member_status
from sales
join menu
on sales.product_id = menu.product_id
left join members
on sales.customer_id = members.customer_id
)

select customer_id, order_date, product_name, price, member_status,
case 
when member_status = 'N' then null
else row_number () over (partition by customer_id, member_status order by customer_id) 
end as ranking
from cte_table
````




