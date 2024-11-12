# Home-Sales-challenge
- Module 22 Challenge
- Steph Abegg

This challenge demonstrates the usage of PySpark and SparkSQL to determine key metrics on home sale data from 2019 to 2022. We use Spark to create temporary views, partition the data, cache and uncache a temporary table, and verify that the table has been uncached.

The data is loaded from an S3 bucket but can also be found in this repository at [home_sales_revised.csv](Resources/home_sales_revised.csv) . The analysis is done in the [Home_Sales.ipynb](Home_Sales.ipynb) notebook.

## Analysis

I analyzed the data to answer the following questions:

1. What is the average price for a four-bedroom house sold for each year?

```python
SELECT
    YEAR(date) AS year_sold,
    ROUND(AVG(price), 2) AS avg_price
FROM home_sales
WHERE bedrooms=4
GROUP BY year_sold
ORDER BY year_sold
```

<img src="images/q1.png" width=300>

For these four years of data, there is not an obvious trend in average price of four-bedroom homes from year to year.

2. What is the average price of a home for each year the home was built, that has three bedrooms and three bathrooms?

```python
SELECT
    date_built as year_built,
    ROUND(AVG(price), 2) AS avg_price
FROM home_sales
WHERE bedrooms=3 AND bathrooms=3
GROUP BY year_built
ORDER BY year_built
```

<img src="images/q2.png" width=300>

Three-bedroom, three-bathroom homes built in 2015 have the lowest average sale price of these kind of homes while three-bedroom, three-bathroom homes built in 2013 have the highest average sale price of these kind of homes.

3. What is the average price of a home for each year the home was built, that has three bedrooms, three bathrooms, two floors, and is greater than or equal to 2,000 square feet?

```python
SELECT
    date_built as year_built,
    ROUND(AVG(price), 2) AS avg_price
FROM home_sales
WHERE
    bedrooms=3
    AND bathrooms=3
    AND floors=2
    AND sqft_living>=2000
GROUP BY year_built
ORDER BY year_built
```

<img src="images/q3.png" width=300>

Three-bedroom, three-bathroom, two-floor, at least 2000 square foot homes built in 2011 have the lowest average sale price of these kind of homes while Three-bedroom, three-bathroom, two-floor, at least 2000 square foot built in 2012 have the highest average sale price of these kind of homes.


4. What is the average price of a home per "view" rating having an average home price greater than or equal to $350,000?
   
```python
SELECT
    view,
    ROUND(AVG(price), 2) AS avg_price
FROM home_sales
GROUP BY view
HAVING avg_price >= 350000
ORDER BY view DESC
```

<img src="images/q4_1.png" width=300>

50 results returned.

At least for the top 20 view ratings, there does not appear to be a trend of average price.


The last question was used to compare run time with a cached table and partitioned parquet data (with the query being modified to use the partitioned table). The partition was by the "date_built" field. The results of running the query five times are shown in the table below in seconds:

|   | 1 (load)    | 2    | 3    | 4    | 5    | Avg 2-5    |
|-------------|-------------|-------------|-------------|-------------|-------------|-------------|
|Standard	|0.4185|	0.2205	|0.4027	|0.3108|	0.5537|	0.3719|
|Cached|	0.4307|	0.2818|	0.2551|	0.1975|	0.2650|	0.2499 |
|Parquet	|0.7521	|0.4725	|0.4590|	0.5148	|0.3451	|0.4479 |

The first run was not included in the average as it was when the cell was run for the first time.

## Conclusion

From the time table, we can see that running a query on the cached table was the fastest option. We also see that the partitioned parquet files were the slowest. This is likely due to the partition being on the "date_built" column, which was not used in the query and the size of the data was not very big.
