# Netflix Movies and TV shows Data Analysis using SQL
![Netflix logo](https://github.com/neha0697/Netflix-SQL-Project/blob/main/logo.png)

## Objective
- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.


## Dataset
The data for this project is sourced from the Kaggle dataset:
- Dataset Link: [Movie Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

# Schema
```
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
```

## Business Problems and Solutions

### 1. Count the Number of Movies vs TV Shows
```
SELECT 
	type, 
	COUNT(*) AS total_content
FROM netflix
GROUP BY type;
```

### 2. Find the most common rating for movies and TV shows
```
WITH RankedRatings AS(
	SELECT
		type,
		rating,
		COUNT(*),
		RANK() OVER (PARTITION BY type ORDER BY COUNT(*) DESC) AS rnk
	FROM netflix
	GROUP BY type, rating
	)
SELECT 
	type,
	rating
FROM RankedRatings
WHERE rnk = 1;
```

### 3. List all movies released in a specific year (e.g., 2020)
```
SELECT * FROM netflix
WHERE
	type = 'Movie'
	AND
	release_year = 2020;
```

### 4. Find the top 5 countries with the most content on Netflix
```
SELECT
	TRIM(UNNEST(STRING_TO_ARRAY(Country,','))) AS new_country,
	COUNT(show_id) AS total_content
FROM netflix
GROUP BY 1
ORDER BY total_content DESC
LIMIT 5;
```

### 5. Identify the longest movie
```
SELECT
	title,
	CAST((substring(duration, 1, (length(duration)-4))) AS INTEGER) AS duration
FROM netflix
WHERE type = 'Movie' AND (substring(duration, 1, (length(duration)-4))) IS NOT NULL
ORDER BY 2 DESC
LIMIT 1;
```

### 6. Find content added in the last 5 years
```
SELECT 
	*,
	TO_DATE(date_added,'Month DD, YYYY') 
FROM netflix
WHERE (TO_DATE(date_added,'Month DD, YYYY')) >= CURRENT_DATE - INTERVAL '5 years';
```

### 7. Find all the movies/TV shows by director 'Rajiv Chilaka'
```
SELECT 
	type,
	title,
	director
FROM netflix
WHERE director ILIKE '%Rajiv Chilaka%';
```

### 8. List all TV shows with more than 5 seasons
```
SELECT *,
	SPLIT_PART(duration,' ',1) AS split_duration
	FROM netflix
WHERE
	type = 'TV Show'
	AND
	CAST((SPLIT_PART(duration,' ',1)) AS INTEGER) > 5
ORDER BY split_duration DESC;
```

### 9. Count the number of content items in each genre
```
SELECT 
	TRIM(UNNEST(STRING_TO_ARRAY(listed_in,','))) AS genre,
	COUNT(show_id) AS total_content
FROM netflix
GROUP BY 1
ORDER BY 2 DESC;
```

### 10. Find each year and the average numbers of content release in India on netflix. Return top 5 year with highest avg content release!
```
SELECT
	release_year,
	ROUND(AVG(cnt)::numeric) AS avg_numbers
FROM(
SELECT
	release_year,
	country,
	COUNT(show_id) as cnt
FROM netflix
GROUP BY 1,2
HAVING country ILIKE '%India%')
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
```

### 11. List all movies that are documentaries
```
SELECT * FROM netflix
WHERE
	type = 'Movie'
	AND
	listed_in ILIKE '%Documentaries%';
```

### 12. Find all content without a director
```
SELECT * FROM netflix
WHERE 
	director IS NULL;
```

### 13. Find how many movies actor 'Salman Khan' appeared in last 10 years!
```
SELECT
	COUNT(*) AS movie_count
FROM netflix
WHERE
	type = 'Movie'
	AND
	CASTS ILIKE '%Salman Khan%'
	AND
	release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```

### 14. Find the top 10 actors who have appeared in the highest number of movies produced in India.
```
SELECT 
	TRIM(UNNEST(STRING_TO_ARRAY(casts,','))) AS actors,
	COUNT(show_id) AS movie_count
FROM netflix
WHERE 
	type = 'Movie'
	AND
	country ILIKE '%India%'
GROUP BY 1;
```

### 15. Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field. Label content containing these keywords as 'Bad' and all other 
 content as 'Good'. Count how many items fall into each category.
 ```
SELECT 
	content_label,
	COUNT(*) AS total_count
FROM(
SELECT *,
CASE
	WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
	ELSE 'Good'
END AS content_label
FROM netflix
	)
GROUP BY content_label;
```

## Findings and Conclusion

- Content Distribution: The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
- Common Ratings: Insights into the most common ratings provide an understanding of the content's target audience.
- Geographical Insights: The top countries and the average content releases by India highlight regional content distribution.
- Content Categorization: Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.
  
This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.
