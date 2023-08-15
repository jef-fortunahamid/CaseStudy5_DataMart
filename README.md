# Case Study #5 Data Mart

*Note: All information and data related to the case study were obtained from [here](https://8weeksqlchallenge.com/case-study-5/).*
![Screenshot 2023-08-13 at 8 38 19 pm](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/2e39863d-786d-4988-8428-1528c39e094b)

## Business Task
Danny's new venture, "Data Mart," is an online supermarket specializing in fresh produce. In June 2020, Data Mart underwent a significant shift, adopting sustainable packaging methods for all its products, right from the farm to the customer's doorstep.
Danny seeks to understand the repercussions of this change on Data Mart's sales performance across various business areas. The primary questions to address are:
1. How did the sustainable packaging initiative, introduced in June 2020, affect sales?
2. Which platforms, regions, segments, and customer demographics felt the most impact from this change?
3. Based on the findings, how can Data Mart implement future sustainability measures without adversely affecting sales?

## Entity Relationship Diagram
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/091ba931-9ef9-4540-8410-41822df1b8ef)

## General Insights
- Formatting date columns and extracting date-related attributes like week_number, month_number, and calendar_year.
1. *Data Cleansing and Transformation:* The queries primarily focused on the transformation of the weekly_sales data. This involved:
- Creating new attributes like age_band and demographic based on existing column data.
- Handling null values by replacing them with "unknown" and calculating averages.
2. *Data Exploration:*
- The day of the week corresponding to the week_date was identified.
- Missing week numbers from the dataset were identified.
- Total transactions were aggregated by year.
- The total sales for each region and each month were aggregated.
- Transaction counts were aggregated for each platform.
- The percentage of sales between Retail and Shopify was computed for each month.
- The amount and percentage of sales by demographic were computed for each year.
- The contribution of age_band and demographic values to Retail sales was computed.
3. *Before and After Analysis:*
- Sales were analyzed for periods before and after specific dates (especially around 2020-06-15) to understand the impact of potential events on sales performance.
- This analysis was extended to different facets of the business, like regions, platforms, age bands, and customer types.
4. *Business Impact Analysis:*
- The areas of the business most negatively impacted in sales metrics for the 12 weeks before and after 2020-06-15 were identified. This was done across multiple dimensions: region, platform, age band, demographic, and customer type.

## Key SQL Syntax and Functions:
- Temporary Tables (`CREATE TEMP TABLE`)
- Date Functions (`TO_DATE`, `DATE_PART`, `DATE_TRUNC`, `TO_CHAR`)
- Aggregation Functions (`SUM`, `MAX`)
- Conditional Logic (`CASE WHEN`)
- String Functions (`LEFT`, `RIGHT`)
- Common Table Expressions (CTE)
- Window Functions (`LAG`)
- Joins (LEFT ANTI JOIN - `WHERE NOT EXISTS`)
- Concatenation Operation (`||`)

## Questions and Solutions
*Throughout this case study, I provided alternative solutions to some questions. In SQL, there are a lot of approaches you can tackle a given problem; the key is identifying the most optimised query for your SQL environment.*

