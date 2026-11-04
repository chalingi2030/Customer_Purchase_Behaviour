# Customer_Purchase_Behaviour_2026

# Customer Purchasing Behavior & Revenue Optimization Analysis

## Executive Summary

This project evaluates transactional data from **3,863 customer records** across the United States to analyze revenue drivers, discount efficiency, customer loyalty segments, and purchasing habits. The analysis addresses key business questions regarding revenue performance across demographics, product categories, and promotional efficacy.

### **Key Findings & Executive Insights**

* **Discount Inefficiency:** **49.8%** of customers who received discounts ($816$ out of $1,640$) spent at or above the average order value ($\$59.70$). Discounts provided no basket-size lift, full-price orders averaged $\$60.13$ vs. discounted orders at $\$59.12$.

* **Gender Revenue Disparity:** Male customers account for $67.4\%$ of total revenue ($\$155,438$) compared to female customers ($\$75,191$), driven predominantly by transaction volume rather than average basket size ($\$59.44$ vs. $\$60.25$).

* **Loyalty Concentration:** $79.9\%$ of customers ($3,085$) qualify as **Loyal Repeat Buyers** (>10 previous purchases). Subscription status slightly increases repeat purchase rates ($90.9\%$ vs. $88.4\%$).

* **Top Category Performance:** **Clothing** is the primary revenue driver ($\$102,968$), followed by **Accessories** ($\$73,480$).

## Project Architecture & Data Overview

* **Database Engine:** MySQL

* **Dataset Size:** 3,863 rows | 18 attributes

* **Currency:** USD (\$$)

* **Geographic Scope:** United States

```
customer_purchase_behaviour/
â”‚
â”œâ”€â”€ data/
â”‚   â””â”€â”€ customers.csv              # Raw customer dataset
â”œâ”€â”€ scripts/
â”‚   â””â”€â”€ customer_analysis.sql      # Production SQL analytical queries
â””â”€â”€ README.md                      # Executive summary & documentation

```

##  Business Questions & Analytical Breakdown

### 1. Revenue Performance by Demographics & Location

#### **Gender Breakdown**

While female customers have a slightly higher average spend ($\$60.25$) than male customers ($\$59.44$), male customers account for more than twice the total revenue due to higher user counts.

| Gender | Total Customers | Average Spend ($\$$) | Total Revenue ($\$$) | Revenue Share ($\%$) | 
| ----- | ----- | ----- | ----- | ----- | 
| **Male** | 2,615 | 59.44 | 155,438 | 67.4% | 
| **Female** | 1,248 | 60.25 | 75,191 | 32.6% | 

#### **Age Cohort Analysis**

Revenue is strongly concentrated in middle-aged cohorts ($46\text{--}55$ and $26\text{--}35$), with the $66+$ group generating the lowest total revenue.

```
WITH banded AS (
  SELECT purchase_amount,
         CASE WHEN age BETWEEN 18 AND 25 THEN '18-25'
              WHEN age BETWEEN 26 AND 35 THEN '26-35'
              WHEN age BETWEEN 36 AND 45 THEN '36-45'
              WHEN age BETWEEN 46 AND 55 THEN '46-55'
              WHEN age BETWEEN 56 AND 65 THEN '56-65'
              WHEN age >= 66 THEN '66+'
              ELSE 'Other' END AS age_band
  FROM customers
)
SELECT age_band, 
       COUNT(*) AS customers,
       SUM(purchase_amount) AS total_revenue,
       ROUND(AVG(purchase_amount), 2) AS avg_spend
FROM banded
GROUP BY age_band
ORDER BY total_revenue DESC;

```

| Age Band | Customers | Total Revenue ($\$$) | Avg Spend ($\$$) | 
| ----- | ----- | ----- | ----- | 
| **46â€“55** | 743 | 44,927 | 60.47 | 
| **26â€“35** | 736 | 44,008 | 59.79 | 
| **56â€“65** | 745 | 43,930 | 58.97 | 
| **36â€“45** | 722 | 42,853 | 59.35 | 
| **18â€“25** | 566 | 34,281 | 60.57 | 
| **66+** | 351 | 20,630 | 58.77 | 

#### **Top 5 Revenue Generating States**

1. **Montana:** $\$5,717$

2. **California:** $\$5,605$

3. **Illinois:** $\$5,537$

4. **Idaho:** $\$5,438$

