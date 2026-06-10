# walmart_

drop table if exists walmart_table;

create table walmart_table(
invoice_id float	,
Branch	varchar(50),
City varchar(50),
category	varchar(100),
unit_price	float,
quantity	float,
date date	,
time time	,
payment_method	varchar(50),
rating float	,
profit_margin float	,
total float

)

select * from walmart_table;


--Q.1 Find different payment method and number of transactions, number of qty sold
select payment_method , count(invoice_id) as no_of_transactions,
sum(quantity) as total_quatity_sold  from walmart_table
group by payment_method
order by 3 desc;


-- 2  Identify the highest-rated category in each branch, displaying the branch, category AVG RATING
select * from 
(select  branch , category , avg(rating) as avg_rating ,
rank() over (partition by branch order by avg(rating) desc) as rank
from walmart_table
group by 1 ,  2)
where rank =1;


-- Q.3 Identify the busiest day for each branch based on the number of transactions

select * from (
select branch , to_char(date , 'Day') as day , count(*) as transactions,
rank() over(partition by branch order by count(*)  desc) as rank
from walmart_table
group by 1,2)
where rank = 1
order by transactions desc;

-- Q. 4 
-- Calculate the total quantity of items sold per payment method. List payment_method and total_quantity.


select payment_method ,count(*) as transactions ,  sum(quantity) as total_quantity from walmart_table
group by 1
order by 2 desc;


-- Q.5
-- Determine the average, minimum, and maximum rating of category for each city. 
-- List the city, average_rating, min_rating, and max_rating.

select category, city ,  min(rating) as min_rating , 
max(rating) as max_rating ,
avg(rating) as avg_rating  from walmart_table
group by category , city ;


-- Q.6
-- Calculate the total profit for each category by considering total_profit as
-- (unit_price * quantity * profit_margin). 
-- List category and total_profit, ordered from highest to lowest profit.


select * from walmart_table;

select category , 
round(sum(unit_price * quantity * profit_margin)::numeric,2) as total_profit 
from walmart_table
group by 1
order by 2 desc;


-- Q.7
-- Determine the most common payment method for each Branch. 
-- Display Branch and the preferred_payment_method.

select * from (
select  branch , payment_method  , count(*) as total_transations,
rank() over(partition by branch order by count(payment_method) desc) as rank from walmart_table
group by 1 , 2
order by 3 desc)
where rank = 1






WITH cte 
AS
(SELECT 
	branch,
	payment_method,
	COUNT(*) as total_trans,
	RANK() OVER(PARTITION BY branch ORDER BY COUNT(*) DESC) as rank
FROM walmart_table
GROUP BY 1, 2
)
SELECT *
FROM cte
WHERE rank = 1


-- Q.8
-- Categorize sales into 3 group MORNING, AFTERNOON, EVENING 
-- Find out each of the shift and number of invoices

select  
CASE 
	WHEN extract(hour from time) <12 then 'Morning'
	when extract(hour from time) between 12 and 17 then 'Afternoon'
	else 'Evening'
	end as day_slot , 
	count(*) as invoices 
	from walmart_table
	group by 1;


-- #9 Identify 5 branch with highest decrease ratio in 
-- revevenue compare to last year(current year 2023 and last year 2022)

-- rdr == last_rev-cr_rev/ls_rev*100


select branch , extract(year from date) as year from walmart_table;

with revenue_2022 as 
(
select branch , sum(total) as total_revenue from walmart_table
where extract(year from date) = 2022
group by 1)  ,

revenue_2023 as 
(
select branch , sum(total) as total_revenue from walmart_table
where extract(year from date) = 2023
group by 1) 


select ly.branch , ly.total_revenue as revenue_2022, cy.total_revenue as revenue_2023 , 
round(((ly.total_revenue - cy.total_revenue)::numeric /ly.total_revenue :: numeric) *100,2)  as rev_dec_ratio  
from revenue_2022 as ly
join
revenue_2023 as cy
on ly.branch = cy.branch

where ly.total_revenue > cy.total_revenue
order by  rev_dec_ratio desc
limit 5;