### Part A: Data Cleansing Steps
> In a single query, perform the following operations and generate a new table in the data_mart schema named > clean_weekly_sales:
> 
> - Convert the `week_date` to a `DATE` format
> - Add a `week_number` as the second column for each `week_date` value, for example any value from the 1st of January to 7th ofMONTH January will be 1, 8th to 14th will be 2 etc
> - Add a `month_number` with the calendar month for each `week_date` value as the 3rd column
> - Add a `calendar_year` column as the 4th column containing either 2018, 2019 or 2020 values
> - Add a new column called `age_band` after the original `segment` column using the following mapping on the number inside the `segment` value
>   
> | segment | age_band |
> | --- | --- |
> | 1 | Young Adults |
> | 2 | Middle Aged |
> | 3 or 4 | Retirees |
> - Add a new `demographic` column using the following mapping for the first letter in the `segment` values:
>   
> | segment | demographic |
> | --- | --- |
> | C | Couples |
> | F | Families |
> - Ensure all `null` string values with an `"unknown"` string value in the original `segment` column as well as the new > `age_band` and `demographic` columns
> - Generate a new `avg_transaction` column as the `sales` value divided by `transactions` rounded to 2 decimal places for each record
```sql
DROP TABLE IF EXISTS data_mart.clean_weekly_sales;
CREATE TABLE data_mart.clean_weekly_sales AS
  SELECT
      TO_DATE(week_date, 'DD/MM/YY') AS week_date
    , DATE_PART('week', TO_DATE(week_date, 'DD/MM/YY')) AS week_number
    , DATE_PART('month', TO_DATE(week_date, 'DD/MM/YY')) AS month_number
    , DATE_PART('year', TO_DATE(week_date, 'DD/MM/YY')) AS calendar_year
    , region
    , platform
    , CASE
          WHEN segment = 'null' THEN 'Unknown'
          ELSE segment
        END AS segment
    , CASE
	        WHEN RIGHT(segment, 1) = '1' THEN 'Young Adults'
	        WHEN RIGHT(segment, 1) = '2' THEN 'Middle Aged'
	        WHEN RIGHT(segment, 1) IN ('3', '4') THEN 'Retirees'
	        ELSE 'Unknown'
	      END AS age_band
    , CASE
	        WHEN LEFT(segment, 1) = 'C' THEN 'Couples'
	        WHEN LEFT(segment, 1) = 'F' THEN 'Families'
	        ELSE 'Unknown'
	      END AS demographic
    , customer_type
    , transactions
    , sales
    , ROUND((sales::NUMERIC / transactions::NUMERIC), 2) AS avg_transaction
  FROM data_mart.weekly_sales;

SELECT * FROM data_mart.clean_weekly_sales LIMIT 10;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/44cd2c55-22bd-4ff0-8f5b-9483376dcd44)

### Part B: Data Exploration
> 1. What day of the week is used for each week_date value?
```sql
SELECT
  TO_CHAR(week_date, 'Day') as weekday
FROM data_mart.clean_weekly_sales;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/00aa5f55-2f68-441c-a818-10468909a72a)

> 2. What range of week numbers are missing from the dataset?
```sql
WITH all_week_numbers AS (
  SELECT GENERATE_SERIES(1, 52) AS week_number
)
SELECT
  week_number
FROM all_week_numbers AS t1 
WHERE NOT EXISTS (
  SELECT 1
  FROM data_mart.clean_weekly_sales AS t2
  WHERE t1.week_number = t2.week_number
);
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/c145c438-a2c0-4abb-a534-4a196e7c04e9)
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/a2ece303-3803-432d-9c42-9a61626d47f7)

> 3. How many total transactions were there for each year in the dataset?
```sql
SELECT
    calendar_year
  , SUM(transactions) AS total_transactions
FROM data_mart.clean_weekly_sales
GROUP BY calendar_year
ORDER BY calendar_year;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/b1a84c79-a708-4483-bad6-36649c52eed1)

> 4. What is the total sales for each region for each month?
```sql
SELECT
    TO_DATE(calendar_year || '-' || month_number || '-01', 'YYYY-MM-DD') AS calendar_month
  , region
  , SUM(sales) AS total_sales
FROM data_mart.clean_weekly_sales
GROUP BY calendar_month, region
ORDER BY calendar_month, region;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/a1dd3f22-0585-4dd8-ba69-8c596470b80b)

> 5. What is the total count of transactions for each platform
```sql
SELECT
    platform
  , SUM(transactions) AS total_transactions
FROM data_mart.clean_weekly_sales
GROUP BY platform
ORDER BY platform;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/ab80523e-3197-4748-ab7e-c73f9a257ef9)

> 6. What is the percentage of sales for Retail vs Shopify for each month?
```sql
WITH monthly_platform_sales AS (
  SELECT
      DATE_TRUNC('month', week_date)::DATE AS _month
    , platform
    , SUM(sales) AS monthly_sales
  FROM data_mart.clean_weekly_sales
  GROUP BY _month, platform
)
SELECT
    _month
  , ROUND(100 * 
      SUM(CASE WHEN platform = 'Retail' THEN monthly_sales ELSE 0 END) /
        SUM(monthly_sales)::NUMERIC
      , 2) AS retail_percentage
  , ROUND(100 *
      SUM(CASE WHEN platform = 'Shopify' THEN monthly_sales ELSE 0 END) /
        SUM(monthly_sales)::NUMERIC
      , 2) AS shopify_percentage
FROM monthly_platform_sales
GROUP BY _month
ORDER BY _month;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/4dfa68ec-4ce8-4b0c-bf31-1fbca623e993)

> 7. What is the amount and percentage of sales by demographic for each year in the dataset?
```sql
SELECT
    calendar_year
  , demographic
  , SUM(sales) AS yearly_sales
  , ROUND(
        (
          100 * SUM(sales)::NUMERIC / 
							SUM(SUM(sales)) OVER (PARTITION BY calendar_year)
        )::NUMERIC
        , 2) AS percentage
