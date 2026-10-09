# E-Commerce Retail Sales Analysis (SQL Project)

## 📌 Project Overview
An exploratory data analysis of an online retail dataset containing over 500,000 transactions. The objective of this project is to uncover key revenue drivers, geographic sales distribution, cancellation impacts, and seasonal trends to provide actionable business recommendations.

## 🛠️ Tools Used
* **SQL:** Data extraction, aggregation, grouping, string filtering, and financial calculations.
* **Dataset:** UCI Online Retail Dataset (via Kaggle).

---

## 📊 Key Business Questions & SQL Analysis

### 1. What are the top 5 best-selling products by total revenue?
* **Insight:** Discovered that shipping fees (*DOTCOM POSTAGE*) ranked as the highest revenue driver, revealing that logistics costs play a massive role in customer transactions alongside core merchandise like the 3-tier cake stand.
```sql
SELECT c3, SUM(c4 * c6) AS Total_Revenue
FROM OnlineRetail
GROUP BY c3
ORDER BY Total_Revenue DESC
LIMIT 5;
```
**Results:**
  * DOTCOM POSTAGE: $206,245.48
  * REGENCY CAKESTAND 3 TIER: $164,762.19
  * WHITE HANGING HEART T-LIGHT HOLDER: $99,668.47
  * PARTY BUNTING: $98,302.98
  * JUMBO BAG RED RETROSPOT: $92,356.03
    
### 2. Which country generates the highest total sales?
* **Insight:** The United Kingdom dominates total sales, proving that the company's market is heavily concentrated domestically, though it maintains a strong secondary footprint in Europe (Netherlands, EIRE, Germany).
```sql
SELECT c8, SUM(c4 * c6) AS Country_Revenue
FROM OnlineRetail
GROUP BY c8
ORDER BY Country_Revenue DESC;
```
**Results:**
  * United Kingdom: $8,187,806.36
  * Netherlands: $284,661.54
  * EIRE (Ireland): $263,276.82

### 3. What is the financial impact of cancelled orders?
* **Insight:** Factored in returns and cancellations to show a more accurate financial picture, demonstrating nearly $900k lost to cancelled transactions.
```sql
SELECT 
    COUNT(DISTINCT c1) AS Total_Cancellations,
    SUM(c4 * c6) AS Total_Lost_Revenue
FROM OnlineRetail
WHERE c1 LIKE 'C%';
```
**Results:**
  * Total Cancellations: 3,836 unique invoices
  * Total Lost Revenue: -$896,812.49
    
### 4. What is the single most expensive item by unit price?
* **Insight:** Items listed as "Manual" represent administrative adjustments, custom service charges, or backend billing entries rather than physical merchandise. Spotting this high value highlights the critical need to clean and separate operational overhead fees from core product sales to ensure accurate financial reporting.
```sql
SELECT c3, c6 AS Unit_Price
FROM OnlineRetail
WHERE c6 <> 'UnitPrice' 
ORDER BY CAST(c6 AS REAL) DESC
LIMIT 1;
```
**Results:**
 * Manual: $38,970

### 5. Who are the top customers by total order count?
* **Insight:** The data reveals a distinct tier of "power users" who place orders at an exceptionally high frequency (with the top customer placing nearly 250 orders). These high-frequency buyers are prime candidates for VIP loyalty tiers, wholesale partnership programs, or special retention marketing.
```sql
SELECT 
    c7 AS Customer_ID,
    COUNT(DISTINCT c1) AS Total_Orders
FROM OnlineRetail
WHERE c7 IS NOT NULL 
  AND c7 != ''  -- Filters out empty text strings
GROUP BY c7
ORDER BY Total_Orders DESC
LIMIT 5;
```
**Results:**
 * Customer 14911: 248 orders
 * Customer 12748: 224 orders
 * Customer 17841: 169 orders
 * Customer 14606: 128 orders
 * 
