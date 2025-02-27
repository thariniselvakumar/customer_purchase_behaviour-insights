# Ecommerce Behavior Analysis

## Objective:

To analyze and gain insights into the purchasing behavior of ecommerce customers, identifying trends, patterns, and correlations that can inform data-driven marketing strategies and optimize customer experiences.

## Links 
Datasets : https://github.com/thariniselvakumar/customer_purchase_behaviour-insights/blob/main/shopping_trends.csv

SQL quries : https://github.com/thariniselvakumar/customer_purchase_behaviour-insights/blob/main/ecommerce.sql

### Database  Creation
```
CREATE database ecommerce;

USE ecommerce;
SELECT * FROM shopping_trends;
```
## Data cleaning

## --1 -- Checking duplicates
```
SELECT customer_id ,COUNT(*)
FROM shopping_trends
GROUP BY Customer_ID
HAVING COUNT(*)>1
```

## --2 -- Checking null values 
```
SELECT item_purchased ,purchase_amount_USD --Choosing the neccessary columns 
FROM shopping_trends
WHERE Item_Purchased IS NULL AND Purchase_Amount_USD IS NULL;
```

## --3 --Upating Formats 
```
ALTER TABLE shopping_trends ALTER COLUMN review_rating DECIMAL(10,1)
```
## --Data exploration and findings
```
SELECT * FROM shopping_trends;
```
## --1-- Summary statistics
```
SELECT COUNT(*) AS count,
SUM(Purchase_amount_usd) AS sum,
AVG(Purchase_amount_usd) AS avg,
MAX(Purchase_amount_usd) AS max,
MIN(Purchase_amount_usd) AS min,
STDEV(Purchase_amount_usd) AS stdev
FROM shopping_trends;
```
## --2--Count of the item purchased by category 
```
SELECT item_purchased, COUNT(*) AS count ,Category
FROM shopping_trends
GROUP BY Item_Purchased,Category
ORDER BY Category 
```
## --3-- Total  purchase by category 
```SELECT SUM(purchase_amount_usd) AS Totalamt , category
FROM shopping_trends
GROUP BY category
ORDER BY Totalamt DESC
```
## --4--Categorizing purchases by age group ofthe customers
```
SELECT 
		CASE WHEN age <30 THEN 'age[<30]'
		WHEN age BETWEEN 30 AND 50 THEN 'age[30-50]'
		ELSE 'age[>50]' 
		END AS agecategory ,COUNT(*) AS purchasecount
FROM shopping_trends
GROUP BY  CASE WHEN age <30 THEN 'age[<30]'
		  WHEN age BETWEEN 30 AND 50 THEN 'age[30-50]'
		  ELSE 'age[>50]' 
		  END
ORDER BY Agecategory DESC 
```

## --5--Top reviews by customers  
```
SELECT * FROM shopping_trends
SELECT COUNT(*) AS customerpurchasecount,review_rating
FROM shopping_trends
GROUP BY review_rating
ORDER BY Review_Rating DESC
```
## --6--Counts of the item purchased
```
SELECT item_purchased,COUNT(*) AS purchasecount 
FROM shopping_trends
GROUP BY Item_Purchased
ORDER BY purchasecount DESC
```
## --7--Sum of the Previous purchases by customers categorised by the itemspurchased.
```
SELECT SUM(previous_purchases) AS totalpreviouspurchase,Item_Purchased
FROM shopping_trends
GROUP BY Item_Purchased
ORDER BY totalpreviouspurchase DESC
```

## --8-- Analyze the average purchase amount by frequency of purchases.
```
SELECT AVG(purchase_amount_usd) AS avg ,frequency_of_purchases
FROM shopping_trends
GROUP BY Frequency_of_Purchases
ORDER BY avg DESC;
```
## -- 9--top 5 locations by purchase amount
```
SELECT TOP 5  
  location,
  SUM(Purchase_Amount_USD) AS total_revenue
FROM shopping_trends
GROUP BY location
ORDER BY total_revenue DESC
;
```
## --10--Top 5 preferred payment method by the customers
```
SELECT TOP 5 
		COUNT(*) AS customers,Preferred_payment_method
FROM shopping_trends
GROUP BY Preferred_Payment_Method
ORDER BY customers DESC;
```
## --ADVANCE SQL ANALYSIS