FROM data_mart.clean_weekly_sales
GROUP BY
    calendar_year
  , demographic
ORDER BY 
    calendar_year
  , demographic;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/668c7e8f-5454-4f0f-82ad-32e2d3e40f16)

> 8. Which age_band and demographic values contribute the most to Retail sales?
```sql
--age_band only
SELECT
    age_band
  , SUM(sales) AS total_sales
  , ROUND(
          100 * SUM(sales) / SUM(SUM(sales)) OVER ()
          ) AS sales_percentage
FROM data_mart.clean_weekly_sales
GROUP BY age_band
ORDER BY sales_percentage DESC;

--demographic only
SELECT
    demographic
  , SUM(sales) AS total_sales
  , ROUND(
          100 * SUM(sales) / SUM(SUM(sales)) OVER ()
          ) AS sales_percentage
FROM data_mart.clean_weekly_sales
GROUP BY demographic
ORDER BY sales_percentage DESC;

--both age_band and demographic
SELECT
    age_band
  , demographic
  , SUM(sales) AS total_sales
  , ROUND(
          100 * SUM(sales) / SUM(SUM(sales)) OVER ()
          ) AS sales_percentage
FROM data_mart.clean_weekly_sales
GROUP BY 
    age_band
  , demographic
ORDER BY sales_percentage DESC;
```
**age_band**
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/a8334a42-656e-42ed-a415-f09c7a3a9f2e)
**demographic**
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/8c99cce5-4249-44b8-9158-b386fb48b9bc)
**both age_band and demographic**
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/1bcd4b1f-2f04-404c-a562-ab4e53dafc62)

> 9. Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?
```sql
SELECT
    calendar_year
  , platform
  , ROUND(AVG(avg_transaction), 2) AS average_avg_transaction
  , SUM(sales) / SUM(transactions) AS avg_annual_transaction
FROM data_mart.clean_weekly_sales
GROUP BY
    calendar_year
  , platform
ORDER BY
    calendar_year
  , platform;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/ff5be680-7f63-4e14-8a14-19a6a0231b4a)

### Part C: Before and After Analysis
> 1. What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?
- Identify the `week_number` of 2020-06-15
```sql
SELECT
  DISTINCT week_number
FROM data_mart.clean_weekly_sales
WHERE week_date = '2020-06-15';
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/e953573c-0485-4f94-89e2-0deee129a5c1)
```sql
WITH before_after_sales_2020 AS (
  SELECT
      CASE
        WHEN week_number BETWEEN 21 AND 24 THEN '1. Before'
        WHEN week_number BETWEEN 25 AND 28 THEN '2. After'
      END AS period_name
    , SUM(sales) AS total_sales
  FROM data_mart.clean_weekly_Sales
  WHERE week_number BETWEEN 21 AND 28
    AND calendar_year = '2020'
  GROUP BY period_name
  ORDER BY period_name
),
calculations AS (
  SELECT
      period_name
    , total_sales - LAG(total_sales) OVER (ORDER BY period_name) AS sales_diff
    , ROUND(100 * ((total_sales::NUMERIC / LAG(total_sales) OVER (ORDER BY period_name)) - 1), 2) AS sales_change
  FROM before_after_sales_2020
)
SELECT
    sales_diff
  , sales_change
FROM calculations
WHERE sales_diff IS NOT NULL
  AND sales_change IS NOT NULL;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/8b8fe5de-4e95-468f-9f66-2ee99e63f5d3)

*Another approach*
```sql
WITH before_sales AS (
    SELECT 
        SUM(sales) AS total_sales_before
    FROM data_mart.clean_weekly_sales
    WHERE week_number BETWEEN 21 AND 24
    AND calendar_year = 2020
), 
after_sales AS (
    SELECT 
        SUM(sales) AS total_sales_after
    FROM data_mart.clean_weekly_sales
    WHERE week_number BETWEEN 25 AND 28
    AND calendar_year = 2020
)
SELECT 
    (total_sales_after - total_sales_before) AS sales_diff
  , ROUND((100 * (total_sales_after - total_sales_before)::NUMERIC / total_sales_before), 2) AS sales_growth_rate_percent
