# DataLemur_solved_sql_queries

## 1.Data Science Skills
```
SELECT candidate_id FROM candidates
where skill in ('Python','Tableau','PostgreSQL')
GROUP BY candidate_id
having COUNT(*) >2
ORDER BY candidate_id;
```
## 2.Page With No Likes
```
select page_id from pages
where page_id not in (SELECT page_id FROM page_likes);
```
## 3.Unfinished Parts
```
SELECT distinct(part) FROM parts_assembly
where finish_date is null;
```
## 4.Laptop vs. Mobile Viewership
```
SELECT 
       CASE WHEN device_type = 'laptop_view' THEN 1 ELSE 0 END AS laptop_views,
       CASE when device_type IN ('tablet','phone') THEN 1 ELSE 0 end as mobile_views
FROM viewership;
```
## 5.Cities With Completed Trades
```
SELECT u.city, count(t.*) 
FROM trades t
join users u
on u.user_id = t.user_id
where status = 'Completed'
GROUP BY u.city
order by 2 desc
limit 3;
```
## 6.Duplicate Job Listings
```
with cte as
(SELECT company_id, title, description, 
ROW_NUMBER() OVER(PARTITION BY company_id, title, description) as RN
from job_listings) 
 SELECT count(company_id) as count_of_duplicate_jobs
 from cte 
 where RN != 1
 ```
 ## 7.Histogram of Tweets
```
 with cte as 
(select user_id, count(tweet_id) as tweet_bucket
FROM tweets
where tweet_date between '01/01/2022' and '12/31/2022' 
group by user_id)
select tweet_bucket,count(user_id)
from cte 
group by tweet_bucket
order by tweet_bucket 
```
## 8.LinkedIn Power Creators (Part 1)
```
SELECT profile_id
FROM personal_profiles pp
join company_pages cp
on employer_id = company_id	
where pp.followers > cp.followers
order by 1 
```
## 9.Spare Server Capacity
```
with cte as 
(Select datacenter_id, Sum(monthly_demand) as total_demand
from forecasted_demand fd
group by datacenter_id)
select d.datacenter_id, d.monthly_capacity - cte.total_demand as spare_capacity
from datacenters d
join cte 
on cte.datacenter_id = d.datacenter_id
order by 1
```

# 10.Average Post Hiatus (Part 1)
```
SELECT user_id,
EXTRACT(DAY FROM MAX(post_date)-MIN(post_date)) AS days_between
FROM posts
WHERE post_date BETWEEN  '01/01/2021' and '12/31/2021'
group by user_id
having count(*) >= 2
```

