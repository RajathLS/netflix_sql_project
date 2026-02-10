# Netflix Movies and TV Shows Data Analysis using SQL
![Netflix Logo](https://github.com/RajathLS/netflix_sql_project/blob/main/netflix.png)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives
- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Movies Dataset](https://github.com/RajathLS/netflix_sql_project/blob/main/netflix_titles.csv)

## Schema

```sql

CREATE TABLE netflix
(
show_id VARCHAR (6),
type VARCHAR (10),
title VARCHAR (150),
director VARCHAR (208),
casts VARCHAR (1000),
country VARCHAR (150),
date_added VARCHAR (50),
release_year INT,
rating VARCHAR (10),
duration VARCHAR (15),
listed_in VARCHAR (150),
description VARCHAR(250)
);

## Business Problems and Solutions

## 1. Count the number of Movies vs TV Shows
select count(*) as Total_Contetns ,type
from netflix
group by type;
	

## 2. Find the most common rating for movies and TV shows

SELECT
	type,
	rating
	from
(
	select
	type,
	rating,
	count(*),
	RANK() OVER(PARTITION BY type ORDER BY COUNT(*)DESC) AS ranking
  from netflix
  group by 1,2
) as t1
WHERE 
	ranking=1;

## 3. List all movies released in a specific year (e.g., 2020)

SELECT * from netflix where release_year='2020' and type='Movie';

## 4. Find the top 5 countries with the most content on Netflix

SELECT 
	COUNT(*) as TOTAL_CONTENT,
	UNNEST(STRING_TO_ARRAY(country,',')) as new_country   -- In DB a single cell contain multiple countries so we seprated each country using string fun and unnest will allocate each multiple country to new line
FROM netflix
GROUP BY new_country
ORDER BY TOTAL_CONTENT DESC
LIMIT 5;


--5. Identify the longest movie

SELECT * 
	FROM netflix 
	WHERE type='Movie' 
	AND duration =(SELECT MAX(duration) FROM netflix);


--6. Find content added in the last 5 years


SELECT *
	FROM netflix
	WHERE 
	TO_DATE(date_added,'MONTH DD,YYYY')>= CURRENT_DATE - INTERVAL '5 years';

--7. Find all the movies/TV shows by director 'Rajiv Chilaka'!

SELECT * 
	FROM netflix
	WHERE director ILIKE '%Rajiv Chilaka%';   -- USE LIKE OPERATOR BCZ DIRECTOR COLUMN CONTAINS MORE THAN ONE DIRECTOR. ILIKE---- CASE INSENSITIVE

	

--8. List all TV shows with more than 5 seasons

SELECT *
	FROM netflix 
	WHERE type='TV Show' 
	AND SPLIT_PART(duration,' ',1)> '5';

-- TO SEE HOW SPLIT_PART WORK
SELECT 
	SPLIT_PART('APPLE BANNANA CHERRY',' ',2)

--9. Count the number of content items in each genre

SELECT 
	UNNEST(STRING_TO_ARRAY(listed_in,',')) as genere,
	COUNT(show_id)as total_content
	FROM netflix
	GROUP BY genere;

--10.Find each year and the average numbers of content release in India on netflix.return top 5 year with highest avg content release!


SELECT 
	EXTRACT(YEAR FROM TO_DATE(date_added,'Month DD, YYYY')) AS release_year,
	COUNT(*) AS YEARLY_CONTENT,
	ROUND(COUNT(*)::numeric/(SELECT COUNT(*) FROM netflix WHERE country='India')::numeric * 100,2) as avg_content_per_year
from netflix 
	where country='India' 
	GROUP BY 1 ;
	
--11. List all movies that are documentaries

SELECT * 
	FROM netflix
	WHERE type='Movie'
	AND listed_in ILIKE '%documentaries';
	

--12. Find all content without a director

SELECT * FROM netflix WHERE director IS NULL ;


--13. Find how many movies actor 'Salman Khan' appeared in last 10 years!

SELECT *
	FROM netflix
	WHERE casts ILIKE '%Salman Khan%' AND 
	release_year > EXTRACT(YEAR FROM CURRENT_DATE)- 10;


--14. Find the top 10 actors who have appeared in the highest number of movies produced in India.

SELECT
	UNNEST(STRING_TO_ARRAY(casts,',')) as actors,
	COUNT(*) AS total_movies_acted
	FROM netflix
	WHERE country ILIKE '%India%'
	GROUP BY 1
	ORDER BY 2 DESC
	LIMIT 10;

--15.Categorize the content based on the presence of the keywords 'kill' and 'violence' in 
--the description field. Label content containing these keywords as 'Bad' and all other 
--content as 'Good'. Count how many items fall into each category.

SELECT
    CASE
        WHEN description ILIKE '%kill%' 
          OR description ILIKE '%violence%' 
        THEN 'Bad'
        ELSE 'Good'
    END AS content_category,
    COUNT(*) AS total_items
FROM netflix
GROUP BY content_category;


-- 16.Which content type drives the platform more: Movies or TV Shows?
SELECT 
    type,
    COUNT(*) AS total_content,
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 2) AS percentage
FROM netflix
GROUP BY type;

--17.Which countries should Netflix target for new content production?
SELECT 
    UNNEST(STRING_TO_ARRAY(country,',')) AS country,
    COUNT(*) AS total_content
FROM netflix
GROUP BY country
ORDER BY total_content DESC
LIMIT 10;


--18.Which year had the highest content growth compared to the previous year?

WITH yearly AS (
    SELECT release_year, COUNT(*) AS total
    FROM netflix
    GROUP BY release_year
)
SELECT 
    release_year,
    total,
    total - LAG(total) OVER (ORDER BY release_year) AS growth
FROM yearly
ORDER BY growth DESC
LIMIT 1;

--19.Which directors consistently deliver more content?
SELECT 
    director,
    COUNT(*) AS total_titles
FROM netflix
WHERE director IS NOT NULL
GROUP BY director
ORDER BY total_titles DESC
LIMIT 10;


--20.Do longer movies dominate the platform?
SELECT 
    CASE 
        WHEN CAST(SPLIT_PART(duration,' ',1) AS INT) <= 90 THEN 'Short'
        WHEN CAST(SPLIT_PART(duration,' ',1) AS INT) BETWEEN 91 AND 120 THEN 'Medium'
        ELSE 'Long'
    END AS movie_length_category,
    COUNT(*) AS total_movies
FROM netflix
WHERE type='Movie'
GROUP BY movie_length_category;

--21.Is violent content dominating the platform?
SELECT 
    CASE
        WHEN description ILIKE '%kill%' OR description ILIKE '%violence%'
        THEN 'Violent Content'
        ELSE 'Non-Violent Content'
    END AS content_type,
    COUNT(*) AS total_titles
FROM netflix
GROUP BY content_type;