FROM before_sales, after_sales;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/496131ae-0fcc-4e09-9dae-b4eb180f93e8)

> 2. What about the entire 12 weeks before and after?
```sql
WITH before_after_sales_2020 AS (
  SELECT
      CASE
        WHEN week_number BETWEEN 13 AND 24 THEN '1. Before'
        WHEN week_number BETWEEN 25 AND 36 THEN '2. After'
      END AS period_name
    , SUM(sales) AS total_sales
  FROM data_mart.clean_weekly_Sales
  WHERE week_number BETWEEN 13 AND 36
    AND calendar_year = '2020'
  GROUP BY period_name
  ORDER BY period_name
),
calculations AS (
  SELECT
      period_name
    , total_sales - LAG(total_sales) OVER (ORDER BY period_name) AS sales_diff
    , ROUND(100 * ((total_sales::NUMERIC / LAG(total_sales) OVER (ORDER BY period_name)) - 1), 2) AS sales_change
  FROM before_after_sales_2020
)
SELECT
    sales_diff
  , sales_change
FROM calculations
WHERE sales_diff IS NOT NULL
  AND sales_change IS NOT NULL;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/1bc1f845-46e6-42ce-bef3-a0a27ba2c632)

*Another approach*
```sql
WITH before_sales AS (
    SELECT 
        SUM(sales) AS total_sales_before
    FROM data_mart.clean_weekly_sales
    WHERE week_number BETWEEN 13 AND 24
    AND calendar_year = 2020
), 
after_sales AS (
    SELECT 
        SUM(sales) AS total_sales_after
    FROM data_mart.clean_weekly_sales
    WHERE week_number BETWEEN 25 AND 36
    AND calendar_year = 2020
)
SELECT 
    (total_sales_after - total_sales_before) AS sales_diff
  , ROUND((100 * (total_sales_after - total_sales_before)::NUMERIC / total_sales_before), 2) AS sales_growth_rate_percent
FROM before_sales, after_sales;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/0b76f7f7-bbc7-4e6b-87e6-1f1b3e893e02)

> 3. How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?
- 4 weeks before and after
```sql
WITH before_after_sales AS (
  SELECT
      calendar_year
    , CASE
        WHEN week_number BETWEEN 21 AND 24 THEN '1. Before'
        WHEN week_number BETWEEN 25 AND 28 THEN '2. After'
      END AS period_name
    , SUM(sales) AS total_sales
  FROM data_mart.clean_weekly_Sales
  WHERE week_number BETWEEN 13 AND 36
  GROUP BY calendar_year, period_name
  ORDER BY calendar_year, period_name
),
calculations AS (
  SELECT
      calendar_year
    , period_name
    , total_sales - 
              LAG(total_sales) OVER (PARTITION BY calendar_year ORDER BY period_name) AS sales_diff
    , ROUND(100 * 
              (
                (total_sales::NUMERIC / 
                      LAG(total_sales) OVER (PARTITION BY calendar_year ORDER BY period_name)) - 1
              )
        , 2) AS sales_change
  FROM before_after_sales
)
SELECT
    calendar_year
  , sales_diff
  , sales_change
FROM calculations
WHERE period_name IS NOT NULL
  AND sales_diff IS NOT NULL
  AND sales_change IS NOT NULL
ORDER BY calendar_year;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/ce1aceaa-2904-425c-82e2-e43cf3bc21c8)

*Another approach*
```sql
WITH before_after_sales AS (
    SELECT 
        calendar_year
			, SUM(CASE WHEN week_number BETWEEN 21 AND 24 THEN sales ELSE 0 END) AS total_sales_before
			, SUM(CASE WHEN week_number BETWEEN 25 AND 28 THEN sales ELSE 0 END) AS total_sales_after
    FROM data_mart.clean_weekly_sales
    WHERE calendar_year BETWEEN 2018 AND 2020
    GROUP BY calendar_year
)
SELECT 
    calendar_year
	, (total_sales_after - total_sales_before) AS sales_diff
	, ROUND((100 * (total_sales_after - total_sales_before)::NUMERIC / total_sales_before), 2) AS sales_growth_rate_percent