# 11.Teams Power Users
```
SELECT sender_id, count(message_id) as message_count
FROM messages
where sent_date BETWEEN '08/01/2022' and '08/31/2022' 
group by sender_id
order by 2 DESC
limit 2;
```
# 12. Average Review Ratings
```
SELECT DATE_PART('Month',submit_date) as month,
       product_id, 
       Round(Avg(stars),2) as avg_stars
 FROM reviews
group by 1,2 
order by 1,2
```
# 13. User's Third Transaction
```
with cte as
(SELECT user_id, spend,transaction_date,
row_number() over(PARTITION BY user_id order by transaction_date) as rn
FROM transactions)

select user_id,spend,transaction_date
from cte
where rn = 3
```
# 14. Second Day Confirmation
```
SELECT DISTINCT emails.user_id
FROM emails 
JOIN texts
  ON emails.email_id = texts.email_id
WHERE texts.action_date = emails.signup_date + INTERVAL '1 day'
  AND signup_action = 'Confirmed';
```  
# 15.App Click-through Rate (CTR)
```
SELECT app_id,
Round(100.0* 
Sum(CASE WHEN event_type = 'click' THEN 1 ELSE 0 END)/
    Sum(CASE WHEN event_type = 'impression' THEN 1 ELSE 0 END),2) as ctr 
FROM events
where date_part('Year',timestamp) = '2022'
group by app_id;
```
# 16. Final Account Balance
```
SELECT
  account_id, 
  sum(CASE
    WHEN transaction_type = 'Deposit' THEN amount
    ELSE -amount         
  END) AS balance_amount
FROM transactions
GROUP BY account_id
```
# 17.Compressed Mean
```
with cte as 
(select sum(item_count::DECIMAL*order_occurrences) as Total_items from items_per_order),
cte1 as
(SELECT sum(order_occurrences) as total_orders FROM items_per_order)
select Round(cte.total_items /cte1.total_orders,1) as mean
from cte, cte1
```
# 18. Compressed Mode
```
select item_count as mode1
from items_per_order 
where order_occurrences in 
(select max(order_occurrences) from items_per_order)
order by 1 asc;
```
# 19.ApplePay Volume
```
SELECT merchant_id, 
SUM(CASE WHEN LOWER(payment_method) = 'apple pay' THEN transaction_amount
ELSE 0 END) AS total_transaction
FROM transactions
GROUP BY merchant_id
ORDER BY total_transaction DESC;
```
# 20. Pharmacy Analytics (Part 1)
```
SELECT drug, total_sales-cogs as total_profit
FROM pharmacy_sales
order by 2 DESC
limit 3;
```
# 21. Pharmacy Analytics (Part 2)
```
SELECT manufacturer, count(drug) as drug_count, ABS(Sum(total_sales - cogs)) AS total_loss
FROM pharmacy_sales
where total_sales - cogs <= 0
group by manufacturer
order by 3 desc;
```
# 22. Pharmacy Analytics (Part 3)
```
SELECT manufacturer,
Concat('$',Round(sum(total_sales)/1000000), ' million') as sales
FROM pharmacy_sales
group by manufacturer
order by sum(total_sales) desc;
```
# 23. Patient Support Analysis (Part 1) [UnitedHealth SQL Interview Question]
```
with cte AS
(SELECT policy_holder_id,
count(case_id) as call_count
FROM callers
group by policy_holder_id
having count(case_id) >= 3)

select count(*)
from cte;
```
# 24. Patient Support Analysis (Part 2) [UnitedHealth SQL Interview Question]
```
with cte as  
(SELECT count(*) as unrecognized_call_count
FROM callers
where call_category = 'n/a' or call_category is null)

select 
round( 100.0*  unrecognized_call_count
/ (select count(*) from callers),1) as unrecog_call_percent
from cte 
group by unrecognized_call_count;
```
# 25. Highest-Grossing Items [Amazon SQL Interview Question]
```
with cte as 
(select category, product, SUM(spend) as total_spend
from product_spend 
where transaction_date >= '01/01/2022 12:00:00'
group by category, product),
xyz as 
(Select *,
rank() over (partition by category order by total_spend desc) as rw
from cte)

select category, product, total_spend
from xyz
where rw <= 2
ORDER BY category, rw
```
# 26.Sending vs. Opening Snaps [Snapchat SQL Interview Question]
```
with cte as
(SELECT user_id,
sum(case when activity_type = 'open' then time_spent else 0 end) as opening,
sum(case when activity_type = 'send' then time_spent else 0 end) as sending
FROM activities
group by user_id) 
select age_bucket, 
Round((sending / (opening + sending))*100.0,2) as send_per,
Round((opening / (opening + sending))*100.0,2) as open_per
from cte 
join age_breakdown
on cte.user_id = age_breakdown.user_id
order by age_bucket asc;
```
# 27. Odd and Even Measurements [Google SQL Interview Question]
```
with cte as (
select 
cast(measurement_time as date) as measurement_day, 
measurement_value, 
ROW_NUMBER() over (
partition by cast(measurement_time as date) 
order by measurement_time) as measurement_num 
from measurements
)

select measurement_day,
sum(case when measurement_num %2 <> 0 then measurement_value
else 0 end) as odd_sum,
sum(case when measurement_num % 2 = 0 then measurement_value
else 0 end) as even_sum
from cte
group by measurement_day;
```
# 28. Signup Activation Rate [TikTok SQL Interview Question]

```
with cte as (
select
SUM(case when signup_action = 'Confirmed' then 1 else 0 end) as Conf,
Sum(Case when signup_action = 'Not Confirmed' or signup_action = 'Confirmed' 
then 1 else 0 end) as total
FROM texts)

Select Round(conf * 1.0/total,2)
from cte
```
# 29. Histogram of Users and Purchases [Walmart SQL Interview Question]

```
with cte as 
(select *,
rank() over(partition by user_id order by transaction_date desc) as rnk
from user_transactions)

select transaction_date, user_id, count(*) as pur
from cte 
where rnk = 1
group by transaction_date, user_id
```
# 30. Y-on-Y Growth Rate [Wayfair SQL Interview Question]
```
with cte AS 
(SELECT Extract(year from transaction_date) as year_ , product_id,
spend as current_spend,
lag(spend) over(partition by product_id order by Extract(year from transaction_date)) as prev
from user_transactions )

select * ,
Round(((current_spend - prev)/prev) * 100,2) as yoy
from cte
```

# 31. Patient Support Analysis (Part 3) [UnitedHealth SQL Interview Question]
```
with cte as 
(SELECT policy_holder_id, call_received,
lag(call_received) 
over(partition by policy_holder_id order by call_received) as prev_call
FROM callers
),
cte2 as 
(select *, 
EXTRACT(EPOCH FROM call_received - prev_call)/(24*60*60) as time_diff 
from cte)

SELECT COUNT(DISTINCT policy_holder_id) AS patient_count
FROM cte2
WHERE time_diff <= 7
```








