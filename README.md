# Uncovering-The-Worlds-Oldest-Businesses
Explore the world’s oldest businesses with PostgreSQL. Combine business, country, and category data to find the oldest company per continent, count missing records, and identify enduring categories. Clear steps, reproducible queries, and simple explanations guide this end-to-end historical analysis.
<br>
<br>

## PROJECT OVERVIEW

Some businesses in the world have lasted for centuries, surviving wars, political changes, and economic crises. A prime example is Staffelter Hof Winery in Germany, which dates back to the year 862 and has endured through the rise and fall of dynasties, empires, and modern conflicts. This project investigates what characteristics allow a business to last over such long periods.

The dataset, compiled by BusinessFinancing.co.uk, includes information on the oldest still-operating businesses in most countries worldwide. While data is stored in separate files for businesses, countries, and categories, we need to combine them to perform meaningful analysis.

The main idea of this project is to explore the relationship between business longevity and factors such as location, industry, and missing regional information. By leveraging SQL queries in PostgreSQL, we will join these datasets, extract insights, and attempt to answer questions about which types of businesses last longest and where they are located.

<img width="340" height="340" alt="image" src="https://github.com/user-attachments/assets/ce67398b-d6c4-46dd-8823-160e08151fd9" />
<br>
<br>

## OBJECTIVE / KEY QUESTIONS

* What is the oldest business on each continent?
* How many countries per continent lack data on the oldest businesses?
* Which business categories are best suited to last many years, and on what continent are they?
<br>

## DATASET
* **Source**: BusinessFinancing.co.uk (cleaned dataset)
* **Table**: `businesses`/`new_businesses`, `countries`, `categories`
* **Structure**:

  A. `businesses`/`new_businesses` Table
     | Column         | Description                                 |
     | -------------- | ------------------------------------------- |
     | business       | Name of the business (varchar)              |
     | year\_founded  | Year the business was founded (int)         |
     | category\_code | Code for the business category (varchar)    |
     | country\_code  | ISO 3166-1 three-letter country code (char) |


  B. `countries` Table
     | Column        | Description                                    |
     | ------------- | ---------------------------------------------- |
     | country\_code | ISO 3166-1 three-letter country code (varchar) |
     | country       | Name of the country (varchar)                  |
     | continent     | Name of the continent (varchar)                |


  C. `categories` Table
     | Column         | Description                                    |
     | -------------- | ---------------------------------------------- |
     | category\_code | Code for the business category (varchar)       |
     | category       | Description of the business category (varchar) |

* **Sample Preview:**
  
```
sql

SELECT * FROM businesses LIMIT 10;
```
<img width="1000" height="500" alt="image" src="https://github.com/user-attachments/assets/081c1913-d4ae-4b95-802d-5318c5d5334b" />

```
sql

SELECT * FROM countries LIMIT 10;
```
<img width="1218" height="431" alt="image" src="https://github.com/user-attachments/assets/05b25e1e-e3fc-4ed9-8a9b-68c7365267d0" />

```
sql

SELECT * FROM categories LIMIT 10;
```
<img width="1213" height="413" alt="image" src="https://github.com/user-attachments/assets/3c6afdef-a2ed-4a28-aa2f-eaab87d5bd75" />

Previewing the first rows gives us an immediate sense of the dataset’s structure and content. It helps validate data quality, understand the variety of industries, and confirm whether columns align properly before joining tables. This makes analysis much smoother and ensures queries target the correct fields.
<br>
<br>

## TOOLS AND SKILLS USED

* **SQL (PostgreSQL) concepts found in the code**: `INNER JOIN, LEFT JOIN, USING(), MIN(), COUNT(), GROUP BY, ORDER BY, WHERE`, Subqueries, `UNION ALL`, `LIMIT`.
* **Techniques used:** 
  - Data preview and schema profiling to understand content and quality
  - Data integration across tables to enrich business records with country and continent info
  - Gap analysis to count missing entries by region
  - Category-level longevity analysis using the earliest founding years
  - Stepwise validation with intermediate checks (e.g., oldest year per continent before attaching names)
<br>

## ANALYSIS & APPROACH

### 1. Exploring Tables

```
sql

SELECT * FROM businesses;
```
<img width="1000" height="500" alt="image" src="https://github.com/user-attachments/assets/f8210580-11dd-49d5-bdec-dabcefcc12a3" />

```
sql

SELECT * FROM countries;
```
<img width="1000" height="500" alt="image" src="https://github.com/user-attachments/assets/9e8affde-472a-4bb2-b590-9f2b5dae62a8" />

```
sql

SELECT * FROM categories;
```

These queries fetch the first few rows of each table - `businesses`, `countries`, and `categories`. The purpose is to get an initial understanding of the data, such as what information is available, the types of values stored, and whether there are any obvious missing or unusual entries. Previewing the data helps in planning the analysis by showing how `businesses` are linked to `countries` and `categories`, which is essential for combining tables later in queries. It provides a real-world view of what we are working with before performing calculations or aggregations.

<img width="1000" height="500" alt="image" src="https://github.com/user-attachments/assets/5a2c55f7-3843-4b10-b767-020f2ab3063f" />



### 2.  Schema Exploration

```
sql

SELECT 
    table_name, 
    column_name, 
    data_type, 
    is_nullable,
    character_maximum_length as max_len 
FROM information_schema.columns 
WHERE table_name IN ('businesses', 'countries', 'categories') 
ORDER BY table_name, ordinal_position;
```

