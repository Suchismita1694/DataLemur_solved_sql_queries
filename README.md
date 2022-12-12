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

SELECT
  account_id, 
  sum(CASE
    WHEN transaction_type = 'Deposit' THEN amount
    ELSE -amount         
  END) AS balance_amount
FROM transactions
GROUP BY account_id

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











