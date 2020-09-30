
# SQL:  Structured Query Language  Exercise

### Getting Started
1. Go to BigQuery UI https://console.cloud.google.com/bigquery
2. Add in the public data sets. 
	3. Click the Add Data icon
	4. Add any dataset
	5. `bigquery-public-data` should become visible and populate in the BigQuery UI. 
3. Add your queries where it says [YOUR QUERY HERE].
4. Make sure you add your query in between the triple tick marks. 
---

For this section of the exercise we will be using the `bigquery-public-data.austin_311.311_service_requests`  table. 

5. Write a query that tells us how many rows are in the table. 
	```
	SELECT
      COUNT(*)
    FROM
      `bigquery-public-data.austin_311.311_service_requests`
	```

7. Write a query that tells us how many _distinct_ values there are in the complaint_description column.
	``` 
	SELECT
      count(distinct(complaint_description ))
    FROM
      `bigquery-public-data.austin_311.311_service_requests`
	```
  
8. Write a query that counts how many times each owning_department appears in the table and orders them from highest to lowest. 
	``` 
	SELECT
      owning_department, count(owning_department)
    FROM
      `bigquery-public-data.austin_311.311_service_requests`
    group by owning_department

	```

9. Write a query that lists the top 5 complaint_description that appear most and the amount of times they appear in this table. (hint... limit)
	```
	SELECT
      complaint_description,
      COUNT(complaint_description) as n
    FROM
      `bigquery-public-data.austin_311.311_service_requests`
    GROUP BY
      complaint_description
    order by n desc
    limit 5
	  ```
10. Write a query that lists and counts all the complaint_description, just for the where the owning_department is 'Animal Services Office'.

	```
	SELECT
      complaint_description,
      COUNT(complaint_description) AS n
    FROM
      `bigquery-public-data.austin_311.311_service_requests`
    where owning_department = 'Animal Services Office'
    GROUP BY complaint_description
	```

11. Write a query to check if there are any duplicate values in the unique_key column (hint.. There are two was to do this, one is to use a temporary table for the groupby, then filter for values that have more than one count, or, using just one table but including the `having` function). 
	```
	SELECT COUNT(unique_key)
    FROM `bigquery-public-data.austin_311.311_service_requests`
    GROUP BY unique_key 
    HAVING COUNT(unique_key) > 1

	```


### For the next question, use the `census_bureau_usa` tables.

1. Write a query that returns each zipcode and their population for 2000 and 2010. 
	```
	SELECT zipcode, population 
    FROM `bigquery-public-data.census_bureau_usa.*` 
	```

### For the next section, use the  `bigquery-public-data.google_political_ads.advertiser_weekly_spend` table.
1. Using the `advertiser_weekly_spend` table, write a query that finds the advertiser_name that spent the most in usd. 
	```
	SELECT
      advertiser_name,
      sum(spend_usd) as spent
    FROM
      `bigquery-public-data.google_political_ads.advertiser_weekly_spend`
    group by advertiser_name 
    order by spent desc
    LIMIT
      1000
	```
2. Who was the 6th highest spender? (No need to insert query here, just type in the answer.)
	```
	Tom Seyer 2020
	```

3. What week_start_date had the highest spend? (No need to insert query here, just type in the answer.)
	```
	2020-09-20 26594900
	```

4. Using the `advertiser_weekly_spend` table, write a query that returns the sum of spend by week (using week_start_date) in usd for the month of August only. 
	```
	SELECT
      week_start_date,
      sum(spend_usd) as spent 
    FROM
      `bigquery-public-data.google_political_ads.advertiser_weekly_spend`
    Where extract(month from week_start_date) = 08
    group by week_start_date  
    order by spent desc
    LIMIT
      1000
	```
6.  How many ads did the 'TOM STEYER 2020' campaign run? (No need to insert query here, just type in the answer.)
	```
	50
	```
