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
- Date Functions (`TO_DATE`, `DATE_PART`, `DATE_TRUNC`)
- Aggregation Functions (`SUM`, `MAX`)
- Conditional Logic (`CASE WHEN`)
- String Functions (`LEFT`, `RIGHT`)
- Common Table Expressions (CTE)
- Window Functions (`LAG`)

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










