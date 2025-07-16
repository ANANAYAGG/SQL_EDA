# SQL EDA Project: Analysis of World Layoffs

## Overview

This project explores global tech layoffs through exploratory data analysis (EDA) using the `layoffs_staging2` dataset. The SQL code investigates patterns, trends, and outliers in layoffs data, helping uncover insights about company size, industry impacts, geography, and funding.

## 1. Data Exploration & Sample Rows

A full snapshot of the data helps understand its structure and gather initial impressions:

| company       | location      | industry    | total_laid_off | percentage_laid_off | date      | stage    | country       | funds_raised_millions |
|---------------|--------------|-------------|---------------:|---------------------|-----------|----------|---------------|----------------------:|
| Atlassian     | Sydney       | Other       |            500 | 0.05                | 3/6/2023  | Post-IPO | Australia     |                   210 |
| SiriusXM      | New York City| Media       |            475 | 0.08                | 3/6/2023  | Post-IPO | United States |                   525 |
| ...           | ...          | ...         |            ... | ...                 | ...       | ...      | ...           |                   ... |

## 2. Quick Insights

### Maximum Layoffs in a Single Event

```sql
SELECT MAX(total_laid_off)
FROM world_layoffs.layoffs_staging2;
```
- **Finds the largest recorded layoff event.**

### Percentage Laid Off (Impact Size)

```sql
SELECT MAX(percentage_laid_off), MIN(percentage_laid_off)
FROM world_layoffs.layoffs_staging2
WHERE percentage_laid_off IS NOT NULL;
```
- Reveals the range of layoff impact—some companies laid off nearly all staff.

### Companies That Laid Off 100% of Staff

```sql
SELECT *
FROM world_layoffs.layoffs_staging2
WHERE percentage_laid_off = 1
ORDER BY funds_raised_millions DESC;
```
- Identifies companies that shut down, often well-funded startups such as Quibi.

## 3. Group Analysis

### Largest Single-Day Layoffs (Top 5)

```sql
SELECT company, total_laid_off
FROM world_layoffs.layoffs_staging
ORDER BY 2 DESC
LIMIT 5;
```

### Companies with Most Total Layoffs

```sql
SELECT company, SUM(total_laid_off)
FROM world_layoffs.layoffs_staging2
GROUP BY company
ORDER BY 2 DESC
LIMIT 10;
```
- Shows which companies have the highest combined layoffs over time.

### Locations with Highest Layoff Sums

```sql
SELECT location, SUM(total_laid_off)
FROM world_layoffs.layoffs_staging2
GROUP BY location
ORDER BY 2 DESC
LIMIT 10;
```
- Geographical layoff clustering.

### Layoffs by Country

```sql
SELECT country, SUM(total_laid_off)
FROM world_layoffs.layoffs_staging2
GROUP BY country
ORDER BY 2 DESC;
```
- Country-level impact distribution.

### Year-by-Year Layoff Trends

```sql
SELECT YEAR(date), SUM(total_laid_off)
FROM world_layoffs.layoffs_staging2
GROUP BY YEAR(date)
ORDER BY 1 ASC;
```
- Insights into how layoffs fluctuate over different years.

### Layoffs by Industry and Stage

```sql
SELECT industry, SUM(total_laid_off)
FROM world_layoffs.layoffs_staging2
GROUP BY industry
ORDER BY 2 DESC;

SELECT stage, SUM(total_laid_off)
FROM world_layoffs.layoffs_staging2
GROUP BY stage
ORDER BY 2 DESC;
```
- Reveals which industries and company stages faced the most reductions.

## 4. Advanced Analysis

### Top Companies by Layoffs Per Year

```sql
WITH Company_Year AS (
  SELECT company, YEAR(date) AS years, SUM(total_laid_off) AS total_laid_off
  FROM layoffs_staging2
  GROUP BY company, YEAR(date)
),
Company_Year_Rank AS (
  SELECT company, years, total_laid_off, DENSE_RANK() OVER (PARTITION BY years ORDER BY total_laid_off DESC) AS ranking
  FROM Company_Year
)
SELECT company, years, total_laid_off, ranking
FROM Company_Year_Rank
WHERE ranking <= 3
AND years IS NOT NULL
ORDER BY years ASC, total_laid_off DESC;
```
- Tracks the top 3 companies with the largest layoffs for each year.

### Rolling Monthly Layoff Totals

```sql
WITH DATE_CTE AS (
  SELECT SUBSTRING(date, 1, 7) AS dates, SUM(total_laid_off) AS total_laid_off
  FROM layoffs_staging2
  GROUP BY dates
  ORDER BY dates ASC
)
SELECT dates, SUM(total_laid_off) OVER (ORDER BY dates ASC) AS rolling_total_layoffs
FROM DATE_CTE
ORDER BY dates ASC;
```
- Measures the cumulative layoffs over time for trend visualization.

## 5. Key Findings & Observations

- **Extreme Layoffs:** Some companies laid off 100% of their staff (often recently funded startups).
- **Not Just Startups:** Large, well-funded companies also executed mass layoffs, showing industry volatility.
- **Geographic Concentration:** Major tech hubs (e.g., SF Bay Area) lead in both frequency and magnitude of layoffs.
- **Industry Trends:** Certain industries (e.g., tech, media) and *late-stage* companies tend to see larger layoffs.
- **Macro Impact:** Layoff spikes often align with broader economic downturns.
- **Outliers:** Companies like Quibi represent cases where hundreds of millions (even billions) were raised before total failure.


