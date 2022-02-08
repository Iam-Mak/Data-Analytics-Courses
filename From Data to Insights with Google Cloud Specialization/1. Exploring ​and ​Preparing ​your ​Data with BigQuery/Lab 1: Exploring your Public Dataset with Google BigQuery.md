### [Explore your Ecommerce Dataset with SQL in Google BigQuery](https://support.google.com/analytics/answer/3437719?hl=en)

### Explore ecommerce data and identify duplicate records
#### Identify duplicate rows
```
#standardSQL
SELECT
  COUNT(*) AS num_duplicate_row,
  *
FROM
  `data-to-insights.ecommerce.all_sessions_raw`
GROUP BY
  fullVisitorId,
  channelGrouping,
  time,
  country,
  city,
  totalTransactionRevenue,
  transactions,
  timeOnSite,
  pageviews,
  sessionQualityDim,
  date,
  visitId,
  type,
  productRefundAmount,
  productQuantity,
  productPrice,
  productRevenue,
  productSKU,
  v2ProductName,
  v2ProductCategory,
  productVariant,
  currencyCode,
  itemQuantity,
  itemRevenue,
  transactionRevenue,
  transactionId,
  pageTitle,
  searchKeyword,
  pagePathLevel1,
  eCommerceAction_type,
  eCommerceAction_step,
  eCommerceAction_option
HAVING
  num_duplicate_row > 1

```
#### Run the query to confirm that no duplicates exist, this time in the all_sessions table:
```
#standardSQL
SELECT 
  fullVisitorId,        # the unique visitor ID  
  visitId,              # a visitor can have multiple visits
  date,                 # session date stored as string YYYYMMDD
  time,                 # time of the individual site hit  (can be 0 to many per visitor session)
  v2ProductName,        # not unique since a product can have variants like Color
  productSKU,           # unique for each product
  type,                 # a visitor can visit Pages and/or can trigger Events (even at the same time)
  eCommerceAction_type, # maps to ‘add to cart’, ‘completed checkout’
  eCommerceAction_step, 
  eCommerceAction_option,
  transactionRevenue,   # revenue of the order
  transactionId,        # unique identifier for revenue bearing transaction
COUNT(*) as row_count 
FROM 
  `data-to-insights.ecommerce.all_sessions` 
  
GROUP BY 
  1,2,3 ,4, 5, 6, 7, 8, 9, 10,11,12

HAVING
  row_count > 1 # find duplicates 

```

