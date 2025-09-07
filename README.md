
# 🎬 Netflix SQL Data Analysis Project

This project involves performing data analysis on a **Netflix dataset** using **SQL**. The goal was to clean, transform, and explore structured data to uncover meaningful business insights about Netflix’s global content library, including content type trends, popular genres, director/actor involvement, regional content breakdown, and more.

---

## 📁 Dataset Overview

The dataset was sourced from **Kaggle** and originally scraped from **Netflix’s official content catalog**. It contains metadata about Netflix's movies and TV shows, such as title, director, cast, country, date added, duration, release year, rating, genre, and description. This dataset closely resembles what a real-world content management or recommendation system might deal with.

---

## 🧰 Tools & Technologies Used

* **PostgreSQL / MySQL** – for querying and data manipulation
* **SQL** – for data cleaning, exploration, and business analysis
* **Kaggle** – as the dataset source
* **Git & GitHub** – for version control and project sharing

---

## 🧾 Database Schema

```sql
CREATE TABLE netflix
(
	show_id	    VARCHAR(5),
	type        VARCHAR(10),
	title	    VARCHAR(250),
	director    VARCHAR(550),
	casts	    VARCHAR(1050),
	country	    VARCHAR(550),
	date_added	VARCHAR(55),
	release_year	INT,
	rating	    VARCHAR(15),
	duration	VARCHAR(15),
	listed_in	VARCHAR(250),
	description VARCHAR(550)
);
```

---

## 💼 Business Questions & SQL Solutions

Below are 15 real-world business questions answered using SQL, each followed by its solution:

---

### **Q1. Count the number of Movies vs TV Shows**

```sql
SELECT 
    type,
    COUNT(*) AS total_count
FROM netflix
GROUP BY type;
```

---

### **Q2. Find the most common rating for movies and TV shows**

```sql
WITH RatingCounts AS (
    SELECT 
        type,
        rating,
        COUNT(*) AS rating_count
    FROM netflix
    GROUP BY type, rating
),
RankedRatings AS (
    SELECT 
        type,
        rating,
        rating_count,
        RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
    FROM RatingCounts
)
SELECT 
    type,
    rating AS most_frequent_rating
FROM RankedRatings
WHERE rank = 1;
```

---

### **Q3. List all movies released in a specific year (e.g., 2020)**

```sql
SELECT * 
FROM netflix
WHERE release_year = 2020;
```

---

### **Q4. Find the top 5 countries with the most content on Netflix**

```sql
SELECT * 
FROM (
    SELECT 
        UNNEST(STRING_TO_ARRAY(country, ',')) AS country,
        COUNT(*) AS total_content
    FROM netflix
    GROUP BY 1
) AS country_count
WHERE country IS NOT NULL
ORDER BY total_content DESC
LIMIT 5;
```

---

### **Q5. Identify the longest movie**

```sql
SELECT * 
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC
LIMIT 1;
```

---

### **Q6. Find content added in the last 5 years**

```sql
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```

---

### **Q7. Find all the movies/TV shows by director 'Rajiv Chilaka'**

```sql
SELECT *
FROM (
    SELECT *,
           UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name
    FROM netflix
) AS director_data
WHERE director_name = 'Rajiv Chilaka';
```

---

### **Q8. List all TV shows with more than 5 seasons**

```sql
SELECT *
FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```

---

### **Q9. Count the number of content items in each genre**

```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre,
    COUNT(*) AS total_content
FROM netflix
GROUP BY genre
ORDER BY total_content DESC;
```

---

### **Q10. Find each year and the average number of content releases in India. Return top 5 years with highest average release.**

```sql
SELECT 
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::NUMERIC / 
        (SELECT COUNT(*) FROM netflix WHERE country = 'India')::NUMERIC * 100,
        2
    ) AS avg_release_percentage
FROM netflix
WHERE country = 'India'
GROUP BY release_year
ORDER BY avg_release_percentage DESC
LIMIT 5;
```

---

### **Q11. List all movies that are documentaries**

```sql
SELECT *
FROM netflix
WHERE listed_in ILIKE '%Documentaries%'
  AND type = 'Movie';
```

---

### **Q12. Find all content without a director**

```sql
SELECT *
FROM netflix
WHERE director IS NULL OR director = '';
```

---

### **Q13. Find how many movies actor 'Salman Khan' appeared in over the last 10 years**

```sql
SELECT *
FROM netflix
WHERE casts ILIKE '%Salman Khan%'
  AND release_year >= EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```

---

### **Q14. Find the top 10 actors who have appeared in the highest number of movies produced in India**

```sql
SELECT 
    TRIM(UNNEST(STRING_TO_ARRAY(casts, ','))) AS actor,
    COUNT(*) AS movie_count
FROM netflix
WHERE country = 'India'
  AND type = 'Movie'
  AND casts IS NOT NULL
GROUP BY actor
ORDER BY movie_count DESC
LIMIT 10;
```

---

### **Q15. Categorize content as 'Good' or 'Bad' based on keywords 'kill' or 'violence' in the description. Count how many items fall into each category.**

```sql
SELECT 
    category,
    type,
    COUNT(*) AS content_count
FROM (
    SELECT *,
           CASE 
               WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
               ELSE 'Good'
           END AS category
    FROM netflix
) AS categorized_content
GROUP BY category, type
ORDER BY type;
```

---

## 🧹 Data Cleaning & Transformation Highlights

* **Normalized** multi-value fields (`director`, `casts`, `listed_in`) using `UNNEST` and `STRING_TO_ARRAY`
* Parsed `duration` strings to extract numeric values (for season or minute count)
* Converted textual dates to date format using `TO_DATE()`
* Filtered and handled **NULLs** and **missing directors**
* Used **window functions**, **CTEs**, and **ILIKE** for flexible querying

---

## 📊 Key Insights

* Movies are more frequent than TV shows, with different rating and genre distributions.
* USA, India, UK, Canada, and Japan contribute most to Netflix’s catalog.
* Some actors and directors like **Rajiv Chilaka** and **Salman Khan** appear prominently.
* Keywords like **"kill"** and **"violence"** appear in a notable percentage of content descriptions.
* Genres like **Documentaries**, **Comedies**, and **Dramas** dominate the listings.

---

## 🚀 How to Run the Project

1. Clone this repository.
2. Set up a PostgreSQL or MySQL database.
3. Create the `netflix` table using the provided schema.
4. Import the dataset from Kaggle.
5. Run the SQL queries from `netflix_analysis.sql`.
6. View and analyze results or extend with your own queries.

---

## 📂 Files Included

* `netflix_analysis.sql` – SQL queries solving all 15 business questions
* `netflix_data.csv` – Dataset (or link to Kaggle source)
* `README.md` – Project documentation and summary

---

## 📚 Learning Outcomes

* Real-world application of **SQL for business intelligence**
* Gained experience with **data wrangling, filtering, parsing**, and **text processing**
* Developed insights into content strategy and performance on streaming platforms
* Improved skills with **window functions, CTEs**, and **data cleaning techniques**


