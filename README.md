# Uncovering-The-Worlds-Oldest-Businesses
This project investigates the world’s oldest businesses using PostgreSQL, combining business, country, and category datasets to uncover patterns in longevity. By identifying the oldest businesses per continent, analyzing missing country-level data, and exploring enduring categories, the project provides insights into which business types and regions have historically sustained long-term success.

<br>

## PROJECT OVERVIEW

Some businesses have survived for centuries, enduring wars, political changes, and economic crises. For example, Staffelter Hof Winery in Germany dates back to 862, thriving through empires, dynasties, and modern conflicts. This project examines what allows businesses to last over long periods by analyzing structured datasets using SQL in PostgreSQL. By linking business, country, and category information, we explore patterns in longevity, including location-based trends and resilient business categories.

<img src="https://github.com/user-attachments/assets/fd971e98-e7d4-4023-b6e1-5c9e91ecbe01" width="1000" alt="World's Oldest Business">

The analysis focuses on three key aspects: determining the oldest business on each continent, identifying countries lacking oldest business records, and discovering categories that historically produce long-lived businesses. Stepwise queries ensure reproducibility, clarity, and reliability of the findings.

<br>

## OBJECTIVE / KEY QUESTIONS

* What is the oldest business on each continent?
* How many countries per continent lack data on the oldest businesses?
* Which business categories are best suited to last many years, and on what continent are they?
  
<br>

## DATASET
* Source: [BusinessFinancing.co.uk](https://businessfinancing.co.uk/)
* Table:
  
   `businesses`/`new_businesses` 
     | Column         | Description                                 |
     | -------------- | ------------------------------------------- |
     | business       | Name of the business (varchar)              |
     | year\_founded  | Year the business was founded (int)         |
     | category\_code | Code for the business category (varchar)    |
     | country\_code  | ISO 3166-1 three-letter country code (char) |

   `countries` 
     | Column        | Description                                    |
     | ------------- | ---------------------------------------------- |
     | country\_code | ISO 3166-1 three-letter country code (varchar) |
     | country       | Name of the country (varchar)                  |
     | continent     | Name of the continent (varchar)                |

   `categories` 
     | Column         | Description                                    |
     | -------------- | ---------------------------------------------- |
     | category\_code | Code for the business category (varchar)       |
     | category       | Description of the business category (varchar) |

<br>

## TOOLS AND SKILLS USED

* [SQL (PostgreSQL)](https://www.postgresql.org/download/)
* Concepts: `INNER JOIN, LEFT JOIN, USING(), MIN(), COUNT(), GROUP BY, ORDER BY, WHERE`, Subqueries, `UNION ALL`, `LIMIT`.
* Techniques used:
  - Data preview and schema profiling to understand content and quality
  - Data integration across tables to enrich business records with country and continent info
  - Gap analysis to count missing entries by region
  - Category-level longevity analysis using the earliest founding years
  - Stepwise validation with intermediate checks (e.g., oldest year per continent before attaching names)

<br>

## ANALYSIS & APPROACH

### 1. Exploring Tables

```sql
-- Overview of the table businesses

SELECT *
FROM businesses
LIMIT 10;
```

<img width="800" alt="image" src="https://github.com/user-attachments/assets/9232f311-d650-4839-bafa-3e27894351f8" />

```sql
-- Overview of the table countries

SELECT *
FROM countries
LIMIT 10;
```

<img width="800" alt="image" src="https://github.com/user-attachments/assets/8cc14268-592c-413c-a3f0-40fe5aea5fe9" />

```sql
-- Overview of the table categories

SELECT *
FROM categories
LIMIT 10;
```
<img width="800" alt="image" src="https://github.com/user-attachments/assets/101ba49c-4553-48c9-ab02-8b18347b1da7" />

I began by exploring the three main tables — `businesses`, `countries`, and `categories` — to get an **initial understanding of the data, confirm field structure, and assess data quality**. Previewing the first few rows allowed me to verify that there were **no obvious missing or unusual entries, `businesses` were correctly linked to `countries` and `categories` column, `categories` were properly matched, and numeric fields (like `year_founded`) were usable for aggregation**. This step ensured **reliable joins and prevented misleading results** caused by missing or misaligned data.


### 2.  Schema Exploration

```sql

SELECT 
    table_name, 
    column_name, 
    data_type
FROM information_schema.columns 
WHERE table_name IN ('businesses', 'countries', 'categories') 
ORDER BY table_name, ordinal_position;
```

I queried `information_schema.columns` to **validate column names, data types, and table structure**. This helped determine which fields could be used in joins, aggregations, and filters. Knowing data types and nullability ensured **accurate computation of minimum years and correct joins across code fields**. By analyzing table structures, I ensured my queries and calculations were correct and reliable, **reducing errors and improving the accuracy of insights**.

<img width="800" alt="image" src="https://github.com/user-attachments/assets/6c420219-cd9a-49e6-bd75-0ee9c8ae56de" />


### 3. Oldest Year Founded by Continent

```sql

SELECT 
    MIN(b.year_founded)
FROM businesses AS b
INNER JOIN countries AS c
USING (country_code)
GROUP BY c.continent;
```

I calculated the earliest `founding_year`  for each continent to identify how far back business records extend in different regions. This step helped verify the completeness and historical depth of the dataset, providing a clear starting point for later analysis focused on the world’s oldest businesses and regional longevity trends.

<img width="1218" height="283" alt="image" src="https://github.com/user-attachments/assets/5da27c8d-456e-4f9e-a29a-0dd9e60fcaa2" />


### 4. Oldest Business on Each Continent

```sql

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

I linked each business with its respective `country` and `continent`, then identified the oldest businesses in every continent based on the earliest `founding_year`. This approach ensured that the analysis accurately captured not just the years but the actual business names representing each region’s historical origins. It provided a clear, transparent view of global business longevity, helping compare how early commerce began across different parts of the world.

<img width="1214" height="291" alt="image" src="https://github.com/user-attachments/assets/102fb06a-d479-4290-bb8e-4354a05dce78" />


### 5. Countries Without Business Data

```sql

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

I conducted a gap analysis to identify `countries` without any recorded `businesses`, using both the main and supplementary datasets. By starting with all `countries` and matching them to available business records, I pinpointed where data was missing or incomplete. Including the `new_businesses` table ensured newer or corrected entries were considered, giving a fuller picture of global coverage. This step helped assess dataset reliability and guided later analysis by highlighting regions where missing data could impact accuracy or insights.

<img width="1217" height="284" alt="image" src="https://github.com/user-attachments/assets/6db50afe-a621-4759-a58c-31b8994f4253" />


### 6. Categories That Last the Longest

```sql

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

I linked each business to its continent and category, then identified the earliest founding year for every continent-category pair. This allowed me to highlight which categories have historically produced long-lived businesses in each region. By focusing on the oldest examples, the analysis revealed resilient business models and helped identify industries likely to sustain longevity across different continents.

<img width="1216" height="627" alt="image" src="https://github.com/user-attachments/assets/da2e29b7-9fa8-463c-ac8f-1ed726d43661" />

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

