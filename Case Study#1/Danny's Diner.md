# Case Study 1: Danny's Diner
![image](https://github.com/user-attachments/assets/f91efc4e-73b9-407b-9865-b495a0a09dad)
# Case Study & Bonus Questions
1) What is the total amount each customer spent at the restaurant?  
2) How many days has each customer visited the restaurant?  
3) What was the first item from the menu purchased by each customer?  
4) What is the most purchased item on the menu and how many times was it purchased by all customers?  
5) Which item was the most popular for each customer?  
6) Which item was purchased first by the customer after they became a member?  
7) Which item was purchased just before the customer became a member?  
8) What is the total items and amount spent for each member before they became a member?  
9) If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?  
10) Join all the things  
11) Rank all the things

## Q1: What is the total amount each customer spent at the restaurant?
#### SQL Code:
``` sql
select s.customer_id, sum(m.price) as amt_spent
from sales s
join menu m
on s.product_id = m.product_id
group by customer_id;
```
#### Output:
![image](https://github.com/user-attachments/assets/9d063a2b-183a-487f-b797-531edd24d2de)

## Q2: How many days has each customer visited the restaurant?
#### SQL Code:
``` sql
select customer_id, count(distinct order_date) as visit_days_count
from sales
group by customer_id;
```
#### Output:
![image](https://github.com/user-attachments/assets/eb88fb22-c492-453e-85a0-f289639c4dc4)


## Q3: What was the first item from the menu purchased by each customer?
#### SQL Code:
``` sql
with cte as 
(
select s.customer_id, m.product_name, dense_rank() over(partition by s.customer_id order by s.order_date) as rank_num
from sales s
join menu m
on s.product_id = m.product_id
)
select customer_id, product_name
from cte
where rank_num = 1
group by customer_id, product_name;
```
#### Output:
![image](https://github.com/user-attachments/assets/432e914f-9590-4112-8596-794ea9d32231)


## Q4: What is the most purchased item on the menu and how many times was it purchased by all customers?
#### SQL Code:
``` sql
select m.product_name,count(s.product_id) as order_count 
from sales s
join menu m
on s.product_id = m.product_id
group by m.product_name
order by order_count desc;
```
#### Output:
![image](https://github.com/user-attachments/assets/5e135194-d415-4b87-902c-a097e6ae72e0)


## Q5:Which item was the most popular for each customer?
#### SQL Code:
``` sql
with sample as 
(select s.customer_id,m.product_name,count(s.product_id)as order_count,
rank() over(partition by customer_id order by count(product_name) desc)  as rank_num
from sales s
join menu m
on s.product_id = m.product_id
group by s.customer_id,m.product_name)

select customer_id,product_name, order_count
from sample
where rank_num =1;
```
#### Output:
![image](https://github.com/user-attachments/assets/8c4a0c59-eb33-436f-91a4-61da910869ef)


## Q6: Which item was purchased first by the customer after they became a member?
#### SQL Code:
``` sql
with diner_info as 
(select s.customer_id, s.product_id, order_date, m.product_name, mem.join_date,
rank() over(partition by customer_id order by s.order_date desc) as rank_num
from sales s
join menu m on s.product_id = m.product_id
join members mem on s.customer_id = mem.customer_id
where order_date < join_date)

select customer_id, product_name, order_date, join_date
from diner_info
where rank_num =1;
```
#### Output:
![image](https://github.com/user-attachments/assets/c1083ae1-26dd-4b42-b2dc-b93d2a0ec880)


## Q7: Which item was purchased just before the customer became a member?
#### SQL Code:
``` sql
with diner_info as 
(select s.customer_id, s.product_id, order_date, m.product_name, mem.join_date,
rank() over(partition by customer_id order by s.order_date desc) as rank_num
from sales s
join menu m on s.product_id = m.product_id
join members mem on s.customer_id = mem.customer_id
where order_date < join_date)

select customer_id, product_name, order_date, join_date
from diner_info
where rank_num =1;
```
#### Output:
![image](https://github.com/user-attachments/assets/a303697a-0bd2-4deb-b525-4867bd732ff8)


## Q8: What is the total items and amount spent for each member before they became a member?
#### SQL Code:
``` sql
select s.customer_id,count(s.product_id) as total_items, sum(m.price) as amount_spent
from sales s
join menu m on s.product_id = m.product_id
join members mem on s.customer_id = mem.customer_id
where s.order_date < mem.join_date
group by s.customer_id
order by customer_id;
```
#### Output:
![image](https://github.com/user-attachments/assets/3e473936-79eb-4c4c-a7d5-d89c0e2363af)


## Q9: If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
#### SQL Code:
``` sql
select customer_id,sum(case when s.product_id = 1 then 20*m.price 
else m.price*10 end) as sushi_bonus
from menu AS m
INNER JOIN sales AS s ON m.product_id = s.product_id
GROUP BY customer_id
ORDER BY customer_id;
```
#### Output:
![image](https://github.com/user-attachments/assets/baec6d9a-7afe-445f-af90-0ff600771214)


## Bonus:
## Join all the things
We are creating a table includes customer IDs, order dates, product names, prices, and membership status. It uses left joins to combine data from the sales, menu, and members tables, indicating membership status as 'Y' if the order date is on or after the customer's join date, and 'N' otherwise. 
``` sql
select s.customer_id, s.order_date, m.product_name, m.price,
if(order_date >= join_date, 'Y','N') as member
from sales s
left join menu m on s.product_id = m.product_id
left join members mem on s.customer_id = mem.customer_id;
```
#### Output:
![image](https://github.com/user-attachments/assets/9544c436-4fc3-447d-b00a-f394cbd6bf4e)


## Rank all the things
We are creating a table where Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.
``` sql
with data_table as 
(select s.customer_id, s.order_date, m.product_name, m.price, 
if(s.order_date >= mem.join_date, 'Y','N') as member
from sales s
left join menu m on s.product_id = m.product_id
left join members mem on s.customer_id = mem.customer_id)

select *, if(member='N',NULL,dense_rank() over(partition by s.customer_id,member order by s.order_date)) as ranking
from data_table;
```
#### Output:
![image](https://github.com/user-attachments/assets/e9b007c0-8171-48f6-abbb-55dcb63fce6d)