FROM before_after_sales
ORDER BY calendar_year;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/dffd1ddf-5c84-43aa-8b74-86729e545e77)

- 12 weeks before and after
```sql
WITH before_after_sales AS (
  SELECT
      calendar_year
    , CASE
        WHEN week_number BETWEEN 13 AND 24 THEN '1. Before'
        WHEN week_number BETWEEN 25 AND 36 THEN '2. After'
      END AS period_name
    , SUM(sales) AS total_sales
  FROM data_mart.clean_weekly_Sales
  WHERE week_number BETWEEN 13 AND 36
  GROUP BY calendar_year, period_name
  ORDER BY calendar_year, period_name
),
calculations AS (
  SELECT
      calendar_year
    , period_name
    , total_sales - 
              LAG(total_sales) OVER (PARTITION BY calendar_year ORDER BY period_name) AS sales_diff
    , ROUND(100 * 
              (
                (total_sales::NUMERIC / 
                      LAG(total_sales) OVER (PARTITION BY calendar_year ORDER BY period_name)) - 1
              )
        , 2) AS sales_change
  FROM before_after_sales
)
SELECT
    calendar_year
  , sales_diff
  , sales_change
FROM calculations
WHERE period_name IS NOT NULL
  AND sales_diff IS NOT NULL
  AND sales_change IS NOT NULL
ORDER BY calendar_year;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/5e94a176-ce6b-45f5-abbc-b6e2bc717fca)

*Another approach*
```sql
WITH before_after_sales AS (
    SELECT 
        calendar_year
			, SUM(CASE WHEN week_number BETWEEN 13 AND 24 THEN sales ELSE 0 END) AS total_sales_before
			, SUM(CASE WHEN week_number BETWEEN 25 AND 36 THEN sales ELSE 0 END) AS total_sales_after
    FROM data_mart.clean_weekly_sales
    WHERE calendar_year BETWEEN 2018 AND 2020
    GROUP BY calendar_year
)
SELECT 
    calendar_year
	, (total_sales_after - total_sales_before) AS sales_diff
  , ROUND((100 * (total_sales_after - total_sales_before)::NUMERIC / total_sales_before), 2) AS sales_growth_rate_percent
FROM before_after_sales
ORDER BY calendar_year;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/713ceb82-64d6-43ec-9dc6-11bb8624eb3f)

### Part D: Bonus Question
> Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period?
> - `region`
> - `platform`
> - `age_band`
> - `demographic`
> - `customer_type`

**By `region`**
```sql
WITH before_after_sales AS (
  SELECT
      calendar_year
    , region
    , CASE
        WHEN week_number BETWEEN 13 AND 24 THEN '1. Before'
        WHEN week_number BETWEEN 25 AND 36 THEN '2. After'
      END AS period_name
    , SUM(sales) AS total_sales
  FROM data_mart.clean_weekly_Sales
  WHERE week_number BETWEEN 13 AND 36
    AND calendar_year = '2020'
  GROUP BY calendar_year, region, period_name
  ORDER BY calendar_year, region, period_name
),
calculations AS (
  SELECT
      calendar_year
    , region
    , period_name
    , total_sales - 
              LAG(total_sales) OVER (PARTITION BY region ORDER BY period_name) AS sales_diff
    , ROUND(100 * 
              (
                (total_sales::NUMERIC / 
                      LAG(total_sales) OVER (PARTITION BY region ORDER BY period_name)) - 1
              )
        , 2) AS sales_change
  FROM before_after_sales
)
SELECT
    calendar_year
  , region
  , sales_diff
  , sales_change
FROM calculations
WHERE period_name IS NOT NULL
  AND sales_diff IS NOT NULL
  AND sales_change IS NOT NULL
ORDER BY sales_change;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/01fb8f1f-68a2-46b3-bdc0-6b99a6d88344)

*Another approach*
```sql
WITH before_after_sales AS (
  SELECT 
      calendar_year
    , region
    , SUM(CASE WHEN week_number BETWEEN 13 AND 24 THEN sales ELSE 0 END) AS total_sales_before
    , SUM(CASE WHEN week_number BETWEEN 25 AND 36 THEN sales ELSE 0 END) AS total_sales_after
  FROM data_mart.clean_weekly_sales
  WHERE calendar_year = 2020
  GROUP BY calendar_year, region
)
SELECT 
    calendar_year
  , region
  , (total_sales_after - total_sales_before) AS sales_diff
  , ROUND((100 * (total_sales_after - total_sales_before)::NUMERIC / total_sales_before), 2) AS sales_growth_rate_percent
