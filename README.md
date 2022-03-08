# Case_Study#8: FreshSegments

<p align="center">
        <img src ="./images/FreshSegments.png">
</p>

---

#Introduction
Fresh Segments is a digitial marketing agency that helps other businesses to analyze trends in online ad click behavior. The dataset used in this case study consists of aggregated metrics for each interest for each month. This dataset consists of data from only a single client of Fresh Segments.

# Table of Contents

-[Problem Statement](#Problem-Statement)
-[Exploration](#Exploration)
-[Analysis](#Analysis)
-[Report](#Report)

# Problem Statement <a name = "Problem-Statement"></a>




# Exploration
The dataset consists of two tables: Interest Metrics and Interest Map. The ERD for the dataset is below: 

<iframe width="560" height="315" 
src='https://dbdiagram.io/embed/6226656061d06e6eadbb3b69'> </iframe>

**Metric Description:**

Composition: The composition value is the percentage of client's customers that interacted with specific interest.

Index_value: The index value describes composition value for an interest compared to all Fresh Segments clients' customer. So this means an index of 2 for a specific interest means that the composition value of the interest is 2 times the average composition value for all Fresh Segments clients' customer. 

## Dealing with NULLs
NULL entries in the `interest_metrics` table were found. It was observed that the NULL rows contained data for composition and index_value but no information was present for `_month`, `_year` and `month_year` columns. Since this information cannot be inferred, I decided to delete NULL rows from the table and use the resulting table for further analysis.

In my current approach, I deleted NULL rows direclty from the dataset. However, a better appoach would be to copy the data to a temp table and perform analysis on that table so that your raw data is preserved. 

```sql
SELECT 
  COUNT(month_year) AS record_counts
FROM fresh_segments.interest_metrics
WHERE month_year IS NULL;
--Dealing with Nulls : here it is best to delete those rows
DELETE FROM fresh_segments.interest_metrics WHERE month_year IS NULL;
```
Similarly, the `interest_map` table was also checked for NULLs and none were found. 

## Checking for Missing Data
Next I wanted to see if there is any missing/unexplained data in the dataset. For this I 
1. Check if all the `interest_id` in the `interest_metrics` table are present in the `interest_map` table. 
    This is to make sure that we don't have any records that have `interest_id`s that are not present in the `interest_map` table. (No foreign keys that do not match with primary keys are present) 
  
  ```sql
  SELECT 
    COUNT(DISTINCT(t1.interest_id)) AS total_interest_metrics_id,
    COUNT(DISTINCT(t2.id)) AS total_interest_map_id,
    COUNT(CASE WHEN t2.id IS NULL THEN t1.interest_id ELSE NULL END) AS in_metrics_notin_map,
    COUNT(CASE WHEN t1.interest_id IS NULL THEN t2.id ELSE NULL END) as notin_metrics_in_map
  FROM fresh_segments.interest_metrics AS t1 
  FULL OUTER JOIN fresh_segments.interest_map AS t2 
    ON t1.interest_id = t2.id;
  ```

  No records in the `interest_metrics` table were found with a foreign key(interest_id) that doesn't exist in `interest_map` table.

2. Check if there are any records of interests where the corresponding `month_year` is before the `created_at` date.
    ```sql
    WITH cte_join AS (
    SELECT
        interest_metrics.*,
        interest_map.interest_name,
        interest_map.interest_summary,
        interest_map.created_at,
        interest_map.last_modified
    FROM fresh_segments.interest_metrics
    INNER JOIN fresh_segments.interest_map
        ON interest_metrics.interest_id = interest_map.id
    WHERE interest_metrics.month_year IS NOT NULL
    )
    SELECT
        COUNT(*)
    FROM cte_join
    WHERE month_year < created_at;
    ```
    There are 188 records where `month_date` of a record is before the `created_at` date.
     In this particular case, this data makes sense as new interests could be created by breaking down and modifying older and broader categories to put more focus on certain interest categories. This is also useful when you want to take a deep dive at the data and understand which specific ads(interests) are performing well in terms of capturing customer attention. 

    For example, let's say we have two interst categories: Sports and automobile enthusiasts. The metrics for these don't really give us a lot of information and insight into performance of difference ads. If we want to know if bike ads have performing better than car ads, it is not possible with the current data. But if we break them down into narrow categories such as bike enthusiasts, car enthusiasts, NFL fans, NBA fans, the data is much more specific and gives us a lot more insights into how well the specific ad campaigns are working. 

    Another reason for this anomaly to make sense is that we are using first day of the month as a proxy for our aggregared monthly metrics. The new interest might be created sometime in the middle of a month. 

3. Check the uniqueness of primary keys in `interest_map` table. 
    To check if there are multiple entries with the same interest_id. 
    ```sql
    WITH record_counts AS(
    SELECT 
        id,
        COUNT(id) AS records
    FROM fresh_segments.interest_map
    GROUP BY id
    )
    SELECT 
        records,
        COUNT(id) AS interest_ids
    FROM record_counts
    GROUP BY records;
    ```
    All interest ids were found to be unique. 

4. Check if we are dealing with an SCD(Slowly Changing Dimension).
    >Usually when we see any columns in a dataset which have the words >created or modified - we should be hearing alarm bells ringing and >we should consider whether we have a SCD table on our hands or a >slowly changing dimension.
    > --<cite>[Danny Ma]</cite>




# Analysis



# Report