7. Write a query that has, in the US region only, the total spend in usd for each advertiser_name and how many ads they ran. (Hint, you're going to have to join tables for this one). 
	```
		WITH
      advertiser_stat AS (
      SELECT
        regions,
        advertiser_name
      FROM
        `bigquery-public-data.google_political_ads.advertiser_stats`
      WHERE
        regions = 'US' ),
      advertiser_weekly AS (
      SELECT
        advertiser_name,
        spend_usd
      FROM
        `bigquery-public-data.google_political_ads.advertiser_weekly_spend` )
    SELECT
     
      advertiser_stat.advertiser_name AS advertiser_name,
      advertiser_weekly.spend_usd AS Total_spend,
      COUNT(advertiser_weekly.advertiser_name) AS total_ads
      
    FROM
      advertiser_stat,
      advertiser_weekly
    WHERE
      advertiser_stat.advertiser_name = advertiser_weekly.advertiser_name
    GROUP BY
      advertiser_stat.advertiser_name, advertiser_weekly.spend_usd 
    ORDER BY
      advertiser_weekly.spend_usd DESC
	```
8. For each advertiser_name, find the average spend per ad. 
	```
	WITH
      advertiser_stat AS (
      SELECT
        regions,
        advertiser_name
      FROM
        `bigquery-public-data.google_political_ads.advertiser_stats`
      WHERE
        regions = 'US' ),
      advertiser_weekly AS (
      SELECT
        advertiser_name,
        spend_usd
      FROM
        `bigquery-public-data.google_political_ads.advertiser_weekly_spend` )
    SELECT
     
      advertiser_stat.advertiser_name AS advertiser_name,
      avg(advertiser_weekly.spend_usd) AS averge_spent,
      COUNT(advertiser_weekly.advertiser_name) AS total_ads
      
    FROM
      advertiser_stat,
      advertiser_weekly
    WHERE
      advertiser_stat.advertiser_name = advertiser_weekly.advertiser_name
    GROUP BY
      advertiser_stat.advertiser_name
    ORDER BY
      Avg(advertiser_weekly.spend_usd) DESC
	```
10. Which advertiser_name had the lowest average spend per ad that was at least above 0. 
	``` 
	FRIENDS OF EARL GRANVILLE
	```
## For this next section, use the `new_york_citibike` datasets.

1. Who went on more bike trips, Males or Females?
	```
	Males
	```
2. What was the average, shortest, and longest bike trip taken in minutes?
	```
	  AVG  | Shortest | Longest
    962.49     60       19510049
	```

3. Write a query that, for every station_name, has the amount of trips that started there and the amount of trips that ended there. (Hint, use two temporary tables, one that counts the amount of starts, the other that counts the number of ends, and then join the two.) 
	```
	WITH
      citibike_trips_start AS (
      SELECT
        start_station_id, 
        count(start_station_id) as start_count
      FROM
       `bigquery-public-data.new_york_citibike.citibike_trips`
       group by start_station_id 
      ),
      citibike_trips_end AS (
      SELECT
       end_station_id,
      COUNT(end_station_id) AS end_count
    FROM
      `bigquery-public-data.new_york_citibike.citibike_trips`
      group by end_station_id 
      )
    SELECT
      *
    FROM
      citibike_trips_start join citibike_trips_end on citibike_trips_start.start_station_id = citibike_trips_end.end_station_id 
	```
# The next section is the Google Colab section.  
1. Open up this [this Colab notebook](https://colab.research.google.com/drive/1kHdTtuHTPEaMH32GotVum41YVdeyzQ74?usp=sharing).
2. Save a copy of it in your drive. 
3. Rename your saved version with your initials. 
4. Click the 'Share' button on the top right.  
5. Change the permissions so anyone with link can view. 
6. Copy the link and paste it right below this line. 
	* YOUR LINK: https://colab.research.google.com/drive/1UPDys2GfxnF9w86v73k-tj3Bzm-qCiD4?usp=sharing#scrollTo=JySF3z4E6xT3
9. Complete the two questions in the colab notebook file. 