#### Write a query that shows total unique visitors
```
#standardSQL
SELECT
  COUNT(*) AS product_views,
  COUNT(DISTINCT fullVisitorId) AS unique_visitors
FROM
  `data-to-insights.ecommerce.all_sessions`
```
![image](https://user-images.githubusercontent.com/92245436/152939417-74c924fb-4b9b-4caf-99da-48408dc4a572.png)


#### Now write a query that shows total unique visitors(fullVisitorID) by the referring site (channelGrouping):
```
#standardSQL
SELECT
  COUNT(DISTINCT fullVisitorId) AS unique_visitors,
  channelGrouping
FROM
  `data-to-insights.ecommerce.all_sessions`
GROUP BY
  channelGrouping
ORDER BY
  channelGrouping DESC;
```
![image](https://user-images.githubusercontent.com/92245436/152939826-7a9db346-5d4c-4ad9-9a2e-0853df2b58db.png)

#### What are all the unique product names listed alphabetically?

```
#standardSQL
SELECT
  (v2ProductName) AS ProductName
FROM
  `data-to-insights.ecommerce.all_sessions`
GROUP BY
  ProductName
ORDER BY
  ProductName
```
![image](https://user-images.githubusercontent.com/92245436/152940926-1987be02-e73b-4e73-8ad6-f5203a3cfc30.png)

#### Write a query to list the five products with the most views (product_views) from all visitors

```
#standardSQL
SELECT
  COUNT(*) AS product_views,
  (v2ProductName) AS ProductName
FROM
  `data-to-insights.ecommerce.all_sessions`
WHERE
  type = 'PAGE'
GROUP BY
  v2ProductName
ORDER BY
  product_views DESC
LIMIT
  5;
```
![image](https://user-images.githubusercontent.com/92245436/152940132-76d75f94-9039-4d00-87b9-7f730d2752a2.png)

#### Now refine the query to no longer double-count product views for visitors who have viewed a product many times. Each distinct product view should only count once per visitor.
```
WITH
  unique_product_views_by_person AS ( -- find each unique product viewed BY each visitor
  SELECT
    fullVisitorId,
    (v2ProductName) AS ProductName
  FROM
    `data-to-insights.ecommerce.all_sessions`
  WHERE
    type = 'PAGE'
  GROUP BY
    fullVisitorId,
    v2ProductName ) -- aggregate the top viewed products and sort them
SELECT
  COUNT(*) AS unique_view_count,
  ProductName
FROM
  unique_product_views_by_person
GROUP BY
  ProductName
ORDER BY
  unique_view_count DESC
LIMIT
  5
```
![image](https://user-images.githubusercontent.com/92245436/152941766-5f3db10b-35ce-46ef-a4cf-bbc7735389e4.png)


#### Expand your previous query to include the total number of distinct products ordered and the total number of total units ordered (productQuantity)
```
#standardSQL
SELECT
  COUNT(*) AS product_views,
  COUNT(productQuantity) AS orders,
  SUM(productQuantity) AS quantity_product_ordered,
  v2ProductName AS Product_name
FROM
  `data-to-insights.ecommerce.all_sessions`
WHERE
  type = 'PAGE'
GROUP BY
  v2ProductName
ORDER BY
  product_views DESC
LIMIT
  5;
```
![image](https://user-images.githubusercontent.com/92245436/152942150-41595159-8568-4dc4-ba76-6c5df00cbd04.png)

#### Expand the query to include the average amount of product per order (total number of units ordered/total number of orders, or SUM(productQuantity)/COUNT(productQuantity)).

```
#standardSQL
SELECT
  COUNT(*) AS product_views,
  COUNT(productQuantity) AS orders,
  SUM(productQuantity) AS quantity_product_ordered,
  SUM(productQuantity) / COUNT(productQuantity) AS avg_per_order,
  (v2ProductName) AS ProductName
FROM
  `data-to-insights.ecommerce.all_sessions`
WHERE
  type = 'PAGE'
GROUP BY
  v2ProductName
ORDER BY
  product_views DESC
LIMIT
  5;
```
![image](https://user-images.githubusercontent.com/92245436/152942673-89a3509a-d329-4c98-87f2-389211cc6dd4.png)



## Lab 1 CHALLENGE
### Challenge 1:
- For products with over 1000 units that have been added to cart or ordered that are not frisbees: 
- How many distinct times was the product part of an order (either complete or incomplete order)? 
-  How many total units of the product were part of orders (either complete or incomplete)? 
- Which product had the highest conversion rate?

``` 
SELECT
  COUNT(*) AS product_views,
  COUNT(productQuantity) AS potential_orders,
  SUM(productQuantity) AS quantity_product_added,
  v2ProductName,
  COUNT(productQuantity) / COUNT(*) AS conversion_rate
FROM
  `data-to-insights.ecommerce.all_sessions`
WHERE
  LOWER(v2ProductName) NOT LIKE '%frisbee%'
GROUP BY
  v2ProductName
HAVING
  quantity_product_added > 1000
ORDER BY
  conversion_rate DESC
LIMIT
  10;
```

### Challenge 2: 
- Write a query that shows the eCommerceAction_type and the distinct count of fullVisitorId associated with each type. 
```
SELECT
  COUNT(DISTINCT fullVisitorId) AS number_of_unique_visitors,
  eCommerceAction_type,
  CASE eCommerceAction_type
    WHEN '0' THEN 'Unknown'
    WHEN '1' THEN 'Click through of product lists'
    WHEN '2' THEN 'Product detail views'
    WHEN '3' THEN 'Add product(s) to cart'
    WHEN '4' THEN 'Remove product(s) from cart'
    WHEN '5' THEN 'Check out'
    WHEN '6' THEN 'Completed purchase'
    WHEN '7' THEN 'Refund of purchase'
    WHEN '8' THEN 'Checkout options'
  ELSE
  NULL
END
  AS eCommerceAction_type_label
FROM
  `data-to-insights.ecommerce.all_sessions`
GROUP BY
  eCommerceAction_type
ORDER BY
  eCommerceAction_type

```

### Challenge 3: 
- Write a query using aggregation functions that returns the unique session ids of those visitors who have added a product to their cart but never completed checkout (abandoned their shopping cart) high quality sessions .

```
SELECT
  CONCAT(fullVisitorId, CAST(visitId AS STRING)) AS unique_session_id,
  # combine TO get a unique session sessionQualityDim,
  SUM(productRevenue) AS transactions_revenue,
  MAX(CAST(eCommerceAction_type AS INT64)) AS checkout_progress
FROM
  `data-to-insights.ecommerce.all_sessions`
WHERE
  sessionQualityDim > 60 # high quality session
GROUP BY
  unique_session_id,
  sessionQualityDim
HAVING
  checkout_progress = 3
  AND # 3 = added TO cart (transactions_revenue IS NULL
    OR transactions_revenue = 0)

```
