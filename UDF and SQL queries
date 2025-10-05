--UDF

create or replace table yelp_reviews (review_text variant)
select * from yelp_reviews;
CREATE or replace table yelp_businesses (business_text variant)
select * from yelp_businesses limit 10;

create or replace table tbl_yelp_reviews as 
select  review_text:business_id::string as business_id 
,review_text:date::date as review_date 
,review_text:user_id::string as user_id
,review_text:stars::number as review_stars
,review_text:text::string as review_text
,analyze_sentiment(review_text) as sentiments
from yelp_reviews 


create or replace table tbl_yelp_businesses as 
select business_text:business_id::string as business_id
,business_text:name::string as name
,business_text:city::string as city
,business_text:state::string as state
,business_text:review_count::string as review_count
,business_text:stars::number as stars
,business_text:categories::string as categories
from yelp_businesses limit 100

select * from tbl_yelp_reviews limit 10;
select * from tbl_yelp_businesses limit 10;

--find the no of businesses in each category
with cte as (
select business_id, trim(A.value) as category from tbl_yelp_businesses, lateral split_to_table(categories,',') A
)
select category, count(*) as no_of_business from cte
group by 1 order by 2 desc.

--find the top 10 users who have reviewed the most businessess in the 'restaurants' category

select r.user_id, count(distinct b.business_id ) 
from tbl_yelp_reviews r 
inner join tbl_yelp_businesses b on r.business_id=b.business_id
where b.categories ilike '%restaurant%'
group by 1
order by 2 desc
limit 10

--find the most popular category of businesses based on the no of reviews
with cte as (
select business_id, trim(A.value) as category from tbl_yelp_businesses, lateral split_to_table(categories,',') A
)

select category, count(*) from cte inner join tbl_yelp_reviews r on cte.business_id=r.business_id
group by 1
order by 2 desc limit 10


--find the month with the highest numbre of reviews.

Select month(review_date) as review_month, count(*) from tbl_yelp_reviews
group by 1
order by 2 desc

--find percentage of 5 star reviews foreach business.
select b.business_id, b.name,count(*) as no_of_reviews, 
sum(case when r.review_stars=5 then 1 else 0 end) as star_reviews,
star_reviews*100/no_of_reviews as percent_star
from tbl_yelp_reviews r 
inner join tbl_yelp_businesses b on r.business_id=b.business_id
group by 1,2

--find the top 5 most reviewed businesses in each city.
with cte as (
select b.city, b.business_id, b.name,count(*) as no_of_reviews, 
from tbl_yelp_reviews r 
inner join tbl_yelp_businesses b on r.business_id=b.business_id
group by 1,2, 3
)
select *
from cte
qualify row_number() over(partition by city order by no_of_reviews desc) <=5

--find the average rating of biusinesses that have at least 100 reviews.

select b.business_id, b.name,count(*) as no_of_reviews, 
avg(review_stars) as avg_rating
from tbl_yelp_reviews r 
inner join tbl_yelp_businesses b on r.business_id=b.business_id
group by 1,2
having count(*)>=100

--list the top 10 users who have written the most reviews along with the business they reviewed.

with cte as (select r.user_id,count(*) as no_of_reviews, 
avg(review_stars) as avg_rating
from tbl_yelp_reviews r 
inner join tbl_yelp_businesses b on r.business_id=b.business_id
group by 1
order by 2
limit 10
)
select user_id, business_id from tbl_yelp_reviews
where user_id in (select user_id from cte)
group by 1, 2
order by 1

--top 10 business wuth highest positive sentiment reviews.

select b.business_id, b.name,count(*) as no_of_reviews, 
from tbl_yelp_reviews r 
inner join tbl_yelp_businesses b on r.business_id=b.business_id
where sentiments='Positive'
group by 1,2
order by 3 desc
limit 10
