
# ![Apple Logo](https://github.com/najirh/Apple-Retail-Sales-SQL-Project---Analyzing-Millions-of-Sales-Rows/blob/main/Apple_Changsha_RetailTeamMembers_09012021_big.jpg.slideshow-xlarge_2x.jpg) Apple Retail Sales SQL Project - Analyzing Millions of Sales Rows


## Project Overview

This project is designed to showcase advanced SQL querying techniques through the analysis of over 1 million rows of Apple retail sales data. The dataset includes information about products, stores, sales transactions, and warranty claims across various Apple retail locations globally. By tackling a variety of questions, from basic to complex, you'll demonstrate your ability to write sophisticated SQL queries that extract valuable insights from large datasets.

The project is ideal for data analysts looking to enhance their SQL skills by working with a large-scale dataset and solving real-world business questions.

## Entity Relationship Diagram (ERD)

![ERD](https://github.com/najirh/Apple-Retail-Sales-SQL-Project---Analyzing-Millions-of-Sales-Rows/blob/main/erd.png)


## Database Schema

The project uses five main tables:

1. **stores**: Contains information about Apple retail stores.
   - `store_id`: Unique identifier for each store.
   - `store_name`: Name of the store.
   - `city`: City where the store is located.
   - `country`: Country of the store.

2. **category**: Holds product category information.
   - `category_id`: Unique identifier for each product category.
   - `category_name`: Name of the category.

3. **products**: Details about Apple products.
   - `product_id`: Unique identifier for each product.
   - `product_name`: Name of the product.
   - `category_id`: References the category table.
   - `launch_date`: Date when the product was launched.
   - `price`: Price of the product.

4. **sales**: Stores sales transactions.
   - `sale_id`: Unique identifier for each sale.
   - `sale_date`: Date of the sale.
   - `store_id`: References the store table.
   - `product_id`: References the product table.
   - `quantity`: Number of units sold.

5. **warranty**: Contains information about warranty claims.
   - `claim_id`: Unique identifier for each warranty claim.
   - `claim_date`: Date the claim was made.
   - `sale_id`: References the sales table.
   - `repair_status`: Status of the warranty claim (e.g., Paid Repaired, Warranty Void).

## Objectives

The project is split into three tiers of questions to test SQL skills of increasing complexity:

### Easy to Medium (10 Questions)

1. Find the number of stores in each country.
```SQL
SELECT 
    country,
    COUNT(store_id) AS Total
FROM 
    stores
GROUP BY 
    country
ORDER BY 
    Total DESC;

```
2. Calculate the total number of units sold by each store.
```SQL
SELECT
    f.store_id,
    s.store_name,
    SUM(f.quantity) AS units_sold
FROM 
    fact_sales f
JOIN 
    stores s ON s.store_id = f.store_id
GROUP BY 
    f.store_id, s.store_name
ORDER BY 
    units_sold DESC;
```
3. Identify how many sales occurred in December 2023.
```SQL
SELECT 
    COUNT(sale_id) AS sales_in_december
FROM 
    fact_sales
WHERE 
    DATE_FORMAT(sale_date, '%m-%Y') = '12-2023';
```
4. Determine how many stores have never had a warranty claim filed.
```SQL
SELECT 
    * 
FROM 
    stores
WHERE 
    store_id NOT IN (
        SELECT 
            s.store_id 
        FROM 
            fact_sales s
        JOIN 
            warranty w ON w.sale_id = s.sale_id
    );
```
5. Calculate the percentage of warranty claims marked as "Warranty Void".
```SQL
SELECT 
    ROUND(
        COUNT(*) / (SELECT COUNT(*) FROM warranty) * 100, 2
    ) AS Warranty_void_pct
FROM 
    warranty
WHERE 
    repair_status = 'Warranty Void';
```
6.. Identify which store had the highest total units sold in the last year.
```SQL
SELECT 
    f.store_id,
    s.store_name,
    SUM(f.quantity) AS total
FROM 
    fact_sales f
JOIN 
    stores s ON f.store_id = s.store_id
WHERE 
    f.sale_date >= (CURDATE() - INTERVAL 1 YEAR)
GROUP BY 
    f.store_id, s.store_name
ORDER BY 
    total DESC
LIMIT 1;
```
7. Count the number of unique products sold in the last year.
```SQL
SELECT 
    COUNT(DISTINCT(product_id)) AS Product_count_last_YR  
FROM 
    fact_sales
WHERE 
    sale_date >= '2023-09-27';
```
8. Find the average price of products in each category.
```SQL
SELECT 
    p.category_id,
    c.category_name,
    ROUND(AVG(price), 2) AS Avg_price
FROM 
    products p
JOIN 
    category c ON c.category_id = p.category_id
GROUP BY 
    p.category_id, c.category_name
ORDER BY 
    Avg_price DESC;
```
9. How many warranty claims were filed in 2020?
```SQL
SELECT 
    COUNT(claim_id) AS claims_in_2020
FROM 
    warranty
WHERE 
    YEAR(claim_date) = 2020;
```
10. For each store, identify the best-selling day based on highest quantity sold.
```SQL
SELECT 
    * 
FROM (
    SELECT 
        store_id,
        DAYNAME(sale_date) AS Day,
        SUM(quantity) AS Total,
        RANK() OVER (PARTITION BY store_id ORDER BY SUM(quantity) DESC) AS rank_no
    FROM 
        fact_sales
    GROUP BY 
        store_id, DAY
) AS cte
WHERE 
    rank_no = 1;
```

### Medium to Hard (5 Questions)

11. Identify the least selling product in each country for each year based on total units sold.
```SQL
WITH cte1 AS (
    SELECT 
        s.country,
        f.product_id,
        p.product_name,
        YEAR(f.sale_date) AS Year,
        SUM(f.quantity) AS Total_unit_sold,
        DENSE_RANK() OVER (PARTITION BY s.country, YEAR(f.sale_date) ORDER BY SUM(f.quantity) ASC) AS Rank_no
    FROM 
        stores s
    JOIN 
        fact_sales f ON f.store_id = s.store_id
    JOIN 
        products p ON p.product_id = f.product_id
    GROUP BY 
        s.country, f.product_id, p.product_name, YEAR(f.sale_date)
)
SELECT 
    *
FROM 
    cte1
WHERE 
    Rank_no = 1;
```
12. Calculate how many warranty claims were filed within 180 days of a product sale.
```SQL
SELECT 
    w.*, 
    f.sale_date,
    DATEDIFF(w.claim_date, f.sale_date) AS days
FROM 
    warranty w
LEFT JOIN 
    fact_sales f ON w.sale_id = f.sale_id
WHERE 
    DATEDIFF(w.claim_date, f.sale_date) <= 180;
```
13. Determine how many warranty claims were filed for products launched in the last two years.
```SQL
SELECT 
    p.product_name,
    COUNT(w.claim_id) AS no_of_claims,
    COUNT(f.sale_id) AS total_sales
FROM 
    warranty w
RIGHT JOIN 
    fact_sales f ON f.sale_id = w.sale_id
JOIN 
    products p ON f.product_id = p.product_id
WHERE 
    p.launch_date >= CURDATE() - INTERVAL 2 YEAR
GROUP BY 
    p.product_name;
```
14. List the months in the last three years where sales exceeded 5,000 units in the USA.
```SQL
WITH cte1 AS (
    SELECT 
        s.country,
        DATE_FORMAT(f.sale_date, "%m/%Y") AS Date,
        SUM(f.quantity) AS Total_unit_sold
    FROM 
        fact_sales f
    JOIN 
        stores s ON f.store_id = s.store_id
    WHERE 
        s.country = "USA" 
        AND f.sale_date > CURDATE() - INTERVAL 3 YEAR
    GROUP BY 
        s.country, DATE_FORMAT(f.sale_date, "%m/%Y")
)
SELECT 
    *
FROM 
    cte1
WHERE 
    Total_unit_sold > 5000
ORDER BY 
    s.country, Date ASC;
```
15. Identify the product category with the most warranty claims filed in the last two years.
```SQL
SELECT 
    c.category_name,
    COUNT(w.claim_id) AS Total_claims
FROM 
    warranty w
LEFT JOIN 
    fact_sales f ON w.sale_id = f.sale_id
JOIN 
    products p ON p.product_id = f.product_id
JOIN 
    category c ON c.category_id = p.category_id
WHERE 
    w.claim_date >= CURRENT_DATE() - INTERVAL 2 YEAR
GROUP BY 
    c.category_name
ORDER BY 
    Total_claims DESC;
```

### Complex (5 Questions)

16. Determine the percentage chance of receiving warranty claims after each purchase for each country.
```SQL
SELECT 
    country,
    Total_Qty,
    Total_claims,
    ROUND((Total_claims / Total_Qty) * 100, 2) AS pct_of_claims
FROM (
    SELECT 
        country,
        SUM(quantity) AS Total_Qty,
        COUNT(claim_id) AS Total_claims
    FROM fact_sales AS f
    JOIN stores AS s ON f.store_id = s.store_id
    LEFT JOIN warranty AS w ON f.sale_id = w.sale_id
    GROUP BY country
    HAVING Total_claims > 0
) AS data;
```
17. Analyze the year-by-year growth ratio for each store.
```SQL
WITH Yearly_sales AS (
    SELECT 
        store_name,
        YEAR(sale_date) AS Year,
        SUM(quantity * price) AS Revenue
    FROM fact_sales AS f
    JOIN stores AS s ON f.store_id = s.store_id
    JOIN products AS p ON f.product_id = p.product_id
    GROUP BY store_name, YEAR(sale_date)
),
growth_ratio AS (
    SELECT 
        store_name,
        Year,
        LAG(Revenue, 1) OVER (PARTITION BY store_name ORDER BY Year) AS Last_year_sale,
        Revenue AS Current_year_sale
    FROM Yearly_sales
)
SELECT 
    store_name,
    Year,
    Last_year_sale,
    Current_year_sale,
    ROUND((Current_year_sale - Last_year_sale) / Last_year_sale * 100, 2) AS YoY_Growth_ratio
FROM growth_ratio
WHERE Last_year_sale IS NOT NULL;
```
18. Calculate the correlation between product price and warranty claims for products sold in the last five years, segmented by price range.
```SQL
SELECT 
    CASE
        WHEN price < 500 THEN 'Less expensive product'
        WHEN price BETWEEN 500 AND 1000 THEN 'Mid-range product'
        ELSE 'Expensive product'
    END AS price_segment,
    COUNT(claim_id) AS Total_claims
FROM warranty AS w
LEFT JOIN fact_sales AS f ON w.sale_id = f.sale_id
JOIN products AS p ON p.product_id = f.product_id
WHERE claim_date > CURRENT_DATE - INTERVAL 5 YEAR
GROUP BY price_segment;
```
19. Identify the store with the highest percentage of "Paid Repaired" claims relative to total claims filed.
```SQL
WITH paid_repaired AS (
    SELECT 
        store_name,
        COUNT(*) AS paid_repaired
    FROM warranty AS w
    JOIN fact_sales AS f ON w.sale_id = f.sale_id
    JOIN stores AS s ON f.store_id = s.store_id
    WHERE repair_status = 'Paid Repaired'
    GROUP BY store_name
),
Total AS (
    SELECT 
        store_name,
        COUNT(claim_id) AS Total_repaired
    FROM warranty AS w
    JOIN fact_sales AS f ON w.sale_id = f.sale_id
    JOIN stores AS s ON f.store_id = s.store_id
    GROUP BY store_name
)
SELECT 
    t.store_name,
    p.paid_repaired,
    t.Total_repaired,
    ROUND((p.paid_repaired / t.Total_repaired) * 100, 2) AS pct
FROM Total AS t
JOIN paid_repaired AS p ON t.store_name = p.store_name;
```
20. Write a query to calculate the monthly running total of sales for each store over the past four years and compare trends during this period.
```SQL
SELECT
    f.store_id,
    store_name,
    EXTRACT(YEAR FROM sale_date) AS Year,
    MONTH(sale_date) AS Month,
    SUM(price * quantity) AS Total_sales
FROM fact_sales AS f
JOIN stores AS s ON f.store_id = s.store_id
JOIN products AS p ON p.product_id = f.product_id
WHERE sale_date >= CURRENT_DATE - INTERVAL 4 YEAR
GROUP BY f.store_id, store_name, Year, Month
ORDER BY store_id, Year, Month;
```

### Bonus Question

- Analyze product sales trends over time, segmented into key periods: from launch to 6 months, 6-12 months, 12-18 months, and beyond 18 months.
```SQL
WITH PLC AS (
    SELECT 
        product_name,
        CASE 
            WHEN sale_date BETWEEN launch_date AND launch_date + INTERVAL 6 MONTH THEN '0 to 6 months'
            WHEN sale_date BETWEEN launch_date + INTERVAL 6 MONTH AND launch_date + INTERVAL 12 MONTH THEN '6 to 12 months'
            WHEN sale_date BETWEEN launch_date + INTERVAL 12 MONTH AND launch_date + INTERVAL 18 MONTH THEN '12 to 18 months'
            ELSE 'Beyond 18 months'
        END AS Product_life_cycle,
        quantity
    FROM fact_sales AS f
    JOIN products AS p ON f.product_id = p.product_id
)
SELECT 
    product_name,
    Product_life_cycle,
    SUM(quantity) AS Total_qty,
    CASE 
        WHEN Product_life_cycle = '0 to 6 months' THEN 1
        WHEN Product_life_cycle = '6 to 12 months' THEN 2
        WHEN Product_life_cycle = '12 to 18 months' THEN 3
        ELSE 4
    END AS Rank_no
FROM PLC
GROUP BY product_name, Product_life_cycle
ORDER BY product_name, Rank_no;
```

## Project Focus

This project primarily focuses on developing and showcasing the following SQL skills:

- **Complex Joins and Aggregations**: Demonstrating the ability to perform complex SQL joins and aggregate data meaningfully.
- **Window Functions**: Using advanced window functions for running totals, growth analysis, and time-based queries.
- **Data Segmentation**: Analyzing data across different time frames to gain insights into product performance.
- **Correlation Analysis**: Applying SQL functions to determine relationships between variables, such as product price and warranty claims.
- **Real-World Problem Solving**: Answering business-related questions that reflect real-world scenarios faced by data analysts.


## Dataset

- **Size**: 1 million+ rows of sales data.
- **Period Covered**: The data spans multiple years, allowing for long-term trend analysis.
- **Geographical Coverage**: Sales data from Apple stores across various countries.

## Conclusion

Completing this project has significantly enhanced my advanced SQL querying skills and my ability to work with large, complex datasets. I gained practical experience in solving real-world data analysis challenges that are vital for informed business decision-making. This project not only demonstrates my technical expertise in SQL but also reflects my capability to derive actionable insights from data.

---
