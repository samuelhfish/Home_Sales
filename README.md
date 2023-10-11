# Home_Sales
Exercise with home sales data using Spark to create temporary views, run SQL queries, partition data, cache and uncache a temporary table, and verify that the table has been uncached.

Complete code, notebook and results are located in the "Notebooks" folder.

Work was done on Google Colab. That notebook can be found here:
https://drive.google.com/file/d/1XTfRPODIEg1SPiq5f9iW9gwjlzWb27Xz/view?usp=sharing

Note: Make sure you are using the latest version of spark 3.x from http://www.apache.org/dist/spark/

<![Screenshot 2023-10-11 at 1 31 22 PM](https://github.com/samuelhfish/Home_Sales/assets/125224990/8ac074d5-0305-4f62-9113-b36fc0c550bf)><br>

Full instructions and code accessible via Google Colab link above.

## Results

```python
# What is the average price for a four bedroom house sold in each year rounded to two decimal places?
spark.sql("""
  SELECT
    year_sold,
    ROUND(AVG(price),2) AS average_price
  FROM home_sales
  WHERE bedrooms = 4
  GROUP BY year_sold
  """).show()
```
```
+---------+-------------+
|year_sold|average_price|
+---------+-------------+
|     2022|    296363.88|
|     2019|     300263.7|
|     2020|    298353.78|
|     2021|    301819.44|
+---------+-------------+
```

```python
# What is the average price of a home for each year the home was built that have 3 bedrooms and 3 bathrooms rounded to two decimal places?
spark.sql("""
  SELECT
    date_built,
    ROUND(AVG(price),2) AS average_price
  FROM home_sales
  WHERE bedrooms = 3 AND bathrooms = 3
  GROUP BY date_built
  """).show()
```
```
+----------+-------------+
|date_built|average_price|
+----------+-------------+
|      2015|     288770.3|
|      2013|    295962.27|
|      2014|    290852.27|
|      2012|    293683.19|
|      2016|    290555.07|
|      2010|    292859.62|
|      2011|    291117.47|
|      2017|    292676.79|
+----------+-------------+
```

```python
# What is the average price of a home for each year built that have 3 bedrooms, 3 bathrooms, with two floors,
# and are greater than or equal to 2,000 square feet rounded to two decimal places?
spark.sql("""
  SELECT
    date_built,
    ROUND(AVG(price),2) AS average_price
  FROM home_sales
  WHERE bedrooms = 3 AND bathrooms = 3 AND floors = 2 AND sqft_living >= 2000
  GROUP BY date_built
  """).show()
```

```
+----------+-------------+
|date_built|average_price|
+----------+-------------+
|      2015|    297609.97|
|      2013|    303676.79|
|      2014|    298264.72|
|      2012|    307539.97|
|      2016|     293965.1|
|      2010|    285010.22|
|      2011|    276553.81|
|      2017|    280317.58|
+----------+-------------+
```

```python
# What is the "view" rating for the average price of a home, rounded to two decimal places, where the homes are greater than
# or equal to $350,000? Although this is a small dataset, determine the run time for this query.

start_time = time.time()

spark.sql("""
  SELECT
    view,
    ROUND(AVG(price),2) AS average_price
  FROM home_sales
  WHERE price >= 350000
  GROUP BY view
  """).show()

print("--- %s seconds ---" % (time.time() - start_time))
```
```
+----+-------------+
|view|average_price|
+----+-------------+
|  31|    399856.95|
|  85|   1056336.74|
|  65|    736679.93|
|  53|     755214.8|
|  78|   1080649.37|
|  34|    401419.75|
|  81|   1053472.79|
|  28|    402124.62|
|  76|   1058802.78|
|  26|    401506.97|
|  27|    399537.66|
|  44|    400598.05|
|  12|    401501.32|
|  91|   1137372.73|
|  22|    402022.68|
|  93|   1026006.06|
|  47|     398447.5|
|   1|    401044.25|
|  52|    733780.26|
|  13|    398917.98|
+----+-------------+
only showing top 20 rows

--- 0.7393250465393066 seconds ---
```

```python
#  Using the cached data, run the query that filters out the view ratings with average price 
#  greater than or equal to $350,000. Determine the runtime and compare it to uncached runtime.

start_time = time.time()

spark.sql("""
  SELECT
    view,
    ROUND(AVG(price),2) AS average_price
  FROM home_sales
  WHERE price >= 350000
  GROUP BY view
  """).show()

```

```
+----+-------------+
|view|average_price|
+----+-------------+
|  31|    399856.95|
|  85|   1056336.74|
|  65|    736679.93|
|  53|     755214.8|
|  78|   1080649.37|
|  34|    401419.75|
|  81|   1053472.79|
|  28|    402124.62|
|  76|   1058802.78|
|  26|    401506.97|
|  27|    399537.66|
|  44|    400598.05|
|  12|    401501.32|
|  91|   1137372.73|
|  22|    402022.68|
|  93|   1026006.06|
|  47|     398447.5|
|   1|    401044.25|
|  52|    733780.26|
|  13|    398917.98|
+----+-------------+
only showing top 20 rows

--- 0.6342208385467529 seconds ---
```

```python 
# Run the query that filters out the view ratings with average price of greater than or equal to $350,000 
# with the parquet DataFrame. Round your average to two decimal places. 
# Determine the runtime and compare it to the cached version.

start_time = time.time()

spark.sql("""
  SELECT
    view,
    ROUND(AVG(price),2) AS average_price
  FROM p_home_sales
  WHERE price >= 350000
  GROUP BY view
  """).show()
```

```
+----+-------------+
|view|average_price|
+----+-------------+
|  31|    399856.95|
|  85|   1056336.74|
|  65|    736679.93|
|  53|     755214.8|
|  78|   1080649.37|
|  34|    401419.75|
|  81|   1053472.79|
|  28|    402124.62|
|  76|   1058802.78|
|  26|    401506.97|
|  27|    399537.66|
|  44|    400598.05|
|  12|    401501.32|
|  91|   1137372.73|
|  22|    402022.68|
|  93|   1026006.06|
|  47|     398447.5|
|   1|    401044.25|
|  52|    733780.26|
|  13|    398917.98|
+----+-------------+
only showing top 20 rows

--- 0.3803226947784424 seconds ---
```
## Analysis
While the data set wasn't extremely large, by the end of the exercise we can see the time saved by using the cached table and the parquet formatted table. The required time  goes down from .74 to .63 and finally .38 seconds with the parquet formatted table.