5. **Nevada:** $\$5,379$

### 2. Discount Effectiveness & Promotional Analysis

To determine if promotional discounts drive higher order values, we evaluated customer spend behavior across discounted vs. non-discounted orders.

```
-- Query: Discounted vs Non-Discounted Average Spend
SELECT discount_applied, 
       COUNT(*) AS orders,
       ROUND(AVG(purchase_amount), 2) AS avg_spend
FROM customers
GROUP BY discount_applied;

```

```
+------------------+--------+-----------+
| discount_applied | orders | avg_spend |
+------------------+--------+-----------+
| NO               |   2223 |     60.13 |
| YES              |   1640 |     59.12 |
+------------------+--------+-----------+

```

* **Insight:** Discounted orders yield an average spend of $\$59.12$, whereas full-price orders average $\$60.13$ (a $\$1.01$ decline per order under promotions).

* **Discount Waste:** $816$ out of $1,640$ discounted customers ($49.8\%$) would have met or exceeded the average spend threshold regardless of discount allocation.

#### **Products Most Reliant on Discounts**

* **Hats:** $49.67\%$ discount rate

* **Sneakers:** $49.66\%$ discount rate

* **Coats:** $48.43\%$ discount rate

* **Sweaters:** $47.20\%$ discount rate

* **Pants:** $46.43\%$ discount rate

### 3. Product Performance & Categorization

#### **Revenue Distribution by Category**

* **Clothing:** $\$102,968$

* **Accessories:** $\$73,480$

*  **Footwear:** $\$35,778$

*  **Outerwear:** $\$18,403$

#### **Top Ranked Products per Category (Using Window Functions)**

```
WITH item_counts AS (
  SELECT category, item_purchased, COUNT(customer_id) AS total_orders,
         RANK() OVER(PARTITION BY category ORDER BY COUNT(customer_id) DESC) AS item_rank
  FROM customers
  GROUP BY category, item_purchased
)
SELECT item_rank, category, item_purchased, total_orders
FROM item_counts
WHERE item_rank <= 3;

```

* **Accessories:** Jewelry ($170$ orders), Sunglasses & Belts ($159$ orders each)

* **Clothing:** Blouse ($169$ orders), Shirt & Pants ($168$ orders each)

* **Footwear:** Sandals ($159$ orders), Shoes ($147$ orders), Sneakers ($145$ orders)

* **Outerwear:** Jacket ($163$ orders), Coat ($159$ orders)

### 4. Customer Segmentation & Subscription Dynamics

Customers were categorized into three tiers based on past transaction history:

* **New:** 1 purchase ($83$ customers)

* **Returning:** 2â€“10 purchases ($695$ customers)

* **Loyal:** >10 purchases ($3,085$ customers)

#### **Subscription Impact**

```
SELECT subscription_status, 
       COUNT(*) AS total_customers,
       ROUND(AVG(purchase_amount), 2) AS Avg_spent,
       SUM(purchase_amount) AS total_amount
FROM customers
GROUP BY subscription_status;

```

* **Non-Subscribers:** $2,830$ customers | Avg Spend: $\$59.81$ | Total Revenue: $\$169,252$

* **Subscribers:** $1,033$ customers | Avg Spend: $\$59.42$ | Total Revenue: $\$61,377$

* **Repeat Rate:** Subscribers exhibit a $90.9\%$ repeat rate compared to $88.4\%$ for non-subscribers.

##  Strategic Business Recommendations

1. **Restructure Promotional Strategy:** Eliminate broad-based discounts. Since discounts do not increase average order value ($\$59.12$ vs $\$60.13$), transition from upfront percentage discounts to threshold-based incentives (e.g., *"Spend* $\$75$ *to get* $\$10$ *off"*).

2. **Female Segment Acquisition:** Focus marketing efforts on acquiring female buyers. Female customers have a higher basket value ($\$60.25$), but represent only $32.6\%$ of revenue due to lower total customer counts.

3. **Optimize Subscription Value Proposition:** Subscribers currently spend slightly less per order ($\$59.42$) than non-subscribers ($\$59.81$). Introduce subscriber-exclusive bundles to elevate average transaction value.

4. **Seasonal Inventory Management:** Maximize stock allocation during Fall ($\$59,620$) and Winter ($\$58,035$), which account for peak revenue cycles.

   SOURCE scripts/customer analysis.
   

## 