FROM before_after_sales
ORDER BY sales_growth_rate_percent;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/9931a91b-de81-4525-845c-895028b111f3)


**By `platform`**
```sql
WITH before_after_sales AS (
  SELECT
      calendar_year
    , platform
    , CASE
        WHEN week_number BETWEEN 13 AND 24 THEN '1. Before'
        WHEN week_number BETWEEN 25 AND 36 THEN '2. After'
      END AS period_name
    , SUM(sales) AS total_sales
  FROM data_mart.clean_weekly_Sales
  WHERE week_number BETWEEN 13 AND 36
    AND calendar_year = '2020'
  GROUP BY calendar_year, platform, period_name
),
calculations AS (
  SELECT
      calendar_year
    , platform
    , period_name
    , total_sales - 
              LAG(total_sales) OVER (PARTITION BY platform ORDER BY period_name) AS sales_diff
    , ROUND(100 * 
              (
                (total_sales::NUMERIC / 
                      LAG(total_sales) OVER (PARTITION BY platform ORDER BY period_name)) - 1
              )
        , 2) AS sales_change
  FROM before_after_sales
)
SELECT
    calendar_year
  , platform
  , sales_diff
  , sales_change
FROM calculations
WHERE sales_diff IS NOT NULL
  AND sales_change IS NOT NULL
ORDER BY sales_change;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/638e98a5-5223-46d0-a6ac-ca6f063f8fa4)

*Another approach*
```sql
WITH before_after_sales AS (
  SELECT 
      calendar_year
    , platform
    , SUM(CASE WHEN week_number BETWEEN 13 AND 24 THEN sales ELSE 0 END) AS total_sales_before
    , SUM(CASE WHEN week_number BETWEEN 25 AND 36 THEN sales ELSE 0 END) AS total_sales_after
  FROM data_mart.clean_weekly_sales
  WHERE calendar_year = 2020
  GROUP BY calendar_year, platform
)
SELECT 
    calendar_year
  , platform
  , (total_sales_after - total_sales_before) AS sales_diff
  , ROUND((100 * (total_sales_after - total_sales_before)::NUMERIC / total_sales_before), 2) AS sales_growth_rate_percent
FROM before_after_sales
ORDER BY sales_growth_rate_percent;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/1fbfb7bc-cf38-412d-b8f9-93f41d275db9)

**By `age_band`**
```sql
WITH before_after_sales AS (
  SELECT
      calendar_year
    , age_band
    , CASE
        WHEN week_number BETWEEN 13 AND 24 THEN '1. Before'
        WHEN week_number BETWEEN 25 AND 36 THEN '2. After'
      END AS period_name
    , SUM(sales) AS total_sales
  FROM data_mart.clean_weekly_Sales
  WHERE week_number BETWEEN 13 AND 36
    AND calendar_year = '2020'
  GROUP BY calendar_year, age_band, period_name
),
calculations AS (
  SELECT
      calendar_year
    , age_band
    , period_name
    , total_sales - 
              LAG(total_sales) OVER (PARTITION BY age_band ORDER BY period_name) AS sales_diff
    , ROUND(100 * 
              (
                (total_sales::NUMERIC / 
                      LAG(total_sales) OVER (PARTITION BY age_band ORDER BY period_name)) - 1
              )
        , 2) AS sales_change
  FROM before_after_sales
)
SELECT
    calendar_year
  , age_band
  , sales_diff
  , sales_change
FROM calculations
WHERE sales_diff IS NOT NULL
  AND sales_change IS NOT NULL
ORDER BY sales_change;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/ccf417e7-fc08-413b-abd4-acbbbad35757)

*Another approach*
```sql
WITH before_after_sales AS (
  SELECT 
      calendar_year
    , age_band
    , SUM(CASE WHEN week_number BETWEEN 13 AND 24 THEN sales ELSE 0 END) AS total_sales_before
    , SUM(CASE WHEN week_number BETWEEN 25 AND 36 THEN sales ELSE 0 END) AS total_sales_after
  FROM data_mart.clean_weekly_sales
  WHERE calendar_year = 2020
  GROUP BY calendar_year, age_band
)
SELECT 
    calendar_year
  , age_band
  , (total_sales_after - total_sales_before) AS sales_diff
  , ROUND((100 * (total_sales_after - total_sales_before)::NUMERIC / total_sales_before), 2) AS sales_growth_rate_percent
