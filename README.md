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