### 6. How can we segment customers into spending tiers (VIP, Regular, Casual)?
* **Insight:** Segmenting the customer base using conditional logic reveals that while the vast majority are Casual shoppers (< $1k), there is a highly valuable core of 275 VIP customers (> $5k) who drive substantial recurring revenue. This breakdown provides targeted customer groups for future marketing and loyalty campaigns.
```sql
WITH CustomerSpend AS (
    SELECT 
        c7 AS Customer_ID,
        SUM(c4 * c6) AS Total_Spent
    FROM OnlineRetail
    WHERE c7 IS NOT NULL AND c7 != '' AND c1 NOT LIKE 'C%'
    GROUP BY c7
)
SELECT 
    CASE 
        WHEN Total_Spent > 5000 THEN 'VIP (> $5,000)'
        WHEN Total_Spent BETWEEN 1000 AND 5000 THEN 'Regular ($1k - $5k)'
        ELSE 'Casual (< $1k)'
    END AS Spending_Tier,
    COUNT(Customer_ID) AS Customer_Count
FROM CustomerSpend
GROUP BY Spending_Tier
ORDER BY Customer_Count DESC;
```
**Results:**
 * Casual (< $1k): 2,672 customers
 * Regular ($1k - $5k): 1,393 customers
 * VIP (> $5,000): 275 customers

### 7. What is the repeat customer rate and how loyal is our customer base?
* **Insight:** The repeat customer rate stands at an impressive 65.55% (out of 4,340 total customers, 2,845 have returned to make repeat purchases), indicating exceptionally high brand loyalty. Most customers do not just buy once and disappear, representing a core strength that can be leveraged to build VIP memberships or loyalty reward programs to retain this high-value customer segment.
```sql
WITH CustomerOrders AS (
    SELECT 
        c7 AS Customer_ID,
        COUNT(DISTINCT c1) AS Order_Count
    FROM OnlineRetail
    WHERE c7 IS NOT NULL AND c7 != '' AND c1 NOT LIKE 'C%'
    GROUP BY c7
)
SELECT 
    SUM(CASE WHEN Order_Count > 1 THEN 1 ELSE 0 END) AS Repeat_Customers,
    COUNT(Customer_ID) AS Total_Customers,
    ROUND(100.0 * SUM(CASE WHEN Order_Count > 1 THEN 1 ELSE 0 END) / COUNT(Customer_ID), 2) AS Repeat_Rate_Percentage
FROM CustomerOrders;
```
**Results:** 
 * Repeat Customers: 2,845 customers
 * Total Customers: 4,340 customers
 * Repeat Rate Percentage: 65.55%

### 8: Which products are most frequently purchased by repeat customers?
* **Insight:** By analyzing the purchasing habits specifically of our loyal, repeat customer base, we can identify core staple items that drive retention. This helps inventory and marketing teams understand which products are essential for keeping customers coming back.
```sql
  SELECT 
    c3 AS Product_Description,
    COUNT(DISTINCT c7) AS Unique_Repeat_Buyers,
    SUM(c4) AS Total_Quantity_Sold
FROM OnlineRetail
WHERE c7 IS NOT NULL 
  AND c7 != '' 
  AND c1 NOT LIKE 'C%'
  AND c7 IN (
      -- Selects only Customer_IDs that have placed more than 1 order
      SELECT c7 
      FROM OnlineRetail 
      WHERE c7 IS NOT NULL AND c7 != '' AND c1 NOT LIKE 'C%'
      GROUP BY c7 
      HAVING COUNT(DISTINCT c1) > 1
  )
GROUP BY c3
ORDER BY Unique_Repeat_Buyers DESC
LIMIT 5;
```
**Results:** 
 * REGENCY CAKESTAND 3 TIER: 741 unique repeat buyers (11,802 total units sold)
 * WHITE HANGING HEART T-LIGHT HOLDER: 698 unique repeat buyers (35,190 total units sold)
 * PARTY BUNTING: 605 unique repeat buyers (14,559 total units sold)
 * ASSORTED COLOUR BIRD ORNAMENT: 565 unique repeat buyers (33,823 total units sold)
 * JUMBO BAG RED RETROSPOT: 557 unique repeat buyers (44,531 total units sold)
