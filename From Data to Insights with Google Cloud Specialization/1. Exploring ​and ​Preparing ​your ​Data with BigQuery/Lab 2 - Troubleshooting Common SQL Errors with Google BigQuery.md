## Troubleshooting Common SQL Errors with Google BigQuery

### Question 1: Find the number unique visitors who reached the checkout confirmation page in the rev_transactions table.

```
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


### Question 2: Which cities that have the most transactions with our ecommerce site?

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

### Update your query and create a new calculated field to return the average number of products per order by city.
```
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


### Filter your aggregated results to only return cities with more than 20 avg_products_ordered.
```
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

### Question 3: Find total number of products in each product category, filtering with NULL values.
```
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

