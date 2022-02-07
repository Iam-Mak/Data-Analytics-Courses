### [Lab: Explore your Ecommerce Dataset with SQL in Google BigQuery](https://support.google.com/analytics/answer/3437719?hl=en)


```
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

```
SELECT 
  fullVisitorId, # the unique visitor ID  
  visitId, # a visitor can have multiple visits
  date, # session date stored as string YYYYMMDD
  time, # time of the individual site hit  (can be 0 to many per visitor session)
  v2ProductName, # not unique since a product can have variants like Color
  productSKU, # unique for each product
  type, # a visitor can visit Pages and/or can trigger Events (even at the same time)
  eCommerceAction_type, # maps to ‘add to cart’, ‘completed checkout’
  eCommerceAction_step, 
  eCommerceAction_option,
  transactionRevenue, # revenue of the order
  transactionId, # unique identifier for revenue bearing transaction
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
SELECT
  COUNT(*) AS product_views,
  COUNT(DISTINCT fullVisitorId) AS unique_visitors
FROM
  `data-to-insights.ecommerce.all_sessions`
```
#### Write a query that shows total unique visitors by channel grouping (organic, referring site)

```
SELECT
  COUNT(*) AS product_views,
  COUNT(DISTINCT fullVisitorId) AS unique_visitors,
  channelGrouping
FROM
  `data-to-insights.ecommerce.all_sessions`
GROUP BY
  channelGrouping
ORDER BY
  unique_visitors DESC
LIMIT
  3;

```
#### #What are all the unique product names listed alphabetically?

```
SELECT
  v2ProductName
FROM
  `data-to-insights.ecommerce.all_sessions`
GROUP BY
  v2ProductName
ORDER BY
  v2ProductName;
```
#### Which 5 products had the most views from unique visitors viewed each product?
```
SELECT
  COUNT(*) AS product_view,
  v2ProductName
FROM
  `data-to-insights.ecommerce.all_sessions`
WHERE
  type = 'PAGE'
GROUP BY
  v2ProductName
ORDER BY
  product_view DESC
LIMIT
  5;
```

#### Expand your previous query to include the total number of distinct products ordered as well as the total number of total units ordered

```
SELECT
  COUNT(*) AS product_view,
  SUM(productQuantity) AS quantity_product_ordered,
  COUNT(productQuantity) AS potential_or_completed_orders,
  v2ProductName
FROM
  `data-to-insights.ecommerce.all_sessions`
WHERE
  type = 'PAGE' #
  AND eCommerceAction_type = '6'
GROUP BY
  v2ProductName
ORDER BY
  product_view DESC
LIMIT
  5;
```
#### Expand the query to include the ratio of product units to order:

```
SELECT
  COUNT(*) AS product_view,
  SUM(productQuantity) AS quantity_product_ordered,
  COUNT(productQuantity) AS potential_or_completed_orders,
  SUM(productQuantity) / COUNT(productQuantity) AS avg_per_order,
  v2ProductName
FROM
  `data-to-insights.ecommerce.all_sessions`
WHERE
  type = 'PAGE' #
  AND eCommerceAction_type = '6'
GROUP BY
  v2ProductName
ORDER BY
  product_view DESC
LIMIT
  5;
```


```
SELECT
  COUNT(*) AS product_view,
  SUM(productQuantity) AS quantity_product_ordered,
  COUNT(productQuantity) AS potential_or_completed_orders,
  v2ProductName
FROM
  `data-to-insights.ecommerce.all_sessions`
WHERE
  type = 'PAGE' #
  AND eCommerceAction_type = '6'
GROUP BY
  v2ProductName
ORDER BY
  product_view DESC
LIMIT
  5;
```
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