FROM before_after_sales
ORDER BY sales_growth_rate_percent;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/2823ef19-495e-4233-a548-633b9e8cee0e)

**By `demographic`**
```sql
WITH before_after_sales AS (
  SELECT
      calendar_year
    , demographic
    , CASE
        WHEN week_number BETWEEN 13 AND 24 THEN '1. Before'
        WHEN week_number BETWEEN 25 AND 36 THEN '2. After'
      END AS period_name
    , SUM(sales) AS total_sales
  FROM data_mart.clean_weekly_Sales
  WHERE week_number BETWEEN 13 AND 36
    AND calendar_year = '2020'
  GROUP BY calendar_year, demographic, period_name
),
calculations AS (
  SELECT
      calendar_year
    , demographic
    , period_name
    , total_sales - 
              LAG(total_sales) OVER (PARTITION BY demographic ORDER BY period_name) AS sales_diff
    , ROUND(100 * 
              (
                (total_sales::NUMERIC / 
                      LAG(total_sales) OVER (PARTITION BY demographic ORDER BY period_name)) - 1
              )
        , 2) AS sales_change
  FROM before_after_sales
)
SELECT
    calendar_year
  , demographic
  , sales_diff
  , sales_change
FROM calculations
WHERE sales_diff IS NOT NULL
  AND sales_change IS NOT NULL
ORDER BY sales_change;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/293de39b-161e-40ad-81bd-00b9e44d8ead)

*Another approach*
```sql
WITH before_after_sales AS (
  SELECT 
      calendar_year
    , demographic
    , SUM(CASE WHEN week_number BETWEEN 13 AND 24 THEN sales ELSE 0 END) AS total_sales_before
    , SUM(CASE WHEN week_number BETWEEN 25 AND 36 THEN sales ELSE 0 END) AS total_sales_after
  FROM data_mart.clean_weekly_sales
  WHERE calendar_year = 2020
  GROUP BY calendar_year, demographic
)
SELECT 
    calendar_year
  , demographic
  , (total_sales_after - total_sales_before) AS sales_diff
  , ROUND((100 * (total_sales_after - total_sales_before)::NUMERIC / total_sales_before), 2) AS sales_growth_rate_percent
FROM before_after_sales
ORDER BY sales_growth_rate_percent;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/122b3b4d-f55a-4272-a93c-3ab393aa2838)

**By `customer_type`**
```sql
WITH before_after_sales AS (
  SELECT
      calendar_year
    , customer_type
    , CASE
        WHEN week_number BETWEEN 13 AND 24 THEN '1. Before'
        WHEN week_number BETWEEN 25 AND 36 THEN '2. After'
      END AS period_name
    , SUM(sales) AS total_sales
  FROM data_mart.clean_weekly_Sales
  WHERE week_number BETWEEN 13 AND 36
    AND calendar_year = '2020'
  GROUP BY calendar_year, customer_type, period_name
),
calculations AS (
  SELECT
      calendar_year
    , customer_type
    , period_name
    , total_sales
    , total_sales - 
              LAG(total_sales) OVER (PARTITION BY customer_type ORDER BY period_name) AS sales_diff
    , ROUND(100 * 
              (
                (total_sales::NUMERIC / 
                      LAG(total_sales) OVER (PARTITION BY customer_type ORDER BY period_name)) - 1
              )
        , 2) AS sales_change
  FROM before_after_sales
)
SELECT
    calendar_year
  , customer_type
  , period_name
  , total_sales
  , sales_diff
  , sales_change
FROM calculations
WHERE sales_diff IS NOT NULL
  AND sales_change IS NOT NULL
ORDER BY sales_change;
```
![image](https://github.com/jef-fortunahamid/CaseStudy5_DataMart/assets/125134025/4e4b07e0-4b65-4a0e-a443-90c9fc22212c)
