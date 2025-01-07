# NETFLIX DATA ANALYSIS USING SQL
![NETFLIX LOGO](https://github.com/shambhavimishr-a/netflix_sql_pr/blob/d7a6816ed1d587933ebd007c409b5fb42463c42e/download.png)

##OBJECTIVES

--1)Count number of Movies vs Tv shows
select type, count(*) as number_content
from netflix
group by type; 


--2)Find the most common ratings for movies and tv shows
select 
type,rating
from netflix;

select
type,rating
from
	(select type, rating,
	count(*),
	rank() over(partition by type order by count(*) desc) as ranking
	from netflix
	group by 1,2
	) as t1
where
ranking =1

--3) List all movies released in a specific year

select type,release_year from netflix
where type='Movie'
and 
release_year=2020

--4)Find top 5 countries with most content on netflix

select
unnest(string_to_array(country,',')) as new_country,
count(show_id) as total_content
from netflix
group by 1
order by 2 desc
limit 5
----------------------------------------------------------------
select 
unnest(string_to_array(country,',')) as new_country
from netflix


--5)Identify the longest movie
select type,duration from netflix
where
type='Movie' and duration=(select max(duration) from netflix)

--6)Find content added in the last 5 years
select *
from netflix
where
to_date(date_added,'Month DD,YYYY') >= current_date-interval '5 years'
--select current_date-interval '5 years'

--7)Find all the movies/tv shows directed by "Rajiv Chilaka"
select type, title, director 
from netflix 
where director ilike '%Rajiv Chilaka%'

--8)List all tv shows with more than 5 seasons
--split_part()
--select 
--split_part('Apple cherry banana',' ',3)
select
type, duration 
from netflix 
where
type='TV Show'
and
split_part(duration,' ',1)::numeric > 5

--9)Count the number of content items in each genre
select 
listed_in,
show_id,
unnest(string_to_array(listed_in,','))
from netflix
-----------------------------------------------------------
select 
unnest(string_to_array(listed_in,',')) as genre,
count(show_id)
from netflix
group by 1

--10)Find each year and the average number of content released by India . 
	--Return top 5 year with highest content release.
	--SELECT COUNT(*) FROM netflix WHERE country='India'
select 
extract(YEAR FROM to_date(date_added,'Month DD,YYYY')) as YEAR,
COUNT(*)as yearly_content,
round(
COUNT(*)::numeric/(SELECT COUNT(*) FROM netflix WHERE country='India')::numeric * 100 
,2)as avg_content_per_year
from netflix
where country='India'
GROUP BY 1

--11)List all movies that are documentaries.
select * from netflix
where
listed_in ilike '%documentaries%'

--12)Find all the content without a drector.
select * from netflix
where
director is null

--13)Find how many movies actor 'Salman Khan' appeared in last 10 years
select * from netflix
where
casts ilike'%salman khan%'
and release_year > extract(year from CURRENT_DATE)-10

--14) Find top 10 actors who have appeared in highest number of movies produced in India.
SELECT
UNNEST(STRING_TO_ARRAY(casts,',')) as actors,
count(*) as total_content
from netflix
where country ilike '%india%'
group by 1
order by 2 desc
limit 10

--15)Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field.
--Label these content keywords as BAD and all other as GOOD.
--Count how many items fall into each category.

with new_table
as
(
select * ,
case when
	description ilike'%kill%' or 
	description ilike'%violence%'
	then 'Bad_Content'
	else 'Good_Content'
end category
from netflix
)
select 
category,
count(*) as total_content
from new_table
group by 1