## --RFM Analysis
`[RFM (Recency, Frequency, Monetary) analysis is a method used to segment customers based on their purchase behavior.]`


```
  WITH aggregated_data AS (
  SELECT 
    customer_id,
    SUM(purchase_amount_usd) AS total_purchase_amount,
    COUNT(item_purchased) AS total_items,
    MAX(purchase_amount_usd) AS max_purchase_amount
  FROM 
    shopping_trends
  GROUP BY 
    customer_id
)
SELECT 
  customer_id,
  total_purchase_amount,
  total_items,
  NTILE(3) OVER (ORDER BY total_purchase_amount DESC) AS monetary,
  NTILE(3) OVER (ORDER BY total_items DESC) AS frequency,
  NTILE(3) OVER (ORDER BY max_purchase_amount DESC) AS recency,
  CASE 
    WHEN NTILE(3) OVER (ORDER BY total_purchase_amount DESC) = 1 THEN 'High-Value Customers'
    WHEN NTILE(3) OVER (ORDER BY total_purchase_amount DESC) = 2 THEN 'Mid-Value Customers'
    WHEN NTILE(3) OVER (ORDER BY total_purchase_amount DESC) = 3 THEN 'Low-Value Customers'
  END AS monetary_segment,
  CASE 
    WHEN NTILE(3) OVER (ORDER BY total_items DESC) = 1 THEN 'Frequent Buyers'
    WHEN NTILE(3) OVER (ORDER BY total_items DESC) = 2 THEN 'Occasional Buyers'
    WHEN NTILE(3) OVER (ORDER BY total_items DESC) = 3 THEN 'Rare Buyers'
  END AS frequency_segment,
  CASE 
    WHEN NTILE(3) OVER (ORDER BY max_purchase_amount DESC) = 1 THEN 'Recent Buyers'
    WHEN NTILE(3) OVER (ORDER BY max_purchase_amount DESC) = 2 THEN 'Moderately Recent Buyers'
    WHEN NTILE(3) OVER (ORDER BY max_purchase_amount DESC) = 3 THEN 'Least Recent Buyers'
  END AS recency_segment
FROM 
  aggregated_data
ORDER BY 
  Customer_ID;
```

## --Customer Segmentation by Demographics
```
SELECT 
  age,
  gender,
  COUNT(DISTINCT customer_id) AS num_customers,
  SUM(purchase_amount_usd) AS total_revenue
FROM 
  shopping_trends
GROUP BY 
  age, gender
ORDER BY 
  num_customers DESC, total_revenue DESC;
```
`--This query segments customers by their age and gender and calculates the number of customers and total revenue for each segment.`

## --Discount and Promo Code Analysis

```
SELECT 
  discount_applied,
  promo_code_used,
  COUNT(DISTINCT customer_id) AS num_customers,
  SUM(Purchase_Amount_USD) AS total_revenue
FROM 
  shopping_trends
GROUP BY 
  discount_applied, promo_code_used
ORDER BY 
  num_customers DESC, total_revenue DESC;
```
`--analyzes the discount and promo code usage and calculates the number of customers and total revenue for segment`


## --Customer Subscription Status Analysis

```
SELECT 
  subscription_status,
  COUNT(DISTINCT customer_id) AS num_customers,
  SUM(Purchase_Amount_USD) AS total_revenue
FROM 
  shopping_trends
GROUP BY 
  subscription_status
ORDER BY 
  num_customers DESC, total_revenue DESC;
```
`--This query analyzes the customer subscription status and calculates the number of customers and total revenue for each subscription status.`



## Findings: 
1. RFM analysis reveals 3 customer segments.
2. Demographic segmentation shows varying customer numbers and revenue.
3. Discount/promo code analysis reveals usage patterns.
4. Subscription status analysis shows customer loyalty.
5. Purchase behavior analysis reveals frequent, occasional, and rare buyers.
6. Revenue analysis shows varying revenue across segments.

## Solutions:
1. Personalized marketing for RFM segments.
2. Demographic-based targeting.
3. Optimize discount/promo code strategy.
4. Offer subscription-based services.