This profiles the structure of each table. Knowing data types, nullability, and max lengths tells us which columns are safe to aggregate (e.g., year_founded as an integer), which fields may be missing and need careful joins or filters, and where to expect short codes vs. longer text. With this clarity, our joins and groupings are less likely to break or produce misleading results.

<img width="1214" height="387" alt="image" src="https://github.com/user-attachments/assets/3d6c9ad1-4dd3-45eb-8310-67ff75ba4544" />



### 3. Oldest Year Founded by Continent

```
sql

SELECT 
    MIN(b.year_founded)
FROM businesses AS b
INNER JOIN countries AS c
USING (country_code)
GROUP BY c.continent;
```

This query finds the minimum founding year for businesses in each continent, essentially identifying the earliest possible business for every continent. While it doesn’t show business names, it gives us a numerical baseline for later analysis. It helps confirm which continents have data dating back the furthest.

<img width="1218" height="283" alt="image" src="https://github.com/user-attachments/assets/5da27c8d-456e-4f9e-a29a-0dd9e60fcaa2" />



### 4. Oldest Business on Each Continent

```
sql

SELECT
	s1.business,
	s1.year_founded,
	s1.country,
	s1.continent
FROM (
	SELECT
		b.country_code,
		b.business,
		b.year_founded,
		c.country,
		c.continent
	FROM businesses AS b
	INNER JOIN countries AS c
	USING(country_code)
) AS s1
INNER JOIN (
	SELECT 
	c.continent,	
	MIN(b.year_founded) AS oldest_year
	FROM businesses AS b
	INNER JOIN countries AS c
	USING (country_code)
	GROUP BY c.continent
) AS s2
ON s1.continent = s2.continent
	AND s1.year_founded = s2.oldest_year
ORDER BY s1.continent ASC;
```

We first attach the continent and country to every business, then separately find the minimum founding year per continent. Joining these two results returns only the businesses whose founding year equals the oldest for their continent. This guarantees we report the actual business names, not just the years. The final ordering by continent makes the output easy to read and compare across regions. This answer is precise and avoids guesswork: if more than one business shares the same earliest year in a continent, they will all appear, which is correct and transparent.

<img width="1214" height="291" alt="image" src="https://github.com/user-attachments/assets/102fb06a-d479-4290-bb8e-4354a05dce78" />



### 5. Countries Without Business Data

```
sql

-- How many countries per continent lack data on the oldest businesses
-- Does including the `new_businesses` data change this?
SELECT 
    c.continent,
    COUNT(c.country) AS countries_without_businesses
FROM countries AS c
LEFT JOIN (
        SELECT *
        FROM businesses
        UNION ALL
        SELECT *
        FROM new_businesses
        WHERE business IS NULL
) AS b
USING(country_code)
WHERE b.business IS NULL
GROUP BY c.continent;
```

Here we perform a gap analysis. We start from all countries (so none are missed), then try to match each to any listed business from the main table or the supplementary new_businesses. After the left join, any country that still has no matching business will have b.business is NULL. Counting those by continent tells us where the dataset lacks coverage. This is crucial because insights are only as strong as the underlying data; knowing where records are missing helps us avoid misleading conclusions and points to where more data collection is needed.
Including new_businesses widens the net, so if newer or corrected entries exist there, they can reduce the count of missing countries. If the numbers remain high in some continents, we know data is genuinely sparse there.

<img width="1217" height="284" alt="image" src="https://github.com/user-attachments/assets/6db50afe-a621-4759-a58c-31b8994f4253" />



### 6. Categories That Last the Longest

```
sql

-- Which business categories are best suited to last over the course of centuries?
SELECT 
    c1.continent,
    c2.category,
    MIN(b.year_founded) AS year_founded
FROM businesses AS b
INNER JOIN countries AS c1
    ON b.country_code = c1.country_code
INNER JOIN categories AS c2
    ON b.category_code = c2.category_code 
GROUP BY 
    c1.continent,
    c2.category
ORDER BY year_founded ASC;
```

We connect each business to its continent and category, then take the earliest founding year for every continent-category pair. Earlier years suggest that a category has had very long-lived examples in that region. Sorting by the earliest year puts the most enduring combinations at the top. This directly answers which categories seem built to last and where they have stood the test of time.
This view is powerful for pattern-spotting: if certain categories (e.g., food & beverage, publishing, hospitality) repeatedly appear with very old founding years across continents, that hints at resilient business models tied to basic, timeless needs.

<img width="1216" height="627" alt="image" src="https://github.com/user-attachments/assets/da2e29b7-9fa8-463c-ac8f-1ed726d43661" />
<br>
<br>


## Insights & Findings

* A clear, continent-by-continent list of the oldest businesses, including their countries and founding years, enabling quick comparison across regions.
* A precise count of how many countries lack records per continent, showing where the dataset is thin and where care is needed when generalizing results.
* A ranked view of continent-category pairs by earliest founding year, highlighting which categories have produced the longest-lived businesses and in which regions they are most enduring.
* If ties occur (same oldest year within a continent), the approach surfaces all matching businesses, preserving accuracy instead of arbitrarily choosing one.
* By validating the oldest years first, then attaching names, the analysis avoids common mistakes and keeps the results traceable and trustworthy.
<br>

## Final Notes

This workflow keeps things simple and reliable: preview the data, understand the schema, confirm the oldest years by continent, then attach business names, measure where data is missing, and profile category longevity by region. Each query serves a clear purpose and builds confidence step by step.

If you extend this project, consider adding summary tables for quick dashboards, checking for duplicate country codes or unexpected year values, and comparing results with new_businesses more deeply to reduce gaps. The same pattern—validate, enrich, aggregate, and interpret—will scale well as you add more data or pose new questions.
<br>


































