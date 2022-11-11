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




