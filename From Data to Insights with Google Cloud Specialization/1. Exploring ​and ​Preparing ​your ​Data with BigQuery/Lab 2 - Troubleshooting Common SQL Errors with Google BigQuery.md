## Troubleshooting Common SQL Errors with Google BigQuery
In this lab,we learn how to perform the following tasks:
- Query the data-to-insights public dataset

- Use the BigQuery Query editor to troubleshoot common SQL errors

- Use the Query Validator

- Troubleshoot syntax and logical SQL errors


dataset = [data-to-insights](https://console.cloud.google.com/bigquery?project=data-to-insights&page=ecommerce)

### Find the total number of customers who went through checkout
Your goal in this section is to construct a query that gives you the number of unique visitors who successfully went through the checkout process for your website. The data is in the rev_transactions table which your data analyst team has provided. They have also given you example queries to help you get started in your analysis but you're not sure they're written correctly.


## Troubleshoot queries that contain query validator, alias, and comma errors
#### Look at the below query and answer the following question:
```
#standardSQL
SELECT
FROM
  `data-to-inghts.ecommerce.rev_transactions`
LIMIT
  1000
 
# There is a typo in the table name
# We have not specified any columns in the SELECT
  
```
#### What about this updated query?
```
#standardSQL
SELECT
  *
FROM
  [DATA-TO-insights:ecommerce.rev_transactions]
LIMIT
  1000
  
# There is a typo in the dataset name
# We are using legacy SQL

```

#### What about this query that uses Standard SQL?
```
#standardSQL
SELECT
FROM
  `data-to-insights.ecommerce.rev_transactions`
  
# Still no columns defined in SELECT
```
#### What about now? This query has a column.

```
#standardSQL
SELECT
  fullVisitorId
FROM
  `data-to-insights.ecommerce.rev_transactions`
  
 # We are missing a column alias
 # Without aggregations, limits, or sorting, this query is not insightful
 # The page title is missing from the columns in SELECT

```
#### What about now? The following query has a page title.
```
#standardSQL
SELECT
  fullVisitorId hits_page_pageTitle
FROM
  `data-to-insights.ecommerce.rev_transactions`
LIMIT
  1000
  
 ## How many columns will the previous query return?
 # 1, a column named hits_page_pageTitle
 
```
#### What about now? The missing comma has been corrected.

```
#standardSQL
SELECT
  fullVisitorId,
  hits_page_pageTitle
FROM
  `data-to-insights.ecommerce.rev_transactions`
LIMIT
  1000
  
 # This returns results, but  visitors are counted twice.

```
## Troubleshoot queries that contain logic errors, GROUP BY statements, and wildcard filters

#### Aggregate the following query to answer the question: How many unique visitors reached checkout?

```
#standardSQL
SELECT
  fullVisitorId,
  hits_page_pageTitle
FROM
  `data-to-insights.ecommerce.rev_transactions`
LIMIT
  1000
```
![image](https://user-images.githubusercontent.com/92245436/152926294-fa53e98b-abfa-4052-8bd4-7752ff3eb1ae.png)

#### What about this? An aggregation function, `COUNT()`, was added.
```
#standardSQL
SELECT
  COUNT(fullVisitorId) AS visitor_count,
  hits_page_pageTitle
FROM
  `data-to-insights.ecommerce.rev_transactions`
  
 # It is missing a GROUP BY clause
 # The COUNT() function does not de-deduplicate the same fullVisitorId
```
#### In this next query, `GROUP BY` and `DISTINCT` statements were added.

```
#standardSQL
SELECT
  COUNT(DISTINCT fullVisitorId) AS visitor_count,
  hits_page_pageTitle
FROM
  `data-to-insights.ecommerce.rev_transactions`
GROUP BY
  hits_page_pageTitle
```
![image](https://user-images.githubusercontent.com/92245436/152926692-ad97375a-4440-418b-8a31-daa23fcd3d9b.png)

### Great! The results are good, but they look strange. Filter to just "Checkout Confirmation" in the results.

#### Question 1: Find the number unique visitors who reached the checkout confirmation page in the rev_transactions table.

```
#standardSQL
SELECT
  COUNT(DISTINCT fullVisitorId) AS visitor_count,
  hits_page_pageTitle
FROM
  `data-to-insights.ecommerce.rev_transactions`
WHERE
  hits_page_pageTitle = "Checkout Confirmation"
GROUP BY
  hits_page_pageTitle
```
![image](https://user-images.githubusercontent.com/92245436/152851137-4120224c-a5ae-4ee3-a678-b815635fae15.png)


### List the cities with the most transactions with your ecommerce site
#### Troubleshoot ordering, calculated fields, and filtering after aggregating errors


#### Question 2: Which cities that have the most transactions with our ecommerce site?

```
SELECT
  geoNetwork_city,
  SUM(totals_transactions) AS totals_transactions,
  COUNT(DISTINCT fullVisitorId) AS distinct_visitors
FROM
  `data-to-insights.ecommerce.rev_transactions`
GROUP BY
  geoNetwork_city
ORDER BY
  distinct_visitors DESC
```
![image](https://user-images.githubusercontent.com/92245436/152851307-c629d3fa-0055-4d7b-84e9-92eef17b86ab.png)

#### Update your query and create a new calculated field to return the average number of products per order by city.
```
#standardSQL
SELECT
  geoNetwork_city,
  SUM(totals_transactions) AS total_products_ordered,
  COUNT( DISTINCT fullVisitorId) AS distinct_visitors,
  SUM(totals_transactions) / COUNT( DISTINCT fullVisitorId) AS avg_products_ordered
FROM
  `data-to-insights.ecommerce.rev_transactions`
GROUP BY
  geoNetwork_city
ORDER BY
  avg_products_ordered DESC
```
![image](https://user-images.githubusercontent.com/92245436/152851616-651db2be-8e06-45a7-b954-f6a7c732f28a.png)

#### Update your query and create a new calculated field to return the average number of products per order by city.
```
#standardSQL
SELECT
  geoNetwork_city,
  SUM(totals_transactions) AS total_products_ordered,
  COUNT( DISTINCT fullVisitorId) AS distinct_visitors,
  SUM(totals_transactions) / COUNT( DISTINCT fullVisitorId) AS avg_products_ordered
FROM
  `data-to-insights.ecommerce.rev_transactions`
GROUP BY
  geoNetwork_city
ORDER BY
  avg_products_ordered DESC
```

![image](https://user-images.githubusercontent.com/92245436/152927620-14c54a95-f274-4303-b313-bafdee683c22.png)

#### Filter your aggregated results to only return cities with more than 20 avg_products_ordered.
```
#standardSQL
SELECT
  geoNetwork_city,
  SUM(totals_transactions) AS total_products_ordered,
  COUNT( DISTINCT fullVisitorId) AS distinct_visitors,
  SUM(totals_transactions) / COUNT( DISTINCT fullVisitorId) AS avg_products_ordered
FROM
  `data-to-insights.ecommerce.rev_transactions`
GROUP BY
  geoNetwork_city
HAVING
  avg_products_ordered > 20
ORDER BY
  avg_products_ordered DESC
```

![image](https://user-images.githubusercontent.com/92245436/152851776-2d6761e2-7324-4be7-8949-e160e4fd2fea.png)

#### Question 3: Find the total number of products in each product category 

#### Find the top selling products by filtering with NULL values
```
#standardSQL
SELECT
  COUNT(DISTINCT hits_product_v2ProductName) AS number_of_products,
  hits_product_v2ProductCategory
FROM
  `data-to-insights.ecommerce.rev_transactions`
WHERE
  hits_product_v2ProductName IS NOT NULL
GROUP BY
  hits_product_v2ProductCategory
ORDER BY
  number_of_products DESC
LIMIT
  5
```
![image](https://user-images.githubusercontent.com/92245436/152851888-7b205a88-f2c9-4e54-970c-9a58e42123cd.png)

